# 图模式-编程技巧

## 如何优化编译性能

### 使用lazy_inline装饰器

神经网络模型编译通常采用默认inline方式，将分层代码展开成扁平计算图。虽然inline优化提升运行性能，但过度inline会增加编译期负担。例如，随着计算图节点数量膨胀，执行pass的耗时也在急剧增长。

为减轻此影响，MindSpore提供了Lazy Inline机制。该机制适用于重复调用相同计算单元的场景，如for循环中调用同一Cell类的不同实例。

#### 大模型pipeline并行场景

在大模型场景中，编译耗时问题突出。以盘古13B网络为例，计算图节点数达13.5万个，单次编译耗时接近3小时。

应用Lazy Inline方案后，计算图节点数量从13万多个下降到2万多个，编译时间从3小时缩短到20分钟。

使用方法是在Cell类的`__init__`函数上加`@lazy_inline`装饰器：

```python
from mindspore import nn, lazy_inline

class PanGUAlphaWithLoss(nn.Cell):
    @lazy_inline
    def __init__(self, ...):
        ...

    def construct(self, ...):
        ...
```

#### 更加泛化的一般场景

`@lazy_inline`装饰器以`__init__`的所有参数生成Cell的`cell_init_args`属性。具有相同`cell_init_args`的Cell实例只需编译解析一次。

GPT网络示例：

```python
from mindspore import nn, lazy_inline

class Block(nn.Cell):
    @lazy_inline
    def __init__(self, config):
        ...

    def construct(self, x, attention_mask, layer_past):
        ...

class GPT_Model(nn.Cell):
    def __init__(self, config):
        ...
        for i in range(config.num_layers):
            self.blocks.append(Block(config))
            ...
```

#### 使用限制

1. Cell与构图相关的属性在`__init__`执行完后不能修改。若修改，该Cell实例无法准确复用同一个子图结构。
2. 若同时在多层Cell的`__init__`函数上加`@lazy_inline`，只有最外层的Cell实例的网络结构会被编译成可复用的计算图并延迟inline。内层的计算图仍会被inline。

### 使用HyperMap

使用场景：用HyperMap替换for循环来优化编译性能。

`HyperMap`是一个特殊类，构造时需传入映射函数f，调用时需传入f的n个参数序列。映射函数f必须是`MultitypeFuncGraph`类型。在用for循环批量处理列表元素时，可通过`HyperMap`等价替换来优化网络编译性能。

### 使用编译缓存

使用场景：如果编译依赖文件未变更，可通过编译缓存缩短编译时间。

编译缓存存储网络模型的编译中间过程文件。当网络模型不变时，生成的中间过程文件也相同，因此可复用上次文件。

通过环境变量`MS_COMPILER_CACHE_ENABLE`指定是否保存和加载编译缓存，通过`MS_COMPILER_CACHE_PATH`指定缓存目录。

示例代码（关闭编译缓存）：

```python
import os
import time
import mindspore
from mindspore import nn

mindspore.set_context(mode=mindspore.GRAPH_MODE)

class Model(nn.Cell):
    def construct(self, input_x, input_y):
        output = input_x
        for _ in range(200):
            output = input_x + input_x * input_y + output
        return output

os.environ['MS_COMPILER_CACHE_ENABLE'] = '0'
x = mindspore.tensor([1], mindspore.float32)
y = mindspore.tensor([2], mindspore.float32)
model = Model()
start_time = time.time()
out = model(x, y)
end_time = time.time()
print("Disable compile_cache cost time:", end_time - start_time)
```

结果：

```
Disable compile_cache cost time: 0.5485098361968994
```

第二次执行耗时减少，因为算子编译缓存默认开启：

```
Disable compile_cache cost time: 0.4614279270172119
```

开启编译缓存示例：

