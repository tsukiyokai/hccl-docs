# 函数式自动微分

神经网络的训练主要使用反向传播算法，模型预测值（logits）与正确标签（label）送入损失函数（loss function）获得loss，然后进行反向传播计算，求得梯度（gradients），最终更新至模型参数（parameters）。自动微分能够计算可导函数在某点处的导数值，是反向传播算法的一般化。自动微分主要解决的问题是将一个复杂的数学运算分解为一系列简单的基本运算，该功能对用户屏蔽了大量的求导细节和过程，大大降低了框架的使用门槛。

MindSpore使用函数式自动微分的设计理念，提供更接近于数学语义的自动微分接口[mindspore.grad](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.grad.html)和[mindspore.value_and_grad](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.value_and_grad.html)。下面我们使用一个简单的单层线性变换模型进行介绍。

```python
import numpy as np
import mindspore
from mindspore import ops, nn, Tensor, Parameter
```

## 函数与计算图

计算图是用图论语言表示数学函数的一种方式，也是深度学习框架表达神经网络模型的统一方法。我们将根据下面的计算图构造计算函数和神经网络。

在这个模型中，$x$为输入，$y$为正确值，$w$和$b$是我们需要优化的参数。

```python
np.random.seed(42)
x = ops.ones(5, mindspore.float32)  # input tensor
y = ops.zeros(3, mindspore.float32)  # expected output
w = Parameter(Tensor(np.random.randn(5, 3), mindspore.float32), name='w') # weight
b = Parameter(Tensor(np.random.randn(3,), mindspore.float32), name='b') # bias
```

我们根据计算图描述的计算过程，构造计算函数。其中，[binary_cross_entropy_with_logits](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/ops/mindspore.ops.binary_cross_entropy_with_logits.html)是一个损失函数，计算预测值和目标值之间的二值交叉熵损失。

```python
def function(x, y, w, b):
    z = ops.matmul(x, w) + b
    loss = ops.binary_cross_entropy_with_logits(z, y, ops.ones_like(z), ops.ones_like(z))
    return loss
```

执行计算函数，可以获得计算的loss值。

```python
loss = function(x, y, w, b)
print(loss)
```

```
1.3423151
```

## 微分函数与梯度计算

为了优化模型参数，需要求参数对loss的导数：$\frac{\partial \operatorname{loss}}{\partial w}$和$\frac{\partial \operatorname{loss}}{\partial b}$，此时我们调用`mindspore.grad`函数，来获得`function`的微分函数。

这里使用了`grad`函数的两个入参，分别为：

- `fn`：待求导的函数。
- `grad_position`：指定求导输入位置的索引。

由于我们对$w$和$b$求导，因此配置其在`function`入参对应的位置`(2, 3)`。

> 使用`grad`获得微分函数是一种函数变换，即输入为函数，输出也为函数。

```python
grad_fn = mindspore.grad(function, (2, 3))
```

执行微分函数，即可获得$w$、$b$对应的梯度。

```python
grads = grad_fn(x, y, w, b)
print(grads)
```

```
(Tensor(shape=[5, 3], dtype=Float32, value=
 [[ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02]]), Tensor(shape=[3], dtype=Float32, value= [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02]))
```

## Stop Gradient

通常情况下，求导时会求loss对参数的导数，因此函数的输出只有loss一项。当我们希望函数输出多项时，微分函数会求所有输出项对参数的导数。此时如果想实现对某个输出项的梯度截断，或消除某个Tensor对梯度的影响，需要用到Stop Gradient操作。

这里我们将`function`改为同时输出loss和z的`function_with_logits`，获得微分函数并执行。

```python
def function_with_logits(x, y, w, b):
    z = ops.matmul(x, w) + b
    loss = ops.binary_cross_entropy_with_logits(z, y, ops.ones_like(z), ops.ones_like(z))
    return loss, z
```

```python
grad_fn = mindspore.grad(function_with_logits, (2, 3))
grads = grad_fn(x, y, w, b)
print(grads)
```

```
(Tensor(shape=[5, 3], dtype=Float32, value=
 [[ 1.32618928e+00,  1.01589143e+00,  1.04216456e+00],
  [ 1.32618928e+00,  1.01589143e+00,  1.04216456e+00],
  [ 1.32618928e+00,  1.01589143e+00,  1.04216456e+00],
  [ 1.32618928e+00,  1.01589143e+00,  1.04216456e+00],
  [ 1.32618928e+00,  1.01589143e+00,  1.04216456e+00]]), Tensor(shape=[3], dtype=Float32, value= [ 1.32618928e+00,  1.01589143e+00,  1.04216456e+00]))
```

