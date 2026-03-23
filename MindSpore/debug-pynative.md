# 动态图调试

## 概述

在MindSpore中，动态图模式（PyNative模式）支持按Python语法执行。该模式便于调试，支持单算子执行、逐行语句执行、查看中间变量等调试手段。

## 基本方法

动态图下的调试面临多线程异步行为的挑战。调试时通常需要打断点、查看日志、查看变量值、单步执行等步骤。

## 同步模式定位问题

### 设置同步模式

使用以下代码开启同步执行模式：

```python
import mindspore
mindspore.runtime.launch_blocking()
```

同步模式下可以获得准确的调用栈。根据错误位置分类：

1. Python语法问题：调用栈与MindSpore API无关
2. 前向API错误：检查输入数据的shape和dtype
3. 反向传播错误：检查输入/输出类型和求导接口

## 单步调试定位问题

### 设置断点

通过IDE或在Python脚本中插入pdb：

```python
def some_function():
    x = 10
    import pdb; pdb.set_trace()  # 设置断点
    y = x + 1
    return y
```

### 单步执行

使用IDE工具栏的"Step Over"、"Step Into"、"Step Out"，或在命令行使用 `n(next)` 和 `s(step)` 命令。

### 日志记录

通过 `GLOG_v` 环境变量控制日志级别（默认值：2）：

- 0-DEBUG
- 1-INFO
- 2-WARNING
- 3-ERROR
- 4-CRITICAL

### 常见PDB调试命令

| 命令           | 功能                       |
| -------------- | -------------------------- |
| c (continue)   | 继续执行至下一个断点       |
| n (next)       | 执行下一行代码             |
| s (step)       | 执行下一行代码，进入函数内部 |
| l (list)       | 显示源代码周围行           |
| p (print)      | 打印表达式值               |
| q (quit)       | 退出调试器                 |
| h (help)       | 显示帮助信息               |

## 反向问题定位

### 使用Tensor的register_hook

```python
import mindspore
from mindspore import Tensor

def hook_fn(grad):
    return grad * 2

def hook_test(x, y):
    z = x * y
    z.register_hook(hook_fn)
    z = z * y
    return z

ms_grad = mindspore.value_and_grad(hook_test, grad_position=(0,1))
output = ms_grad(Tensor(1, mindspore.float32), Tensor(2, mindspore.float32))
print(output)
```

### 使用HookBackward查看梯度

```python
import mindspore
from mindspore import ops, Tensor

def hook_fn(grad):
    print(grad)

hook = ops.HookBackward(hook_fn)

def hook_test(x, y):
    z = x * y
    z = hook(z)
    z = z * y
    return z

def backward(x, y):
    return mindspore.value_and_grad(hook_test, grad_position=(0,1))(x, y)

output = backward(Tensor(1, mindspore.float32), Tensor(2, mindspore.float32))
print(output)
```

### 使用Cell的register_backward_hook

```python
import numpy as np
import mindspore
from mindspore import Tensor, nn

def backward_hook_fn(cell_id, grad_input, grad_output):
    print("backward input: ", grad_input)
    print("backward output: ", grad_output)

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.relu = nn.ReLU()
        self.handle = self.relu.register_backward_hook(backward_hook_fn)

    def construct(self, x):
        x = x + x
        x = self.relu(x)
        return x

net = Net()
output = mindspore.value_and_grad(net, grad_position=(0,))(Tensor(np.ones([1]).astype(np.float32)))
print(output)
```
