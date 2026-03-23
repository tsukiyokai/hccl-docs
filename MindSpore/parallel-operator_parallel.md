# 算子级并行

## 简介

MindSpore提供了两种粒度的算子级并行能力来处理大模型训练。大规模神经网络的参数量超过单设备内存容量，使得单卡或数据并行均无法进行训练。算子级并行通过将网络张量分割并分配到多个设备，降低单设备内存消耗。该框架支持两种方法：

- 算子级并行：通过简单切分策略描述张量维度分布
- 高阶算子级并行：通过开放设备排布描述支持复杂切分场景

两种方法均支持ops和mint算子。

## 算子级并行实践

### Ops算子并行实践

#### 配置分布式环境

```python
import mindspore as ms
from mindspore.communication import init

ms.set_context(mode=ms.GRAPH_MODE)
ms.runtime.set_memory(max_size="28GB")
init()
ms.set_seed(1)
```

关键步骤包括：通过`init`初始化通信域，使用`set_memory`的`max_size`限制设备内存。

#### 数据集加载

```python
import os
import mindspore.dataset as ds

def create_dataset(batch_size):
    dataset_path = os.getenv("DATA_PATH")
    dataset = ds.MnistDataset(dataset_path)
    image_transforms = [
        ds.vision.Rescale(1.0 / 255.0, 0),
        ds.vision.Normalize(mean=(0.1307,), std=(0.3081,)),
        ds.vision.HWC2CHW()
    ]
    label_transform = ds.transforms.TypeCast(ms.int32)
    dataset = dataset.map(image_transforms, 'image')
    dataset = dataset.map(label_transform, 'label')
    dataset = dataset.batch(batch_size)
    return dataset

data_set = create_dataset(32)
```

#### 定义网络

```python
import mindspore as ms
from mindspore import nn, ops
from mindspore.common.initializer import initializer

class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.flatten = ops.Flatten()
        self.fc1_weight = ms.Parameter(initializer("normal", [28*28, 512], ms.float32))
        self.fc2_weight = ms.Parameter(initializer("normal", [512, 512], ms.float32))
        self.fc3_weight = ms.Parameter(initializer("normal", [512, 10], ms.float32))
        self.matmul1 = ops.MatMul().shard(((2, 4), (4, 1)))
        self.relu1 = ops.ReLU().shard(((4, 1),))
        self.matmul2 = ops.MatMul().shard(((1, 8), (8, 1)))
        self.relu2 = ops.ReLU().shard(((8, 1),))
        self.matmul3 = ops.MatMul()

    def construct(self, x):
        x = self.flatten(x)
        x = self.matmul1(x, self.fc1_weight)
        x = self.relu1(x)
        x = self.matmul2(x, self.fc2_weight)
        x = self.relu2(x)
        logits = self.matmul3(x, self.fc3_weight)
        return logits
```

切分策略说明：`ops.MatMul().shard(((2, 4), (4, 1)))`表示第一个输入行切分2份、列切分4份；第二个输入行切分4份。相同策略的算子可复用，但不同策略需分别定义。

#### 训练网络定义

```python
from mindspore.nn.utils import no_init_parameters

with no_init_parameters():
    net = Network()
    optimizer = nn.SGD(net.trainable_params(), 1e-2)

loss_fn = nn.CrossEntropyLoss()

def forward_fn(data, target):
    logits = net(data)
    loss = loss_fn(logits, target)
    return loss, logits

grad_fn = ms.value_and_grad(forward_fn, None, net.trainable_params(), has_aux=True)

def train_step(inputs, targets):
    (loss_value, _), grads = grad_fn(inputs, targets)
    optimizer(grads)
    return loss_value
```

使用`no_init_parameters`进行延迟初始化，将参数初始化推迟到多卡并行阶段。

#### 并行配置

```python
from mindspore.parallel.auto_parallel import AutoParallel

parallel_net = AutoParallel(train_step, parallel_mode="semi_auto")
```

#### 训练循环

```python
for epoch in range(10):
    i = 0
    for image, label in data_set:
        loss_output = parallel_net(image, label)
        if i % 10 == 0:
            print("epoch: %s, step: %s, loss is %s" % (epoch, i, loss_output))
        i += 1
```

#### 运行脚本

```bash
bash run.sh
```

日志输出示例：

```
epoch: 0 step: 0, loss is 2.3016002
epoch: 0 step: 10, loss is 2.2889402
epoch: 0 step: 20, loss is 2.2843816
epoch: 0 step: 30, loss is 2.248126
epoch: 0 step: 40, loss is 2.1581488
epoch: 0 step: 50, loss is 1.8051043
```

### Mint算子并行实践