```python
import os
import time
import mindspore
from mindspore import nn

mindspore.set_context(mode=mindspore.GRAPH_MODE)

class Model(nn.Cell):
    def construct(self, input_x, input_y):
        output = input_x
        for _ in range(200):
            output = input_x + input_x * input_y + output
        return output

os.environ['MS_COMPILER_CACHE_ENABLE'] = '1'
os.environ['MS_COMPILER_CACHE_PATH'] = 'my_compile_cache'
x = mindspore.tensor([1], mindspore.float32)
y = mindspore.tensor([2], mindspore.float32)
model = Model()
start_time = time.time()
out = model(x, y)
end_time = time.time()
os.environ['MS_COMPILER_CACHE_ENABLE'] = '0'
print("Enable compile_cache cost time:", end_time - start_time)
```

结果：

```
Enable compile_cache cost time: 0.6357541084289551
```

第二次执行（利用缓存）：

```
Enable compile_cache cost time: 0.09379792213439941
```

开启编译缓存时，第2次执行样例耗时只有第一次执行耗时的1/7左右。

## 如何优化执行性能

### 使用jit_class

使用场景：用`@jit_class`装饰器修饰自定义类，提高执行性能。jit_class应用于静态图模式，在动态图模式下被忽略。

#### jit_class的介绍

用户可定义三种类型：

- 继承于Cell的类：神经网络基本构成单元，`construct`函数代码被编译成静态计算图。
- 自定义类：不被编译成计算图，函数方法通过Python解释器解释执行。
- @jit_class修饰的类：兼顾Python使用习惯和静态图编译性能。函数代码被编译成静态计算图，编译器可全局优化。

#### jit_class装饰器的使用

jit_class仅支持修饰自定义类，不支持修饰继承于`Cell`的类。

示例：

```python
import numpy as np
import mindspore
from mindspore import nn

@mindspore.jit_class
class InnerNet:
    value = mindspore.tensor(np.array([1, 2, 3]))

class Net(nn.Cell):
    @mindspore.jit
    def construct(self):
        return InnerNet().value

net = Net()
out = net()
print(out)
```

结果：

```
[1 2 3]
```

若jit_class修饰继承于`Cell`的类，将报错：

```
TypeError: Decorator jit_class is used for user-defined classes and cannot be used for nn.Cell: Net<>.
```

jit_class支持自定义类嵌套使用、自定义类与`Cell`嵌套使用。类继承时，若父类使用jit_class，子类也具有此能力。

嵌套示例：

```python
import numpy as np
import mindspore
from mindspore import nn

@mindspore.jit_class
class Inner:
    def __init__(self):
        self.value = mindspore.tensor(np.array([1, 2, 3]))

@mindspore.jit_class
class InnerNet:
    def __init__(self):
        self.inner = Inner()

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.inner_net = InnerNet()

    @mindspore.jit
    def construct(self):
        out = self.inner_net.inner.value
        return out

net = Net()
out = net()
print(out)
```

结果：

```
[1 2 3]
```

#### 获取类的属性和方法

支持通过类名或类实例调用属性和方法：

```python
import mindspore
from mindspore import nn

@mindspore.jit_class
class InnerNet:
    def __init__(self, val):
        self.number = val

    def act(self, x, y):
        return self.number * (x + y)

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.inner_net = InnerNet(2)

    @mindspore.jit
    def construct(self, x, y):
        return self.inner_net.number + self.inner_net.act(x, y)

x = mindspore.tensor(2, dtype=mindspore.int32)
y = mindspore.tensor(3, dtype=mindspore.int32)
net = Net()
out = net(x, y)
print(out)
```

结果：

```
12
```

#### 创建类的实例

对于被编译成静态计算图的函数，如`Cell`的`construct`函数、`@jit`修饰的函数，若需在函数内创建`@jit_class`修饰的类实例，参数要求为常量：

