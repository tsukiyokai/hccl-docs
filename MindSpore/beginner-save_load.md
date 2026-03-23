# 保存与加载

上一章节主要介绍了如何调整超参数，并进行网络模型训练。在训练网络模型的过程中，通常希望保存中间和最后的结果，用于微调（fine-tune）和后续的模型推理与部署，本章节我们将介绍如何保存与加载模型。

```python
import numpy as np
import mindspore
from mindspore import nn
from mindspore import Tensor
```

```python
def network():
    model = nn.SequentialCell(
                nn.Flatten(),
                nn.Dense(28*28, 512),
                nn.ReLU(),
                nn.Dense(512, 512),
                nn.ReLU(),
                nn.Dense(512, 10))
    return model
```

## 保存和加载模型权重

保存模型使用[mindspore.save_checkpoint](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.save_checkpoint.html)接口，传入网络和指定的保存路径：

```python
model = network()
mindspore.save_checkpoint(model, "model.ckpt")
```

为了加载模型权重，需要先创建相同模型的实例，然后使用`load_checkpoint`和`load_param_into_net`方法加载参数。

```python
model = network()
param_dict = mindspore.load_checkpoint("model.ckpt")
param_not_load, _ = mindspore.load_param_into_net(model, param_dict)
print(param_not_load)
```

```
[]
```

> - `param_not_load`是未被加载的参数列表，为空时代表所有参数均加载成功。
> - 当环境中安装有MindX DL（昇腾深度学习组件）6.0及以上版本时，默认启动MindIO加速CheckPoint功能，详情查看[MindIO介绍](https://www.hiascend.com/document/detail/zh/mindcluster/70rc1/clustersched/dlug/mindioacp001.html)。MindX DL在[此处](https://www.hiascend.com/developer/download/community/result?module=dl+cann)下载。

## 保存和加载MindIR

除Checkpoint外，MindSpore提供了云侧（训练）和端侧（推理）统一的中间表示（Intermediate Representation，IR）。可使用`export`接口直接将模型保存为MindIR（当前仅支持严格图模式）。

```python
mindspore.set_context(mode=mindspore.GRAPH_MODE, jit_syntax_level=mindspore.STRICT)
model = network()
inputs = Tensor(np.ones([1, 1, 28, 28]).astype(np.float32))
mindspore.export(model, inputs, file_name="model", file_format="MINDIR")
```

> MindIR同时保存了Checkpoint和模型结构，因此需要定义输入Tensor来获取输入shape。

已有的MindIR模型可以方便地通过`load`接口加载，传入[mindspore.nn.GraphCell](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.GraphCell.html)即可进行推理。

> `nn.GraphCell`仅支持图模式。

```python
graph = mindspore.load("model.mindir")
model = nn.GraphCell(graph)
outputs = model(inputs)
print(outputs.shape)
```

```
(1, 10)
```

### 语法支持范围

并不是所有的Python语法和数据类型都支持MindIR导出，若不在支持范围内，导出时会报错。

1. MindIR导出仅支持STRICT级别的基础语法，详细的支持范围，可参考[静态图语法支持](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/compile/static_graph.html)。

2. 返回值的数据类型只支持：
   - Python内置类型：`int`、`float`、`bool`、`str`、`tuple`、`list`。
   - MindSpore框架内置类型：[Tensor](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.Tensor.html)、[Parameter](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.Parameter.html)、[COOTensor](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.COOTensor.html)、[CSRTensor](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.CSRTensor.html)。

   例如下面的程序，返回值类型是[mindspore.dtype](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.dtype.html)，不在支持范围内，MindIR导出的时候就会报错。

```python
import mindspore
from mindspore import nn, Tensor

class Model(nn.Cell):

    def construct(self, x: Tensor) -> mindspore.dtype:
        return x.dtype
```

3. `nn.Cell`的`construct()`方法中，不支持使用[mindspore.mint](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore.mint.html)包下的随机数生成接口，如`mint.rand`、`mint.randn`、`mint.randint`、`mint.randperm`。（建议改为使用[mindspore.ops](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore.ops.html)包下的随机数生成接口）。

4. `Parameter`对象只能定义在`nn.Cell`的`__init__()`方法中或者作为函数的输入参数，否则MindIR不支持导出该`Parameter`。例如下面的程序，有一个`Parameter`是全局变量，导出时会报错不支持。

```python
import mindspore
from mindspore import Parameter, nn

# Parameter在nn.Cell外创建，并作为全局变量被Model使用。
global_param = Parameter([1, 2, 3], name='global_param')

class Model(nn.Cell):

    def __init__(self):
        super().__init__()
        # Parameter定义在nn.Cell的__init__()方法中，支持导出。
        self.bias = Parameter([0, 1, -1])

    def construct(self, x: Parameter):  # Parameter是函数的入参，支持导出。
        # global_param是全局变量，导出时会报错。
        return x + global_param + self.bias

model = Model()
param = Parameter([1, 2, 3], name='input_param')
mindspore.export(model, param, file_name="model", file_format="MINDIR")
```
