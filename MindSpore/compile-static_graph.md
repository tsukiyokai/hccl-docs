# 图模式编程介绍

## 概述

在即时编译（JIT）模式下，Python代码先被编译成静态计算图，再执行该图。MindSpore通过源码转换方式将Python源码转换为中间表达IR（MindIR），并进行优化后在硬件设备上执行。

将Python源码转换为中间表示的方法主要有三种：基于抽象语法树（AST）、基于字节码（ByteCode）和基于算子调用追踪（Trace）。

静态图执行包含Define和Run两个阶段，封装在Cell的`__call__`方法中：`model(inputs) = model.compile(inputs) + model.construct(inputs)`

## 即时编译下的常量与变量

### 常量定义

编译期内可以获取到值的量。产生场景包括：

- 作为图模式输入的标量、列表和元组（不使用mutable接口）
- 图模式内生成的标量或Tensor
- 常量运算的结果

### 变量定义

编译期内无法获取到值的量。产生场景包括：

- 所有mutable接口的返回值
- 作为静态图输入的Tensor类型参数
- 通过变量计算得到的结果

## 数据类型支持

### Python内置数据类型

#### Number

支持`int`、`float`、`bool`，不支持`complex`。

支持在网络中定义Number和进行类型转换：

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self, x):
      out1 = int(11.1)
      out2 = int(mindspore.tensor([10]))
      return out1, out2

net = Net()
res = net(mindspore.tensor(2))
```

#### String

支持字符串构造、连接、截取、成员运算和格式化输出：

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      var1 = 'Hello!'
      var2 = "MindSpore"
      var3 = str(123)
      var4 = "{} is {}".format("string", var3)
      return var1[0], var2[4:9], var1 + var2
```

#### List

支持列表创建、返回、作为输入。支持索引取值、索引赋值、append、clear、extend、pop、reverse、insert等内置方法。

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      x = [1, 2, 3]
      x.append(4)
      return x

net = Net()
output = net()
```

#### Tuple

支持元组构造、索引取值（支持int、slice、Tensor索引）、连接组合：

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      x = (1, 2, 3)
      y = (4, 5, 6)
      return x + y, x * 2
```

#### Dictionary

支持字典构造，键支持String、Number、常量Tensor及只包含这些类型的Tuple。支持keys、values、items、get、clear、has_key、update、fromkeys等接口：

```python
import mindspore
from mindspore import nn

x = {"a": mindspore.tensor([1, 2, 3]), "b": mindspore.tensor([4, 5, 6])}

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      x_keys = x.keys()
      x_values = x.values()
      value_a = x.get("a")
      return x_keys, x_values, value_a
```

### MindSpore自定义数据类型

#### Tensor

支持在静态图模式下创建和使用Tensor：

```python
import mindspore
from mindspore import nn
import numpy as np

class Net(nn.Cell):
   @mindspore.jit
   def construct(self, x):
      return mindspore.tensor(x, dtype=mindspore.float32)
```

#### Primitive

支持在construct中构造Primitive及其子类的实例。不支持在网络中调用相关属性和接口。

#### Cell

支持在网络中构造Cell及其子类的实例，参数仅支持位置参数方式传入。

#### Parameter

Parameter代表需要在训练网络时被更新的参数。

## 运算符

算术运算符和赋值运算符支持Number和Tensor运算，也支持不同dtype的Tensor运算。

## 原型

### 属性引用与修改

在Cell实例中使用属性修改需满足：

- 被修改属性属于本Cell对象（self.xxx）
- 该属性在Cell的`__init__`函数中初始化且为Parameter类型

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   def __init__(self):
      super().__init__()
      self.weight = mindspore.Parameter(mindspore.tensor(3, mindspore.float32))

   @mindspore.jit
   def construct(self, x, y):
      self.weight = x
      return x

net = Net()
ret = net(1, 2)
```

### 索引取值

对序列Tuple、List、Dictionary、Tensor的索引取值操作。

### 调用

执行可调用对象（Cell、Primitive）的调用：

```python
import mindspore
from mindspore import nn, ops
import numpy as np

class Net(nn.Cell):
   def __init__(self):
      super().__init__()
      self.matmul = ops.MatMul()

   @mindspore.jit
   def construct(self, x, y):
      out = self.matmul(x, y)
      return out
```

## 语句

支持raise、assert、pass、return、break、continue、if、for、while、with、列表生成式、生成器表达式、函数定义等语句。

## Python内置函数

支持部分Python内置函数，使用方法与对应的Python内置函数类似。

## 网络定义

### 网络入参

对整网入参求梯度时，忽略非Tensor的入参，只计算Tensor入参的梯度：

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   def construct(self, x, y, z):
      return x + y + z

class GradNet(nn.Cell):
   def __init__(self, net):
      super(GradNet, self).__init__()
      self.forward_net = net

   @mindspore.jit
   def construct(self, x, y, z):
      return mindspore.grad(self.forward_net, grad_position=(0, 1, 2))(x, y, z)

input_x = mindspore.tensor([1])
input_y = 2
input_z = mindspore.tensor([3])

net = Net()
grad_net = GradNet(net)
ret = grad_net(input_x, input_y, input_z)
```

## View和In-place功能

### 支持view操作

View操作创建共享数据存储但具有不同形状的新张量。仅在Ascend设备上支持，jit_level=O0和O1均支持。