```python
import numpy as np
import mindspore
from mindspore import nn

@mindspore.jit_class
class InnerNet:
    def __init__(self, val):
        self.number = val + 3

class Net(nn.Cell):
    @mindspore.jit
    def construct(self):
        net = InnerNet(2)
        return net.number

net = Net()
out = net()
print(out)
```

结果：

```
5
```

#### 调用类的实例

调用`@jit_class`修饰的类实例时，调用该类的`__call__`函数方法：

```python
import numpy as np
import mindspore
from mindspore import nn

@mindspore.jit_class
class InnerNet:
    def __init__(self, number):
        self.number = number

    def __call__(self, x, y):
        return self.number * (x + y)

class Net(nn.Cell):
    @mindspore.jit
    def construct(self, x, y):
        net = InnerNet(2)
        out = net(x, y)
        return out

x = mindspore.tensor(2, dtype=mindspore.int32)
y = mindspore.tensor(3, dtype=mindspore.int32)
net = Net()
out = net(x, y)
print(out)
```

结果：

```
10
```

若类未定义`__call__`函数，将报错：

```
ValueError: MsClassObject: 'InnerNet' has no __call__ function, please check the code.
```

### 使用Select算子

使用场景：用`Select`算子替代if控制流语句，减少静态图子图生成，提高执行性能。

编写网络时常用if语句。如果if语句的条件为变量，每个if语句都会产生额外子图。在静态图模式下，子图数量越多，编译耗时越久。

使用`Select`算子替换if语句有利有弊：`Select`算子同时执行true和false分支，而if语句仅执行一个分支；但`Select`性能优于if产生的控制流算子。一般来讲，当分支中算子数量较少，建议使用Select算子；当分支中算子数量较多，建议使用if语句。

示例：

```python
import time
import mindspore
from mindspore import ops

@mindspore.jit
def if_net(x, y):
    out = 0
    for _ in range(100):
        if x < y:
            x = x - y
        else:
            x = x + y
        out = out + x
    return out

start_time = time.time()
out = if_net(mindspore.tensor([0]), mindspore.tensor([1]))
end_time = time.time()
print("if net cost time:", end_time - start_time)

@mindspore.jit
def select_net(x, y):
    out = x
    for _ in range(100):
        cond = x < y
        x = ops.select(cond, x - y, x + y)
        out = out + x
    return out

start_time = time.time()
out = select_net(mindspore.tensor([0]), mindspore.tensor([1]))
end_time = time.time()
print("select net cost time:", end_time - start_time)
```

结果：

```
if net cost time: 2.1914021968841553
select net cost time: 0.7116048336029053
```

### 使用Vmap进行批处理

使用场景：处理无依赖关系的批量数据且相关算子支持Vmap功能时，用Vmap替代for循环来优化执行性能。

MindSpore已支持Vmap功能。

示例：

```python
import numpy as np
import time
import mindspore
from mindspore import ops, vmap

def hswish_func(x):
    return ops.HSwish()(x)

@mindspore.jit
def manually_batched(xs):
    output = []
    for i in range(xs.shape[0]):
        output.append(hswish_func(xs[i]))
    return ops.stack(output)

shape = (100, 2)
prop = 100
x_np = (np.random.randn(*shape) * prop).astype(np.float32)
x = mindspore.tensor(x_np)
x = ops.sub(x, 0)

start_time = time.time()
output_vmap = vmap(hswish_func, in_axes=(0,))(x)
end_time = time.time()
print("Vmap cost time:", end_time - start_time)

start_time = time.time()
output_manually = manually_batched(x)
end_time = time.time()
print("for loop cost time:", end_time - start_time)
```

结果：

```
Vmap cost time: 0.05766916275024414
for loop cost time: 1.9284062385559082
```

此示例中批量处理100组Tensor数据，使用Vmap处理的性能超过使用for循环处理性能的30倍。

## 依赖控制保证执行序

具有副作用的函数会改变外部状态或结果依赖外部状态。根据内存属性和IO状态，副作用划分为内存副作用和IO副作用。内存副作用包括Assign、优化器算子等；IO副作用包括Print算子。

