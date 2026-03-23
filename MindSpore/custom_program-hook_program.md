# Hook编程

调试深度学习网络需要捕获中间层算子的数据变化。MindSpore在动态图模式下提供了Hook功能，使用Hook功能可以捕获中间层算子的输入、输出数据以及反向梯度。

动态图模式提供五种Hook形式：HookBackward算子和四种Cell注册方法（register_forward_pre_hook、register_forward_hook、register_backward_pre_hook、register_backward_hook）。

## HookBackward算子

HookBackward将Hook功能实现为算子形式。正向执行时，HookBackward将输入不做修改原样输出；反向传播时，注册的Hook函数捕获梯度。

```python
import mindspore as ms
from mindspore import ops

ms.set_context(mode=ms.PYNATIVE_MODE)

def hook_fn(grad_out):
    """打印梯度"""
    print("hook_fn print grad_out:", grad_out)

hook = ops.HookBackward(hook_fn)
def hook_test(x, y):
    z = x * y
    z = hook(z)
    z = z * y
    return z

def net(x, y):
    return ms.grad(hook_test, grad_position=(0, 1))(x, y)

output = net(ms.Tensor(1, ms.float32), ms.Tensor(2, ms.float32))
print("output:", output)
```

## Cell对象的register_forward_pre_hook功能

使用`register_forward_pre_hook`函数注册Hook函数，捕获正向传入Cell对象的数据。

```python
def forward_pre_hook_fn(cell, inputs):
    print("forward inputs: ", inputs)
```

Hook函数接收cell对象和inputs参数。可自定义对输入数据的操作，如查看、打印或返回新的输入数据。

```python
import numpy as np
import mindspore as ms
import mindspore.nn as nn

ms.set_context(mode=ms.PYNATIVE_MODE)

def forward_pre_hook_fn(cell, inputs):
    print("forward inputs: ", inputs)
    input_x = inputs[0]
    return input_x

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.relu = nn.ReLU()
        self.handle = self.relu.register_forward_pre_hook(forward_pre_hook_fn)

    def construct(self, x, y):
        x = x + y
        x = self.relu(x)
        return x

net = Net()
grad_net = ms.grad(net, grad_position=(0, 1))

x = ms.Tensor(np.ones([1]).astype(np.float32))
y = ms.Tensor(np.ones([1]).astype(np.float32))

output = net(x, y)
print(output)
gradient = grad_net(x, y)
print(gradient)
net.handle.remove()
gradient = grad_net(x, y)
print(gradient)
```

若在Hook函数中直接返回新创建数据而非原始数据计算结果，梯度反向传播将在该Cell对象上截止。

```python
def forward_pre_hook_fn(cell, inputs):
    print("forward inputs: ", inputs)
    return ms.Tensor(np.ones([1]).astype(np.float32))
```

注意：不建议在Cell的`construct`函数中调用`register_forward_pre_hook`和`handle.remove()`，避免脚本切换图模式时失败。

## Cell对象的register_forward_hook功能

使用`register_forward_hook`函数捕获正向传入的数据和Cell对象的输出数据。

```python
def forward_hook_fn(cell, inputs, outputs):
    print("forward inputs: ", inputs)
    print("forward outputs: ", outputs)
```

hook函数包含cell、inputs和outputs三个参数。

```python
import numpy as np
import mindspore as ms
import mindspore.nn as nn

ms.set_context(mode=ms.PYNATIVE_MODE)

def forward_hook_fn(cell, inputs, outputs):
    print("forward inputs: ", inputs)
    print("forward outputs: ", outputs)
    outputs = outputs + outputs
    return outputs

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.relu = nn.ReLU()
        self.handle = self.relu.register_forward_hook(forward_hook_fn)

    def construct(self, x, y):
        x = x + y
        x = self.relu(x)
        return x

net = Net()
grad_net = ms.grad(net, grad_position=(0, 1))

x = ms.Tensor(np.ones([1]).astype(np.float32))
y = ms.Tensor(np.ones([1]).astype(np.float32))

gradient = grad_net(x, y)
print(gradient)
net.handle.remove()
gradient = grad_net(x, y)
print(gradient)
```

## Cell对象的register_backward_pre_hook功能

使用`register_backward_pre_hook`函数捕获反向传播时与Cell相关的梯度。

```python
def backward_pre_hook_function(cell, grad_output):
    print(grad_output)
```

hook函数包含cell和grad_output参数，grad_output为反向传入的梯度。返回新梯度时必须是tuple形式。