#### 配置分布式环境

同ops算子并行。

#### 数据集加载

同ops算子并行。

#### 定义网络

```python
import mindspore as ms
from mindspore import nn, mint

class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.flatten = mint.flatten
        self.fc1_weight = ms.Parameter(initializer("normal", [28*28, 512], ms.float32))
        self.fc2_weight = ms.Parameter(initializer("normal", [512, 512], ms.float32))
        self.fc3_weight = ms.Parameter(initializer("normal", [512, 10], ms.float32))
        self.matmul1 = ms.parallel.shard(mint.matmul, in_strategy=((2, 4), (4, 1)))
        self.relu1 = ms.parallel.shard(mint.nn.functional.relu, in_strategy=((4, 1),))
        self.matmul2 = ms.parallel.shard(mint.matmul, in_strategy=((1, 8), (8, 1)))
        self.relu2 = ms.parallel.shard(mint.nn.functional.relu, in_strategy=((8, 1),))
        self.matmul3 = mint.matmul

    def construct(self, x):
        x = self.flatten(x)
        x = self.matmul1(x, self.fc1_weight)
        x = self.relu1(x, dim=0, keepdims=True)
        x = self.matmul2(x, self.fc2_weight)
        x = self.relu2(x, dim=0, keepdims=True)
        logits = self.matmul3(x, self.fc3_weight)
        return logits

net = Network()
```

mint算子作为函数式接口不直接暴露原语，需用`mindspore.parallel.shard`手动配置切分策略。

#### 并行配置

```python
from mindspore.parallel.auto_parallel import AutoParallel

parallel_net = AutoParallel(net, parallel_mode="semi_auto")
```

#### 执行网络

```python
for epoch in range(10):
    i = 0
    for image, _ in data_set:
        forward_logits = parallel_net(image)
        if i % 10 == 0:
            forward_sum = mint.sum(forward_logits).asnumpy()
            print("epoch: %s, step: %s, forward_sum is %s" % (epoch, i, forward_sum))
        i += 1
```

#### 运行脚本

```bash
bash run_mint.sh
```

日志输出示例：

```
epoch: 0 step: 0, forward_sum is 0.90023
epoch: 0 step: 10, forward_sum is 1.07679
epoch: 0 step: 20, forward_sum is 1.02521
epoch: 0 step: 30, forward_sum is 0.96682
epoch: 0 step: 40, forward_sum is 0.93158
epoch: 0 step: 50, forward_sum is 0.96655
```

## 高阶算子级并行实践

### 高阶Ops算子并行实践

#### 定义网络

高阶算子级并行扩展`shard`接口，`in_strategy`/`out_strategy`参数接收`tuple(Layout)`类型。Layout通过设备矩阵初始化并为每轴命名：

```python
import mindspore as ms
from mindspore import nn, ops
from mindspore.common.initializer import initializer
from mindspore.parallel import Layout

class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.flatten = ops.Flatten()
        self.fc1_weight = ms.Parameter(initializer("normal", [28*28, 512], ms.float32))
        self.fc2_weight = ms.Parameter(initializer("normal", [512, 512], ms.float32))
        self.fc3_weight = ms.Parameter(initializer("normal", [512, 10], ms.float32))
        layout = Layout((2, 2, 2), ("dp", "sp", "mp"))
        layout2 = Layout((8,), ("tp",))
        self.matmul1 = ops.MatMul().shard((layout("mp", ("sp", "dp")), layout(("sp", "dp"), "None")))
        self.relu1 = ops.ReLU().shard(((4, 1),))
        self.matmul2 = ops.MatMul().shard((layout2("None", "tp"), layout2("tp", "None")))
        self.relu2 = ops.ReLU().shard(((8, 1),))
        self.matmul3 = ops.MatMul()

    def construct(self, x):
        x = self.flatten(x)
        x = self.matmul1(x, self.fc1_weight)
        x = self.relu1(x)
        x = self.matmul2(x, self.fc2_weight)
        x = self.relu2(x)
        logits = self.matmul3(x, self.fc3_weight)
        return logits
```

Layout使用设备矩阵`(2, 2, 2)`表示8张卡，每轴命名为"dp"、"sp"、"mp"。`layout("mp", ("sp", "dp"))`表示第一维按mp切2份，第二维合并sp和dp共切4份。

#### 设备与数据切片映射表