Depend用于处理依赖项操作。在大多数情况下，如果操作符有IO副作用或内存副作用，则将根据用户的语义执行，不需要另外使用Depend算子来保证执行顺序。在某些情况下，若两个运算符A和B无顺序依赖关系但A必须在B前执行，建议使用Depend指定顺序。

使用方法：

```python
a = A(x)
b = B(y)
```

插入Depend算子：

```python
a = A(x)
y = Depend(y, a)
b = B(y)
```

浮点数溢出状态检测的特殊算子存在隐含副作用，但不属于IO副作用或内存副作用。使用时有严格的顺序要求：使用NPUClearFloatStatus前需保证NPUAllocFloatStatus已执行，使用NPUGetFloatStatus前需保证NPUClearFloatStatus已执行。这些算子定义为无副作用形式，用Depend确保执行顺序。此类算子仅在Ascend平台支持。

示例：

```python
import numpy as np
import mindspore
from mindspore import nn, ops, Tensor

mindspore.set_device("Ascend")

class Net(nn.Cell):
    def __init__(self):
        super().__init__()
        self.alloc_status = ops.NPUAllocFloatStatus()
        self.get_status = ops.NPUGetFloatStatus()
        self.clear_status = ops.NPUClearFloatStatus()

    @mindspore.jit
    def construct(self, x):
        init = self.alloc_status()
        clear_status = self.clear_status(init)
        x = ops.Depend()(x, clear_status)
        res = ops.sub(x, ops.neg(x))
        init = ops.Depend()(init, res)
        get_status = self.get_status(init)
        res = ops.Depend()(res, get_status)
        return res

value = 5
data = np.full((2, 3), value, dtype=np.float16)
x = mindspore.tensor(data, dtype=mindspore.float16)
net = Net()
res = net(x)
print(res)
```

结果：

```
[[10. 10. 10.]
 [10. 10. 10.]]
```

## 优化冗余显存拷贝操作

在函数式编程中，通过参数和返回值外的渠道与外界交换数据的函数具有副作用。MindSpore框架针对副作用会插入Load算子，该算子为虚拟算子，不占用显存，仅表示需读取全局变量的值。

在图模式下，编译完整个图后才将算子下发到后端执行。多次读取全局变量使用Load算子而非多次使用真实算子，可减少显存消耗。

然而，全局变量值可能变化，若无真实算子保存值，某些场景会出现精度问题。框架会插入真实算子占用显存以保存全局变量值，避免精度问题。

MindSpore提供`MS_DEV_SIDE_EFFECT_LOAD_ELIM`开关优化显存占用，通过`export MS_DEV_SIDE_EFFECT_LOAD_ELIM=0/1/2/3`调整。设置规则：

- 0：对框架内部Load算子都插入真实算子，显存占用最多，保证精度。
- 1或默认：仅在可能出现精度问题的场景保守插入真实算子，保证精度。
- 2：在损耗一定编译性能的前提下，尽量少插入真实算子，优化显存，保证精度。
- 3：不插入真实算子，不保证精度，显存消耗最少。

示例用例：

```python
import numpy as np
import mindspore
from mindspore import nn, ops, Tensor, Parameter

class ForwardNet(nn.Cell):
    def __init__(self):
        super(ForwardNet, self).__init__()
        self.weight = Parameter(mindspore.tensor(np.array(0), mindspore.int32), name="param")

    def construct(self, x):
        out = 0
        i = 0
        while i < 3:
            ops.assign(self.weight, i)
            out = x * self.weight + out
            i = i + 1
        return out

class BackwardNet(nn.Cell):
    def __init__(self, net):
        super(BackwardNet, self).__init__(auto_prefix=False)
        self.forward_net = net
        self.grad = ops.GradOperation(get_all=True)

    @mindspore.jit
    def construct(self, *inputs):
        grads = self.grad(self.forward_net)(*inputs)
        return grads

x = mindspore.tensor(np.array(1), mindspore.int32)
graph_forword_net = ForwardNet()
graph_backword_net = BackwardNet(graph_forword_net)
graph_mode_grads = graph_backword_net(x)
output_except = (mindspore.tensor(np.array(3), mindspore.int32),)
assert np.all(graph_mode_grads == output_except)
```

