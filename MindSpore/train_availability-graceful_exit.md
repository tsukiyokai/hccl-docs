# 进程优雅退出

## 概述

当训练集群中存在亚健康设备时，在亚健康设备发生故障之前完成checkpoint保存并结束集群训练进程，可以有效避免集群损坏时的权重数据丢失问题。同时，这也避免了训练恢复时的数据回滚问题，减少训练资源浪费。

本文档以在第一个训练step时检测到退出配置信息并提前结束训练进程为例，介绍进程优雅退出功能的使用方法。

## 数据和模型准备

### 准备数据

下载MNIST数据集，并解压到项目目录：

```bash
wget http://mindspore-website.obs.cn-north-4.myhuaweicloud.com/notebook/datasets/MNIST_Data.zip
unzip MNIST_Data.zip
```

### 模型定义

```python
import os
import mindspore as ms
import mindspore.context as context
import mindspore.dataset as ds
import mindspore.dataset.transforms as C
import mindspore.dataset.vision as CV
import mindspore.nn as nn
from mindspore.common import dtype as mstype
from mindspore.dataset.vision import Inter
from mindspore.train import Accuracy
from mindspore.train import Model, LossMonitor
from mindspore.train.callback import OnRequestExit
from mindspore.common.initializer import TruncatedNormal
from mindspore.communication.management import init
from mindspore.parallel.auto_parallel import AutoParallel
from mindspore.nn.utils import no_init_parameters

context.set_context(mode=context.GRAPH_MODE)

# 数据集
DATASET_PATH = "./MNIST_Data"

def create_dataset(data_path, batch_size=32, repeat_size=1,
                   num_parallel_workers=1):
    """
    create dataset for train or test
    """
    # define dataset
    mnist_ds = ds.MnistDataset(data_path)

    resize_height, resize_width = 32, 32
    rescale = 1.0 / 255.0
    shift = 0.0
    rescale_nml = 1 / 0.3081
    shift_nml = -1 * 0.1307 / 0.3081

    # define map operations
    resize_op = CV.Resize((resize_height, resize_width), interpolation=Inter.LINEAR)
    rescale_nml_op = CV.Rescale(rescale_nml, shift_nml)
    rescale_op = CV.Rescale(rescale, shift)
    hwc2chw_op = CV.HWC2CHW()
    type_cast_op = C.TypeCast(mstype.int32)

    # apply map operations on images
    mnist_ds = mnist_ds.map(operations=type_cast_op, input_columns="label", num_parallel_workers=num_parallel_workers)
    mnist_ds = mnist_ds.map(operations=resize_op, input_columns="image", num_parallel_workers=num_parallel_workers)
    mnist_ds = mnist_ds.map(operations=rescale_op, input_columns="image", num_parallel_workers=num_parallel_workers)
    mnist_ds = mnist_ds.map(operations=rescale_nml_op, input_columns="image", num_parallel_workers=num_parallel_workers)
    mnist_ds = mnist_ds.map(operations=hwc2chw_op, input_columns="image", num_parallel_workers=num_parallel_workers)

    # apply DatasetOps
    buffer_size = 10000
    mnist_ds = mnist_ds.shuffle(buffer_size=buffer_size)
    mnist_ds = mnist_ds.batch(batch_size, drop_remainder=True)
    mnist_ds = mnist_ds.repeat(repeat_size)

    return mnist_ds

# 定义网络模型
def conv(in_channels, out_channels, kernel_size, stride=1, padding=0):
    """weight initial for conv layer"""
    weight = weight_variable()
    return nn.Conv2d(in_channels, out_channels,
                     kernel_size=kernel_size, stride=stride, padding=padding,
                     weight_init=weight, has_bias=False, pad_mode="valid")

def fc_with_initialize(input_channels, out_channels):
    """weight initial for fc layer"""
    weight = weight_variable()
    bias = weight_variable()
    return nn.Dense(input_channels, out_channels, weight, bias)

def weight_variable():
    """weight initial"""
    return TruncatedNormal(0.02)

class LeNet5(nn.Cell):
    def __init__(self, num_class=10, channel=1):
        super(LeNet5, self).__init__()
        self.num_class = num_class
        self.conv1 = conv(channel, 6, 5)
        self.conv2 = conv(6, 16, 5)
        self.fc1 = fc_with_initialize(16 * 5 * 5, 120)
        self.fc2 = fc_with_initialize(120, 84)
        self.fc3 = fc_with_initialize(84, self.num_class)
        self.relu = nn.ReLU()
        self.max_pool2d = nn.MaxPool2d(kernel_size=2, stride=2)
        self.flatten = nn.Flatten()

    def construct(self, x):
        x = self.conv1(x)
        x = self.relu(x)
        x = self.max_pool2d(x)
        x = self.conv2(x)
        x = self.relu(x)
        x = self.max_pool2d(x)
        x = self.flatten(x)
        x = self.fc1(x)
        x = self.relu(x)
        x = self.fc2(x)
        x = self.relu(x)
        x = self.fc3(x)
        return x
```

