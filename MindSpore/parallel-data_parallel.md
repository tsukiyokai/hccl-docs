# 数据并行

## 概述

数据并行是最常见的分布式训练方式。训练数据被划分到多个计算节点（如多卡或多设备），每个节点使用相同的模型独立处理自己的数据子集，执行前向传播和反向传播，然后同步所有节点的梯度后更新模型参数。

## 示例代码

示例使用Ascend单机8卡环境，目录结构如下：

```
└─ sample_code
    ├─ distributed_data_parallel
       ├── distributed_data_parallel.py
       └── run.sh
```

`distributed_data_parallel.py`定义了网络结构和训练过程，`run.sh`用于执行训练。

## 配置分布式环境

```python
import mindspore as ms
from mindspore.communication import init

ms.set_context(mode=ms.GRAPH_MODE)
ms.set_auto_parallel_context(parallel_mode=ms.ParallelMode.DATA_PARALLEL, gradients_mean=True)
init()
ms.set_seed(1)
```

`gradients_mean=True`参数用于开启梯度聚合。框架会对梯度执行AllReduce(op=ReduceOp.SUM)，当`gradients_mean`为True时再计算均值。

## 数据集加载

数据并行需要并行加载数据。以MNIST数据集为例：

```python
import mindspore.dataset as ds
from mindspore.communication import get_rank, get_group_size

rank_id = get_rank()
rank_size = get_group_size()
dataset = ds.MnistDataset(dataset_path, num_shards=rank_size, shard_id=rank_id)
```

关键参数：

- `get_rank()`：获取当前设备在集群中的ID
- `get_group_size()`：获取集群总大小
- `num_shards`：卡数
- `shard_id`：逻辑序号

> 建议：每张卡应加载相同的数据集文件，以避免影响计算精度。

完整的数据集处理代码：

```python
import os
import mindspore.dataset as ds
from mindspore.communication import get_rank, get_group_size

def create_dataset(batch_size):
    dataset_path = os.getenv("DATA_PATH")
    rank_id = get_rank()
    rank_size = get_group_size()
    dataset = ds.MnistDataset(dataset_path, num_shards=rank_size, shard_id=rank_id)
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

## 网络定义

数据并行模式下的网络定义与单卡实现完全相同：

```python
from mindspore import nn

class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()
        self.dense_relu_sequential = nn.SequentialCell(
            nn.Dense(28*28, 512, weight_init="normal", bias_init="zeros"),
            nn.ReLU(),
            nn.Dense(512, 512, weight_init="normal", bias_init="zeros"),
            nn.ReLU(),
            nn.Dense(512, 10, weight_init="normal", bias_init="zeros")
        )

    def construct(self, x):
        x = self.flatten(x)
        logits = self.dense_relu_sequential(x)
        return logits

net = Network()
```

## 训练网络

定义损失函数、优化器和训练过程。添加`DistributedGradReducer`以聚合各卡梯度：

```python
from mindspore import nn
import mindspore as ms

loss_fn = nn.CrossEntropyLoss()
optimizer = nn.SGD(net.trainable_params(), 1e-2)

def forward_fn(data, label):
    logits = net(data)
    loss = loss_fn(logits, label)
    return loss, logits

grad_fn = ms.value_and_grad(forward_fn, None, net.trainable_params(), has_aux=True)
grad_reducer = nn.DistributedGradReducer(optimizer.parameters)

for epoch in range(10):
    i = 0
    for data, label in data_set:
        (loss, _), grads = grad_fn(data, label)
        grads = grad_reducer(grads)
        optimizer(grads)
        if i % 10 == 0:
            print("epoch: %s, step: %s, loss is %s" % (epoch, i, loss))
        i += 1
```

> 也可以使用`Model.train()`方法进行训练。

## 运行单机8卡脚本

使用msrun执行分布式训练脚本：

```bash
bash run.sh
```

日志文件保存在`log_output`目录下，结构如下：

```
└─ log_output
    └─ 1
        ├─ rank.0
        |   └─ stdout
        ├─ rank.1
        |   └─ stdout
```

`log_output/1/rank.*/stdout`中的loss输出示例：

```
epoch: 0 step: 0, loss is 2.3026438
epoch: 0 step: 50, loss is 2.2963896
epoch: 0 step: 100, loss is 2.2882829
epoch: 0 step: 150, loss is 2.2822685
```