通过设置`export MS_DEV_SAVE_GRAPHS=1`保存中间文件获得IR。

未对框架内部Load算子插入真实算子时（`MS_DEV_SIDE_EFFECT_LOAD_ELIM=3`），IR文件中存在3个Load算子，分别获取`para2_param`全局变量在不同时间点的值。该变量被Assign算子修改，3个Load获取的值不同。若未插入真实算子保存值，最终结果不对，内存占用最少但存在精度问题。

IR示例（未插入真实算子）：

```
# IR entry: @BackwardNet_construct
# Total subgraphs: 1
# Total params: 2
# Params:
%para1_inputs0 : <Tensor[Int32], ()>
%para2_param : <Ref[Tensor[Int32]], (), ref_key=:param>  :  has_default

subgraph @BackwardNet_construct() {
  %0 = Assign(%para2_param, Tensor(shape=[], dtype=Int32, value=0), U)
  %1 = UpdateState(U, %0)
  %2 = Load(%para2_param, %1)
  %3 = UpdateState(%1, %2)
  %4 = Assign(%para2_param, Tensor(shape=[], dtype=Int32, value=1), %3)
  %5 = UpdateState(%3, %4)
  %6 = Load(%para2_param, %5)
  %7 = UpdateState(%5, %6)
  %8 = Assign(%para2_param, Tensor(shape=[], dtype=Int32, value=2), %7)
  %9 = UpdateState(%7, %8)
  %10 = Load(%para2_param, %9)
  %11 = MakeTuple(%10, %6)
  %12 = AddN(%11)
  %13 = MakeTuple(%12, %2)
  %14 = AddN(%13)
  %15 = MakeTuple(%14)
  %16 = UpdateState(%9, %10)
  %17 = Depend(%15, %16)
  Return(%17)
}
```

将`MS_DEV_SIDE_EFFECT_LOAD_ELIM`设置为0、1或2时，IR文件简化后如下。由于该场景中Load算子均需插入真实算子保存每次Assign修改后的值，因此三种设置得到一致的IR文件。

IR示例（插入真实算子）：

```
# IR entry: @BackwardNet_construct
# Total subgraphs: 1
# Total params: 2
# Params:
%para1_inputs0 : <Tensor[Int32], ()>
%para2_param : <Ref[Tensor[Int32]], (), ref_key=:param>  :  has_default

subgraph @BackwardNet_construct() {
  %0 = Assign(%para2_param, Tensor(shape=[], dtype=Int32, value=0), U)
  %1 = UpdateState(U, %0)
  %2 = Load(%para2_param, %1)
  %3 = TensorMove(%2)
  %4 = UpdateState(%1, %3)
  %5 = Assign(%para2_param, Tensor(shape=[], dtype=Int32, value=1), %4)
  %6 = UpdateState(%4, %5)
  %7 = Load(%para2_param, %6)
  %8 = TensorMove(%7)
  %9 = UpdateState(%6, %8)
  %10 = Assign(%para2_param, Tensor(shape=[], dtype=Int32, value=2), %9)
  %11 = UpdateState(%9, %10)
  %12 = Load(%para2_param, %11)
  %13 = TensorMove(%12)
  %14 = MakeTuple(%13, %8)
  %15 = AddN(%14)
  %16 = MakeTuple(%15, %3)
  %17 = AddN(%16)
  %18 = MakeTuple(%17)
  %19 = UpdateState(%11, %13, %15)
  %20 = Depend(%18, %19)
  Return(%20)
}
```
