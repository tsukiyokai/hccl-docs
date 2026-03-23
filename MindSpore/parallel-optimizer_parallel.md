# 优化器并行

## 简介

在数据并行训练中，模型参数更新部分在各卡间存在冗余计算。优化器并行通过将优化器的计算量分散到数据并行维度的卡上，在大规模网络（如Bert、GPT）上可以有效减少内存消耗并提升性能。本教程以Ascend单机8卡为例进行说明。

## 样例代码说明

完整样例代码位置：distributed_optimizer_parallel

目录结构：

```
└─ sample_code
    ├─ distributed_optimizer_parallel
       ├── distributed_optimizer_parallel.py
       └── run.sh
```

其中distributed_optimizer_parallel.py定义网络结构和训练过程，run.sh是执行脚本。

## 配置分布式环境

通过context接口指定运行模式、设备、卡号等，并行脚本需init初始化HCCL或NCCL通信：

```python
import mindspore as ms
from mindspore.communication import init

ms.set_context(mode=ms.GRAPH_MODE)
init()
ms.set_seed(1)
```

## 数据集加载

优化器并行场景下，数据集加载与单卡方式一致：

```python
import os
import mindspore.dataset as ds

def create_dataset(batch_size):
    """create dataset"""
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

## 定义网络和优化器

网络结构与单卡基本一致，区别在于增加通信算子融合配置，需对网络和优化器进行延后初始化：

```python
from mindspore import nn
from mindspore.nn.utils import no_init_parameters

class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()
        self.layer1 = nn.Dense(28*28, 512)
        self.layer2 = nn.Dense(512, 512)
        self.layer3 = nn.Dense(512, 10)
        self.relu = nn.ReLU()

    def construct(self, x):
        x = self.flatten(x)
        x = self.layer1(x)
        x = self.relu(x)
        x = self.layer2(x)
        x = self.relu(x)
        logits = self.layer3(x)
        return logits

with no_init_parameters:
    net = Network()
    optimizer = nn.SGD(net.trainable_params(), 1e-2)
net.layer1.set_comm_fusion(0)
net.layer2.set_comm_fusion(1)
net.layer3.set_comm_fusion(2)
```

为减少通信成本，为不同层配置了通信融合。详见通信算子融合文档。

## 训练网络定义

定义损失函数和训练步骤，与单卡写法一致：

```python
import mindspore as ms
from mindspore import nn

optimizer = nn.SGD(net.trainable_params(), 1e-2)
loss_fn = nn.CrossEntropyLoss()

def forward_fn(data, target):
    logits = net(data)
    loss = loss_fn(logits, target)
    return loss, logits

grad_fn = ms.value_and_grad(forward_fn, None, net.trainable_params(), has_aux=True)

@ms.jit
def train_step(inputs, targets):
    (loss_value, _), grads = grad_fn(inputs, targets)
    optimizer(grads)
    return loss_value
```

## 并行配置

设置并行模式为semi_auto（半自动并行模式），开启优化器并行，配置hsdp：

```python
from mindspore.parallel.auto_parallel import AutoParallel

parallel_net = AutoParallel(train_step, parallel_mode="semi_auto")
parallel_net.hsdp()
```

## 训练循环

进行训练循环，外层循环为epoch数，内层循环遍历数据集：

```python
for epoch in range(10):
    i = 0
    for image, label in data_set:
        loss_output = parallel_net(image, label)
        if i % 10 == 0:
            print("epoch: %s, step: %s, loss is %s" % (epoch, i, loss_output))
        i += 1
```

## 运行单机8卡脚本

通过命令调用脚本，以msrun启动方式：

```bash
bash run.sh
```

训练完后，日志保存到log_output目录下：

```
└─ log_output
    ├─ scheduler.log
    ├─ worker_0.log
    ├─ worker_1.log
```

结果示例（保存在log_output/worker_*.log）：

```
epoch: 0, step: 0, loss is 2.3024087
epoch: 0, step: 10, loss is 2.2921634
epoch: 0, step: 20, loss is 2.278274
epoch: 0, step: 30, loss is 2.2537143
epoch: 0, step: 40, loss is 2.1638
epoch: 0, step: 50, loss is 1.984318
epoch: 0, step: 60, loss is 1.6061916
epoch: 0, step: 70, loss is 1.20966
epoch: 0, step: 80, loss is 0.98156196
epoch: 0, step: 90, loss is 0.77229893
epoch: 0, step: 100, loss is 0.6854114
```