```python
import numpy as np
import mindspore as ms
import mindspore.nn as nn

ms.set_context(mode=ms.PYNATIVE_MODE)

def backward_pre_hook_function(cell, grad_output):
    print(grad_output)

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.conv = nn.Conv2d(1, 2, kernel_size=2, stride=1, padding=0,
                              weight_init="ones", pad_mode="valid")
        self.bn = nn.BatchNorm2d(2, momentum=0.99, eps=0.00001,
                                 gamma_init="ones")
        self.handle = self.bn.register_backward_pre_hook(backward_pre_hook_function)
        self.relu = nn.ReLU()

    def construct(self, x):
        x = self.conv(x)
        x = self.bn(x)
        x = self.relu(x)
        return x

net = Net()
grad_net = ms.grad(net)
output = grad_net(ms.Tensor(np.ones([1, 1, 2, 2]).astype(np.float32)))
print(output)
net.handle.remove()
output = grad_net(ms.Tensor(np.ones([1, 1, 2, 2]).astype(np.float32)))
print("-------------\n", output)
```

## Cell对象的register_backward_hook功能

使用`register_backward_hook`函数捕获反向传入和反向输出梯度。

```python
def backward_hook_function(cell, grad_input, grad_output):
    print(grad_input)
    print(grad_output)
```

hook函数包含cell、grad_input和grad_output三个参数。grad_output为反向传入梯度，grad_input为反向输出梯度。返回新梯度时必须是tuple形式。

```python
import numpy as np
import mindspore as ms
import mindspore.nn as nn

ms.set_context(mode=ms.PYNATIVE_MODE)

def backward_hook_function(cell, grad_input, grad_output):
    print(grad_input)
    print(grad_output)

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.conv = nn.Conv2d(1, 2, kernel_size=2, stride=1, padding=0,
                              weight_init="ones", pad_mode="valid")
        self.bn = nn.BatchNorm2d(2, momentum=0.99, eps=0.00001,
                                 gamma_init="ones")
        self.handle = self.bn.register_backward_hook(backward_hook_function)
        self.relu = nn.ReLU()

    def construct(self, x):
        x = self.conv(x)
        x = self.bn(x)
        x = self.relu(x)
        return x

net = Net()
grad_net = ms.grad(net)
output = grad_net(ms.Tensor(np.ones([1, 1, 2, 2]).astype(np.float32)))
print(output)
net.handle.remove()
output = grad_net(ms.Tensor(np.ones([1, 1, 2, 2]).astype(np.float32)))
print("-------------\n", output)
```

## Cell对象使用多个hook功能

四种Hook函数同时作用于同一Cell对象时，如果forward hook中添加了其他算子处理数据，这些新增算子会参与正向计算，但其反向梯度不在backward hook的捕获范围内。

```python
import numpy as np
import mindspore as ms
import mindspore.nn as nn

ms.set_context(mode=ms.PYNATIVE_MODE)

def forward_pre_hook_fn(cell, inputs):
    print("forward inputs: ", inputs)
    input_x = inputs[0]
    return input_x

def forward_hook_fn(cell, inputs, outputs):
    print("forward inputs: ", inputs)
    print("forward outputs: ", outputs)
    outputs = outputs + outputs
    return outputs

def backward_pre_hook_fn(cell, grad_output):
    print("grad input: ", grad_output)

def backward_hook_fn(cell, grad_input, grad_output):
    print("grad input: ", grad_output)
    print("grad output: ", grad_input)

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.relu = nn.ReLU()
        self.handle = self.relu.register_forward_pre_hook(forward_pre_hook_fn)
        self.handle2 = self.relu.register_forward_hook(forward_hook_fn)
        self.handle3 = self.relu.register_backward_pre_hook(backward_pre_hook_fn)
        self.handle4 = self.relu.register_backward_hook(backward_hook_fn)

    def construct(self, x, y):
        x = x + y
        x = self.relu(x)
        return x

net = Net()
grad_net = ms.grad(net, grad_position=(0, 1))
gradient = grad_net(ms.Tensor(np.ones([1]).astype(np.float32)),
                    ms.Tensor(np.ones([1]).astype(np.float32)))
print(gradient)
```

backward_pre_hook中捕获的梯度是反向传入Cell对象的原始梯度，不包含forward hook新增算子的梯度。backward_hook中捕获的输入、输出梯度也仅限于原始Cell对象，不影响forward hook函数的作用范围。
