# 流水线并行

## 概述

随着神经网络规模的急剧增长，传统的数据并行与模型并行的混合方式因服务器间带宽较低而表现不佳。流水线并行通过将模型按Stage进行空间切分，每个Stage只执行网络一部分，大大节省了内存开销，同时缩小了通信域。

## 训练实践

### 配置分布式环境

```python
import mindspore as ms
from mindspore.communication import init

ms.set_context(mode=ms.GRAPH_MODE)
init()
ms.set_seed(1)
```

### 数据集加载

数据集加载方式与单卡一致：

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

### 网络定义

流水线并行的关键注意事项：

- Print/Summary/TensorDump算子必须放在配置了`pipeline_stage`属性的Cell中
- 网络输出不支持动态shape
- 最外层Cell使用`lazy_inline`装饰器可减少编译时间

```python
from mindspore import nn, ops, Parameter
from mindspore.common.initializer import initializer, HeUniform
import math

class MatMulCell(nn.Cell):
    def __init__(self, param=None, shape=None):
        super().__init__()
        if shape is None:
            shape = [28 * 28, 512]
        weight_init = HeUniform(math.sqrt(5))
        self.param = Parameter(initializer(weight_init, shape), name="param")
        if param is not None:
            self.param = param
        self.print = ops.Print()
        self.matmul = ops.MatMul()

    def construct(self, x):
        out = self.matmul(x, self.param)
        self.print("out is:", out)
        return out

class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()
        self.layer1 = MatMulCell()
        self.relu1 = nn.ReLU()
        self.layer2 = nn.Dense(512, 512)
        self.relu2 = nn.ReLU()
        self.layer3 = nn.Dense(512, 10)

    def construct(self, x):
        x = self.flatten(x)
        x = self.layer1(x)
        x = self.relu1(x)
        x = self.layer2(x)
        x = self.relu2(x)
        logits = self.layer3(x)
        return logits
```

### 训练网络定义

定义损失函数、优化器和训练过程，使用延迟初始化：

```python
import mindspore as ms
from mindspore import nn, ops
from mindspore.parallel.nn import Pipeline, PipelineGradReducer
from mindspore.nn.utils import no_init_parameters

with no_init_parameters():
    net = Network()
    optimizer = nn.SGD(net.trainable_params(), 1e-2)
    pp_grad_reducer = PipelineGradReducer(optimizer.parameters, opt_shard=False)

loss_fn = nn.CrossEntropyLoss()
net_with_loss = Pipeline(nn.WithLossCell(net, loss_fn), 4, stage_config={
    "_backbone.flatten": 0,
    "_backbone.layer1":  0,
    "_backbone.relu1":   0,
    "_backbone.layer2":  1,
    "_backbone.relu2":   1,
    "_backbone.layer3":  1
})
net_with_loss.set_train()

def forward_fn(inputs, target):
    loss = net_with_loss(inputs, target)
    return loss

grad_fn = ops.value_and_grad(forward_fn, None, optimizer.parameters)

@ms.jit
def train_one_step(inputs, target):
    loss, grads = grad_fn(inputs, target)
    grads = pp_grad_reducer(grads)
    optimizer(grads)
    return loss, grads
```

### 交错流水线调度

对于交错流水线，将非连续的模型层配置到不同Stage：

```python
net_with_loss = Pipeline(nn.WithLossCell(net, loss_fn), 4, stage_config={
    "_backbone.flatten": 0,
    "_backbone.layer1":  1,
    "_backbone.relu1":   0,
    "_backbone.layer2":  1,
    "_backbone.relu2":   0,
    "_backbone.layer3":  1
})
```

### 并行配置

```python
import mindspore as ms
from mindspore.parallel.auto_parallel import AutoParallel

parallel_net = AutoParallel(train_one_step, parallel_mode="semi_auto")
parallel_net.pipeline(stages=2)
```

交错调度的配置：

```python
parallel_net = AutoParallel(train_one_step, parallel_mode="semi_auto")
parallel_net.pipeline(stages=2, interleave=True)
```

### 训练循环

```python
for epoch in range(10):
    i = 0
    for data, label in data_set:
        loss, grads = parallel_net(data, label)
        if i % 10 == 0:
            print("epoch: %s, step: %s, loss is %s" % (epoch, i, loss))
        i += 1
```

## 推理实践

### 网络定义（含Pipeline Stage）

```python
import numpy as np
from mindspore import lazy_inline, nn, ops, Tensor, Parameter, sync_pipeline_shared_parameters

class VocabEmbedding(nn.Cell):
    def __init__(self, vocab_size, embedding_size):
        super().__init__()
        self.embedding_table = Parameter(Tensor(np.ones([vocab_size, embedding_size]), ms.float32),
                                         name='embedding_table')
        self.gather = ops.Gather()

    def construct(self, x):
        output = self.gather(self.embedding_table, x, 0)
        output = output.squeeze(1)
        return output, self.embedding_table.value()

class Head(nn.Cell):
    def __init__(self):
        super().__init__()
        self.matmul = ops.MatMul(transpose_b=True)

    def construct(self, state, embed):
        return self.matmul(state, embed)

class Network(nn.Cell):
    @lazy_inline
    def __init__(self):
        super().__init__()
        self.word_embedding = VocabEmbedding(vocab_size=32, embedding_size=32)
        self.layer1 = nn.Dense(32, 32)
        self.layer2 = nn.Dense(32, 32)
        self.head = Head()

    def construct(self, x):
        x, embed = self.word_embedding(x)
        x = self.layer1(x)
        x = self.layer2(x)
        x = self.head(x, embed)
        return x

net = Network()
net.word_embedding.pipeline_stage = 0
net.layer1.pipeline_stage = 1
net.layer2.pipeline_stage = 2
net.head.pipeline_stage = 3
```

### 推理网络封装

```python
from mindspore import nn, ops

class PipelineCellInference(nn.Cell):
    def __init__(self, network, micro_batch_num):
        super().__init__()
        self.network = network
        self.micro_batch_num = micro_batch_num
        self.concat = ops.Concat()

    def construct(self, x):
        ret = ()
        for i in range(self.micro_batch_num):
            micro_batch_size = x.shape[0] // self.micro_batch_num
            start = micro_batch_size * i
            end = micro_batch_size * (i + 1)

            micro_input = x[start:end]
            micro_output = self.network(micro_input)
            ret = ret + (micro_output,)

        ret = self.concat(ret)
        return ret

inference_network = PipelineCellInference(network=net, micro_batch_num=4)
inference_network.set_train(False)

parallel_net = AutoParallel(inference_network, parallel_mode="semi_auto")
parallel_net.dataset_strategy("full_batch")
parallel_net.pipeline(stages=4, output_broadcast=True)

input_ids = Tensor(np.random.randint(low=0, high=32, size=(8, 1)), ms.int32)
parallel_net.compile(input_ids)
sync_pipeline_shared_parameters(parallel_net)

logits = parallel_net(input_ids)
print(logits.asnumpy())
```

## 注意事项

- 流水线并行不支持自动混合精度
- 推荐使用`model.train`进行流水线并行训练，因为其内部封装了优化后的TrainOneStepCell逻辑