## 环境变量以及Callback函数

### 环境变量

开启进程优雅退出功能，需要设置环境变量：

```bash
export MS_ENABLE_GRACEFUL_EXIT=1
```

该环境变量可以控制同步算子入图，来保证所有训练进程可以同步退出。

### Callback函数

设置 `OnRequestExit` Callback函数，并通过参数 `config_file` 指定优雅退出配置文件路径。该函数在训练进程的每个step开始时检查是否存在优雅退出配置文件。如果存在配置文件且文件中的关键字 `GracefulExit` 值为 `1`，则会在step结束时保存checkpoint文件并退出训练进程。

配置文件中的 `GracefulExit` 关键字在训练过程中动态配置，通常在识别到训练集群存在亚健康设备时修改。

```python
# json文件中关键字：{"GracefulExit": 1}
config_json = r"./graceful_exit.json"

# 设置callback函数
cb = OnRequestExit(file_name="LeNet", config_file=config_json)

def graceful_exit_case():
    # init
    device_num = 8
    context.set_context(mode=context.GRAPH_MODE)
    ms.set_device("Ascend")

    init()

    # build
    with no_init_parameters():
        network = LeNet5(10)
        net_opt = nn.Momentum(network.trainable_params(), 0.01, 0.9)
    ds_train = create_dataset(os.path.join(DATASET_PATH, "train"), 32, 1)
    net_loss = nn.SoftmaxCrossEntropyWithLogits(sparse=True, reduction="mean")
    parallel_net = AutoParallel(network, parallel_mode='semi_auto')
    model = Model(parallel_net, net_loss, net_opt, metrics={"Accuracy": Accuracy()})

    # graceful exit json file：{"GracefulExit": 1}
    reset_json = r"./graceful_exit.json"

    # callback
    cb = OnRequestExit(file_name="LeNet", config_file=reset_json)
    # train
    model.train(1, ds_train, callbacks=[cb, LossMonitor()], dataset_sink_mode=False)
```

## 启动训练

使用 `msrun` 命令启动训练：

```bash
msrun --worker_num=8 --local_worker_num=8 --master_addr=127.0.0.1 --master_port=10970 --join=True --log_dir=./comm_subgraph_logs graceful_exit_case.py
```

## 结果分析

训练结束后，日志中会有WARNING打印：`Graceful exit is triggered, stop training`。在当前执行目录下，会生成 `rank_0` 至 `rank_7` 共8个目录，每个目录下都有一个 `LeNet_train.ckpt` 文件（若callback中配置了保存checkpoint）：

```
./rank_0
├── LeNet_train.ckpt
└── LeNet_train.mindir
./rank_1
├── LeNet_train.ckpt
└── LeNet_train.mindir
...
./rank_7
├── LeNet_train.ckpt
└── LeNet_train.mindir
```

## 说明

如果没有重写TrainOneStepCell，则只需要配置 `MS_ENABLE_GRACEFUL_EXIT` 环境变量和 `OnRequestExit` callback函数，并在训练的某一时刻按需修改优雅退出配置文件，即可实现进程优雅退出功能。

如果网络模型需要重写TrainOneStepCell，则：

1. 继承父类TrainOneStepCell，在construct方法里面添加如下 `if` 条件分支代码：

```python
class TrainOneStepCellWithABC(TrainOneStepCell):
    def __init__(self, ...):
        ...

    def construct(self, *inputs):
        ...
        grads = self.grad(self.network, self.weights)(*inputs, sens)
        if self.use_graceful_exit:
            grads = self.graceful_exit.exit_by_request(grads, self.init_param, self.exit_param)

        loss = F.depend(loss, self.optimizer(grads))
        ...
```

2. 没有继承父类TrainOneStepCell，需要在 `__init__` 方法里面新增如下代码，并在 `construct` 方法里面调用：

```python
from mindspore.utils import ExitByRequest

class TrainOneStepCellWithABC(Cell):
    def __init__(self, ...):
        ...
        self.use_graceful_exit = os.environ.get("MS_ENABLE_GRACEFUL_EXIT") == "1"
        if self.use_graceful_exit:
            self.graceful_exit = ExitByRequest()
            self.exit_param = Parameter(Tensor(False, mstype.bool_), name="graceful_exit")
            self.init_param = Parameter(Tensor([0], mstype.int32), name="graceful_init")

    def construct(self, *inputs):
        ...
        if self.use_graceful_exit:
            grads = self.graceful_exit.exit_by_request(grads, self.init_param, self.exit_param)
        loss = F.depend(loss, self.optimizer(grads))
        ...
```