| 设备坐标 (dp, sp, mp) | 输入 x 切片          | 权重 fc1_weight 切片         |
| ---------------------- | -------------------- | ---------------------------- |
| (0, 0, 0)              | `x[0:16, 0:196]`     | `fc1_weight[0:196, 0:512]`   |
| (0, 0, 1)              | `x[16:32, 0:196]`    | `fc1_weight[0:196, 0:512]`   |
| (0, 1, 0)              | `x[0:16, 196:392]`   | `fc1_weight[196:392, 0:512]` |
| (0, 1, 1)              | `x[16:32, 196:392]`  | `fc1_weight[196:392, 0:512]` |
| (1, 0, 0)              | `x[0:16, 392:588]`   | `fc1_weight[392:588, 0:512]` |
| (1, 0, 1)              | `x[16:32, 392:588]`  | `fc1_weight[392:588, 0:512]` |
| (1, 1, 0)              | `x[0:16, 588:784]`   | `fc1_weight[588:784, 0:512]` |
| (1, 1, 1)              | `x[16:32, 588:784]`  | `fc1_weight[588:784, 0:512]` |

#### 运行脚本

```bash
bash run_advanced.sh
```

### 高阶Mint算子并行实践

#### 定义网络

```python
import mindspore as ms
from mindspore import nn, mint
from mindspore.parallel import Layout

class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.flatten = mint.flatten
        self.fc1_weight = ms.Parameter(initializer("normal", [28*28, 512], ms.float32))
        self.fc2_weight = ms.Parameter(initializer("normal", [512, 512], ms.float32))
        self.fc3_weight = ms.Parameter(initializer("normal", [512, 10], ms.float32))
        layout = Layout((2, 2, 2), ("dp", "sp", "mp"))
        layout2 = Layout((8,), ("tp",))
        self.matmul1 = ms.parallel.shard(mint.matmul, in_strategy=(layout("mp", ("sp", "dp")), layout(("sp", "dp"), "None")))
        self.relu1 = ms.parallel.shard(mint.nn.functional.relu, in_strategy=((4, 1),))
        self.matmul2 = ms.parallel.shard(mint.matmul, in_strategy=(layout2("None", "tp"), layout2("tp", "None")))
        self.relu2 = ms.parallel.shard(mint.nn.functional.relu, in_strategy=((8, 1),))
        self.matmul3 = mint.matmul

    def construct(self, x):
        x = self.flatten(x)
        x = self.matmul1(x, self.fc1_weight)
        x = self.relu1(x, dim=0, keepdims=True)
        x = self.matmul2(x, self.fc2_weight)
        x = self.relu2(x, dim=0, keepdims=True)
        logits = self.matmul3(x, self.fc3_weight)
        return logits

net = Network()
```

高阶mint算子并行与高阶ops算子并行切分方法相同，仅使用`mindspore.parallel.shard`包装mint算子。

#### 设备与数据切片映射表

| 设备坐标 (dp, sp, mp) | 输入 x 切片          | 权重 fc1_weight 切片         |
| ---------------------- | -------------------- | ---------------------------- |
| (0, 0, 0)              | `x[0:16, 0:196]`     | `fc1_weight[0:196, 0:512]`   |
| (0, 0, 1)              | `x[16:32, 0:196]`    | `fc1_weight[0:196, 0:512]`   |
| (0, 1, 0)              | `x[0:16, 196:392]`   | `fc1_weight[196:392, 0:512]` |
| (0, 1, 1)              | `x[16:32, 196:392]`  | `fc1_weight[196:392, 0:512]` |
| (1, 0, 0)              | `x[0:16, 392:588]`   | `fc1_weight[392:588, 0:512]` |
| (1, 0, 1)              | `x[16:32, 392:588]`  | `fc1_weight[392:588, 0:512]` |
| (1, 1, 0)              | `x[0:16, 588:784]`   | `fc1_weight[588:784, 0:512]` |
| (1, 1, 1)              | `x[16:32, 588:784]`  | `fc1_weight[588:784, 0:512]` |

#### 并行配置

```python
from mindspore.parallel.auto_parallel import AutoParallel

parallel_net = AutoParallel(net, parallel_mode="semi_auto")
```

#### 执行网络

```python
for epoch in range(10):
    i = 0
    for image, _ in data_set:
        forward_logits = parallel_net(image)
        if i % 10 == 0:
            forward_sum = mint.sum(forward_logits).asnumpy()
            print("epoch: %s, step: %s, forward_sum is %s" % (epoch, i, forward_sum))
        i += 1
```

#### 运行脚本

```bash
bash run_advanced_mint.sh
```

日志输出示例：

```
epoch: 0 step: 0, forward_sum is 0.90023
epoch: 0 step: 10, forward_sum is 1.07679
epoch: 0 step: 20, forward_sum is 1.02521
epoch: 0 step: 30, forward_sum is 0.96682
epoch: 0 step: 40, forward_sum is 0.93158
epoch: 0 step: 50, forward_sum is 0.96655
```