可以看到求得$w$、$b$对应的梯度值发生了变化。此时如果想要屏蔽掉z对梯度的影响，即仍只求参数对loss的导数，可以使用[mindspore.ops.stop_gradient](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/ops/mindspore.ops.stop_gradient.html)接口，将梯度在此处截断。我们将`function`实现加入`stop_gradient`，并执行。

```python
def function_stop_gradient(x, y, w, b):
    z = ops.matmul(x, w) + b
    loss = ops.binary_cross_entropy_with_logits(z, y, ops.ones_like(z), ops.ones_like(z))
    return loss, ops.stop_gradient(z)
```

```python
grad_fn = mindspore.grad(function_stop_gradient, (2, 3))
grads = grad_fn(x, y, w, b)
print(grads)
```

```
(Tensor(shape=[5, 3], dtype=Float32, value=
 [[ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02]]), Tensor(shape=[3], dtype=Float32, value= [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02]))
```

可以看到，求得$w$、$b$对应的梯度值与初始`function`求得的梯度值一致。

## Auxiliary data

Auxiliary data意为辅助数据，是函数除第一个输出项外的其他输出。通常我们会将函数的loss设置为函数的第一个输出，其他的输出即为辅助数据。

`grad`和`value_and_grad`提供`has_aux`参数，当其设置为`True`时，可以自动实现前文手动添加`stop_gradient`的功能，满足返回辅助数据的同时不影响梯度计算的效果。

下面仍使用`function_with_logits`，配置`has_aux=True`，并执行。

```python
grad_fn = mindspore.grad(function_with_logits, (2, 3), has_aux=True)
```

```python
grads, (z,) = grad_fn(x, y, w, b)
print(grads, z)
```

```
(Tensor(shape=[5, 3], dtype=Float32, value=
 [[ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02]]), Tensor(shape=[3], dtype=Float32, value= [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02])) [ 3.8211915 -2.994512  -1.932323 ]
```

可以看到，求得$w$、$b$对应的梯度值与初始`function`求得的梯度值一致，同时z能够作为微分函数的输出返回。

## 神经网络梯度计算

前述章节主要根据计算图对应的函数介绍了MindSpore的函数式自动微分，但我们的神经网络构造是继承自面向对象编程范式的`nn.Cell`。接下来我们通过`Cell`构造同样的神经网络，利用函数式自动微分来实现反向传播。

首先我们继承`nn.Cell`构造单层线性变换神经网络。这里我们直接使用前文的$w$、$b$作为模型参数，使用`mindspore.Parameter`进行包装后，作为内部属性，并在`construct`内实现相同的Tensor操作。

```python
# Define model
class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.w = w
        self.b = b

    def construct(self, x):
        z = ops.matmul(x, self.w) + self.b
        return z
```

接下来我们实例化模型和损失函数。

```python
# Instantiate model
model = Network()
# Instantiate loss function
loss_fn = nn.BCEWithLogitsLoss()
```

完成后，为使用函数式自动微分，需要将神经网络和损失函数的调用封装为前向计算函数。

```python
# Define forward function
def forward_fn(x, y):
    z = model(x)
    loss = loss_fn(z, y)
    return loss
```

完成后，我们使用`value_and_grad`接口获得微分函数，用于计算梯度。

由于使用Cell封装神经网络模型，模型参数为Cell的内部属性，此时无需使用`grad_position`指定对函数输入求导，因此将其配置为`None`。对模型参数求导时，使用`weights`参数，通过`model.trainable_params()`方法从Cell中获取可求导参数。

```python
grad_fn = mindspore.value_and_grad(forward_fn, None, weights=model.trainable_params())
```

```python
loss, grads = grad_fn(x, y)
print(grads)
```

```
(Tensor(shape=[5, 3], dtype=Float32, value=
 [[ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02],
  [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02]]), Tensor(shape=[3], dtype=Float32, value= [ 3.26189250e-01,  1.58914644e-02,  4.21645455e-02]))
```

执行微分函数，可以看到梯度值和前文`function`求得的梯度值一致。
