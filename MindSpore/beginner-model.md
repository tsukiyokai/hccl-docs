# 网络构建

神经网络模型由神经网络层和Tensor操作构成，[mindspore.nn](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore.nn.html)提供了常见神经网络层的实现。在MindSpore中，[Cell](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.Cell.html)类是构建所有网络的基类，也是网络的基本单元。一个神经网络模型可表示为一个`Cell`，由不同的子`Cell`构成。使用这样的嵌套结构，可以简单地使用面向对象编程的思维，对神经网络结构进行构建和管理。

下面我们将构建一个用于Mnist数据集分类的神经网络模型。

```python
import mindspore
from mindspore import nn, ops
```

## 定义模型类

当我们定义神经网络时，可以继承`nn.Cell`类，在`__init__`方法中进行子`Cell`的实例化和状态管理，在`construct`方法中实现Tensor操作。

> `construct`意为神经网络（计算图）构建，相关内容详见[Graph Mode加速](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/accelerate_with_static_graph.html)。

```python
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
```

构建完成后，实例化`Network`对象，并查看其结构。

```python
model = Network()
print(model)
```

```
Network<
  (flatten): Flatten<>
  (dense_relu_sequential): SequentialCell<
    (0): Dense<input_channels=784, output_channels=512, has_bias=True>
    (1): ReLU<>
    (2): Dense<input_channels=512, output_channels=512, has_bias=True>
    (3): ReLU<>
    (4): Dense<input_channels=512, output_channels=10, has_bias=True>
    >
  >
```

我们构造一个输入数据，直接调用模型，可以获得一个二维的Tensor输出，其包含每个类别的原始预测值。

> `model.construct()`方法不可直接调用。

```python
X = ops.ones((1, 28, 28), mindspore.float32)
logits = model(X)
# print logits
logits
```

```
Tensor(shape=[1, 10], dtype=Float32, value=
[[-5.08734025e-04,  3.39190010e-04,  4.62840870e-03 ... -1.20305456e-03, -5.05689112e-03,  3.99264274e-03]])
```

在此基础上，通过一个`nn.Softmax`层实例获得预测概率。

```python
pred_probab = nn.Softmax(axis=1)(logits)
y_pred = pred_probab.argmax(1)
print(f"Predicted class: {y_pred}")
```

```
Predicted class: [4]
```

## 模型层

本节将分解上节构造的神经网络模型中的每一层。首先，构造一个shape为(3, 28, 28)的随机数据（即3个28x28的图像），依次通过每个神经网络层来观察其效果。

```python
input_image = ops.ones((3, 28, 28), mindspore.float32)
print(input_image.shape)
```

```
(3, 28, 28)
```

### nn.Flatten

实例化[nn.Flatten](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.Flatten.html)层，将28x28的2D张量转换为784大小的连续数组。

```python
flatten = nn.Flatten()
flat_image = flatten(input_image)
print(flat_image.shape)
```

```
(3, 784)
```

### nn.Dense

[nn.Dense](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.Dense.html)为全连接层，其使用权重和偏差对输入进行线性变换。

```python
layer1 = nn.Dense(in_channels=28*28, out_channels=20)
hidden1 = layer1(flat_image)
print(hidden1.shape)
```

```
(3, 20)
```

### nn.ReLU

[nn.ReLU](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.ReLU.html)层为网络加入非线性激活函数，帮助神经网络学习各种复杂的特征。

```python
print(f"Before ReLU: {hidden1}\n\n")
hidden1 = nn.ReLU()(hidden1)
print(f"After ReLU: {hidden1}")
```