```python
import numpy as np
import mindspore
from mindspore import nn, mint

mindspore.set_device(device_target="Ascend")

class Net(nn.Cell):
    def construct(self, x):
        out = mint.narrow(x, 1, 1, 2)
        return out

net = Net()
np_x = np.arange(9).reshape(3, 3).astype(np.float32)
x = mindspore.tensor(np_x)
net.construct = mindspore.jit(net.construct, backend='ms_backend')
graph_out = net(x)
```

### 支持in-place操作

In-place操作直接修改输入张量内容。Ascend设备支持。

## AST基础语法（STRICT级别）

## AST扩展语法（LAX级别）

### 调用第三方库

支持Python内置模块、标准库和第三方库（如numpy、scipy）。支持第三方库的数据类型、方法调用和返回。

```python
import numpy as np
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      a = np.array([1, 2, 3])
      b = np.array([4, 5, 6])
      out = a + b
      return out

net = Net()
print(net())
```

### 支持自定义类的使用

支持在图模式下使用用户自定义的类，进行实例化、使用属性及方法：

```python
import mindspore
from mindspore import nn

class GetattrClass():
   def __init__(self):
      self.attr1 = 99

   def method1(self, x):
      return x + self.attr1

class GetattrClassNet(nn.Cell):
   def __init__(self):
      super(GetattrClassNet, self).__init__()
      self.cls = GetattrClass()

   @mindspore.jit
   def construct(self):
      return self.cls.method1(10)
```

### 基础运算符支持更多数据类型

重载运算符支持更多数据类型，如第三方库类型。

### 基础类型扩展支持

#### 支持列表就地修改操作

在LAX模式下支持List的inplace操作（extend、pop、reverse、insert）：

```python
import mindspore
from mindspore import nn

list_input = [1, 2, 3, 4]

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      list_input.reverse()
      return list_input

net = Net()
output = net()
assert id(output) == id(list_input)
```

#### 支持Dictionary的高阶用法

支持顶图返回Dictionary、索引取值和赋值：

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      x = {'a': 'a', 'b': 'b'}
      y = x.get('a')
      z = dict(y=y)
      return z

net = Net()
out = net()
```

#### 支持使用None

None可赋值给任何变量，作为顶图或子图的入参或返回值，作为切片下标或List、Tuple、Dictionary的输入：

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      return 1, "a", None

net = Net()
res = net()
```

### 内置函数支持更多数据类型

扩展内置函数支持范围，包括第三方库数据类型。

### 支持控制流

扩展支持更多数据类型在控制流语句中的使用：

```python
import numpy as np
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      x = np.array(1)
      if x <= 1:
         x += 1
      return mindspore.tensor(x)

net = Net()
res = net()
```

### 支持属性设置与修改

对自定义类对象、第三方类型、Cell的self对象、jit_class对象进行属性设置与修改：

```python
import mindspore
from mindspore import nn

class AssignClass():
   def __init__(self):
      self.x = 1

obj = AssignClass()

class Net(nn.Cell):
   @mindspore.jit
   def construct(self):
      obj.x = 100
      return 0

net = Net()
net()
```

### 支持求导

扩展支持的静态图语法同样支持在求导中使用：

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
   @mindspore.jit
   def construct(self, a):
      x = {'a': a, 'b': 2}
      return a, x

net = Net()
out = mindspore.grad(net)(mindspore.tensor([1]))
```

### Annotation Type

使用`# @jit.typing:`注释指定Python语句输出类型，避免Any类型产生：

```python
import mindspore
from mindspore import nn, ops

class Net(nn.Cell):
   def __init__(self):
      super(Net, self).__init__()
      self.abs = ops.Abs()

   @mindspore.jit
   def construct(self, x, y):
      z = x.asnumpy() + y.asnumpy()
      y1 = mindspore.tensor(z, dtype=mindspore.float32)
      y2 = mindspore.tensor(z, dtype=mindspore.float32) # @jit.typing: () -> tensor_type[float32]
      y3 = mindspore.tensor(z)
      return self.abs(y1), self.abs(y2), self.abs(y3)

net = Net()
x = mindspore.tensor(-1, dtype=mindspore.int32)
y = mindspore.tensor(-1, dtype=mindspore.float32)
y1, y2, y3 = net(x, y)
```

## 扩展语法的语法约束

1. 须在动态图语法范围内
2. 执行性能可能受影响
3. 不能使用MindIR导入导出的能力

## 基础语法的语法约束

1. construct函数使用未定义的类成员时抛出AttributeError
2. nn.Cell不支持classmethod修饰的类方法
3. 部分Python关键字不支持：AsyncFunctionDef、Delete、AnnAssign等
4. 复数和集合类型不支持
5. 某些Python内置函数不支持
6. 第三方库使用受限
7. 图外对类属性的修改不生效

## 基于字节码构图语法介绍

基于字节码构建计算图的方式不支持宽松模式，语法支持与严格模式基本一致，主要差异：

1. 遇到不支持的语法不会报错，而是裂图处理
2. 属性设置相关的副作用操作可以入图
3. 变量场景的控制流无法入图
4. 暂不支持Python 3.12及更高版本

```python
import mindspore
from mindspore import nn

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.attr = 1

    @mindspore.jit(capture_mode="bytecode")
    def construct(self, x):
        self.attr = x + 1
        return self.attr

net = Net()
x = mindspore.tensor([1, 2, 3], dtype=mindspore.int32)
ret = net(x)
```