```
Before ReLU: [[-0.04736331  0.2939465  -0.02713677 -0.30988005 -0.11504349 -0.11661264
   0.18007928  0.43213072  0.12091967 -0.17465964  0.53133243  0.12605792
   0.01825903  0.01287796  0.17238477 -0.1621131  -0.0080034  -0.24523425
  -0.10083733  0.05171938]
 [-0.04736331  0.2939465  -0.02713677 -0.30988005 -0.11504349 -0.11661264
   0.18007928  0.43213072  0.12091967 -0.17465964  0.53133243  0.12605792
   0.01825903  0.01287796  0.17238477 -0.1621131  -0.0080034  -0.24523425
  -0.10083733  0.05171938]
 [-0.04736331  0.2939465  -0.02713677 -0.30988005 -0.11504349 -0.11661264
   0.18007928  0.43213072  0.12091967 -0.17465964  0.53133243  0.12605792
   0.01825903  0.01287796  0.17238477 -0.1621131  -0.0080034  -0.24523425
  -0.10083733  0.05171938]]


After ReLU: [[0.         0.2939465  0.         0.         0.         0.
  0.18007928 0.43213072 0.12091967 0.         0.53133243 0.12605792
  0.01825903 0.01287796 0.17238477 0.         0.         0.
  0.         0.05171938]
 [0.         0.2939465  0.         0.         0.         0.
  0.18007928 0.43213072 0.12091967 0.         0.53133243 0.12605792
  0.01825903 0.01287796 0.17238477 0.         0.         0.
  0.         0.05171938]
 [0.         0.2939465  0.         0.         0.         0.
  0.18007928 0.43213072 0.12091967 0.         0.53133243 0.12605792
  0.01825903 0.01287796 0.17238477 0.         0.         0.
  0.         0.05171938]]
```

### nn.SequentialCell

[nn.SequentialCell](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.SequentialCell.html)是一个有序的Cell容器。输入Tensor将按照定义的顺序通过所有Cell。我们可以使用`nn.SequentialCell`来快速组合构造一个神经网络模型。

```python
seq_modules = nn.SequentialCell(
    flatten,
    layer1,
    nn.ReLU(),
    nn.Dense(20, 10)
)

logits = seq_modules(input_image)
print(logits.shape)
```

```
(3, 10)
```

### nn.Softmax

最后使用[nn.Softmax](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.Softmax.html)将神经网络最后一个全连接层返回的logits的值缩放到[0, 1]区间，表示每个类别的预测概率。`axis`指定的维度数值和为1。

```python
softmax = nn.Softmax(axis=1)
pred_probab = softmax(logits)
```

## 模型参数

网络内部神经网络层具有权重参数和偏置参数（如`nn.Dense`），这些参数会在训练过程中不断进行优化，可通过`model.parameters_and_names()`来获取参数名及对应的参数详情。

```python
print(f"Model structure: {model}\n\n")

for name, param in model.parameters_and_names():
    print(f"Layer: {name}\nSize: {param.shape}\nValues : {param[:2]} \n")
```

```
Model structure: Network<
  (flatten): Flatten<>
  (dense_relu_sequential): SequentialCell<
    (0): Dense<input_channels=784, output_channels=512, has_bias=True>
    (1): ReLU<>
    (2): Dense<input_channels=512, output_channels=512, has_bias=True>
    (3): ReLU<>
    (4): Dense<input_channels=512, output_channels=10, has_bias=True>
    >
  >


Layer: dense_relu_sequential.0.weight
Size: (512, 784)
Values : [[-0.01491369  0.00353318 -0.00694948 ...  0.01226766 -0.00014423
   0.00544263]
 [ 0.00212971  0.0019974  -0.00624789 ... -0.01214037  0.00118004
  -0.01594325]]

Layer: dense_relu_sequential.0.bias
Size: (512,)
Values : [0. 0.]

Layer: dense_relu_sequential.2.weight
Size: (512, 512)
Values : [[ 0.00565423  0.00354313  0.00637383 ... -0.00352688  0.00262949
   0.01157355]
 [-0.01284141  0.00657666 -0.01217057 ...  0.00318963  0.00319115
  -0.00186801]]

Layer: dense_relu_sequential.2.bias
Size: (512,)
Values : [0. 0.]

Layer: dense_relu_sequential.4.weight
Size: (10, 512)
Values : [[ 0.0087168  -0.00381866 -0.00865665 ... -0.00273731 -0.00391623
   0.00612853]
 [-0.00593031  0.0008721  -0.0060081  ... -0.00271535 -0.00850481
  -0.00820513]]

Layer: dense_relu_sequential.4.bias
Size: (10,)
Values : [0. 0.]
```

更多内置神经网络层详见[mindspore.nn API](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore.nn.html)。
