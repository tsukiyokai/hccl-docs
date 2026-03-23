# 图模式语法-Python内置函数

## 概述

MindSpore 2.8.0 的静态图模式支持以下Python内置函数：`int`、`float`、`bool`、`str`、`tuple`、`list`、`dict`、`getattr`、`hasattr`、`len`、`isinstance`、`all`、`any`、`round`、`max`、`min`、`sum`、`abs`、`map`、`zip`、`range`、`enumerate`、`super`、`pow`、`print`、`filter`、`type`。

## int

功能：返回一个基于数字或字符串构造的整数对象。

调用：`int(x=0, base=10)`，默认转换成十进制。

入参：

- `x` - 需要被转换为整数的对象，支持类型为`int`、`float`、`bool`、`str`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。
- `base` - 待转换进制，只有在`x`为常量`str`的时候，才可以设置该输入。

返回值：转换后的整数值。

```python
import mindspore

@mindspore.jit
def func(x):
    a = int(3)
    b = int(3.6)
    c = int('12', 16)
    d = int('0xa', 16)
    e = int('10', 8)
    f = int(x)
    return a, b, c, d, e, f

x = mindspore.tensor([-1.0], mindspore.float32)
a, b, c, d, e, f = func(x)
```

## float

功能：返回一个基于数字或字符串构造的浮点数对象。

调用：`float(x=0)`。

入参：

- `x` - 需要被转换为浮点数的对象，支持类型为`int`、`float`、`bool`、`str`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：转换后的浮点数值。

```python
import mindspore

@mindspore.jit
def func(x):
    a = float(1)
    b = float(112)
    c = float(-123.6)
    d = float('123')
    e = float(x.asnumpy())
    return a, b, c, d, e
```

## bool

功能：返回一个基于输入构造的布尔值的对象。

调用：`bool(x=False)`。

入参：

- `x` - 需要被转换为布尔值的对象，支持类型为`int`、`float`、`bool`、`str`、`list`、`tuple`、`dict`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：转换后的布尔值。

```python
import mindspore

@mindspore.jit
def func():
    a = bool()
    b = bool(0)
    c = bool("abc")
    d = bool([1, 2, 3, 4])
    e = bool(mindspore.tensor([10]).asnumpy())
    return a, b, c, d, e
```

## str

功能：返回一个基于输入构造的字符串的对象。

调用：`str(x='')`。

入参：

- `x` - 需要被转换为字符串的对象，支持类型为`int`、`float`、`bool`、`str`、`list`、`tuple`、`dict`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：输入`x`转换后的字符串。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = str()
    b = str(0)
    c = str([1, 2, 3, 4])
    d = str(mindspore.tensor([10]))
    e = str(np.array([1, 2, 3, 4]))
    return a, b, c, d, e
```

## tuple

功能：返回一个基于输入构造的元组。

调用：`tuple(x=())`。

入参：

- `x` - 需要被转换为元组的对象，支持类型为`list`、`tuple`、`dict`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：按照`x`的第零维度拆分得到的元组。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = tuple((1, 2, 3))
    b = tuple(np.array([1, 2, 3]))
    c = tuple({'a': 1, 'b': 2, 'c': 3})
    d = tuple(mindspore.tensor([1, 2, 3]))
    return a, b, c, d
```

## list

功能：返回一个基于输入构造的列表。

调用：`list(x=())`。

入参：

- `x` - 需要被转换为列表的对象，支持类型为`list`、`tuple`、`dict`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：按照`x`的第零维度拆分得到的列表。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = list((1, 2, 3))
    b = list(np.array([1, 2, 3]))
    c = list({'a':1, 'b':2, 'c':3})
    d = list(mindspore.tensor([1, 2, 3]))
    return a, b, c, d
```

## dict

功能：用于创建一个字典。

```python
import mindspore

@mindspore.jit
def func():
    a = dict()                                          # 创建空字典
    b = dict(a='a', b='b', t='t')                       # 传入关键字
    c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))  # 映射函数方式
    d = dict([('one', 1), ('two', 2), ('three', 3)])   # 可迭代对象方式
    return a, b, c, d
```

## getattr

功能：获取对象的属性。

调用：`getattr(x, attr, default)`。

入参：

- `x` - 需要被获取属性的对象。
- `attr` - 需要获取的属性，需要为`str`。
- `default` - 可选参数。若`x`没有`attr`，则返回`default`。

返回值：目标属性或者`default`。

```python
import numpy as np
import mindspore

@mindspore.jit_class
class MSClass1:
    def __init__(self):
        self.num0 = 0

ms_obj = MSClass1()

@mindspore.jit
def func(x):
    a = getattr(ms_obj, 'num0')
    b = getattr(ms_obj, 'num1', 2)
    c = getattr(x.asnumpy(), "shape", np.array([0, 1, 2, 3, 4]))
    return a, b, c
```

注意：在静态图模式下，对象的属性可能会和动态图模式下有区别。建议使用`default`输入，或者在使用`getattr`前先使用`hasattr`进行校验。

## hasattr

功能：判断对象是否具有该属性。

调用：`hasattr(x, attr)`。

入参：

- `x` - 需要被判断是否具有某属性的对象。
- `attr` - 属性名，需要为`str`。

返回值：布尔值，表示是否具有该属性。

```python
import numpy as np
import mindspore

@mindspore.jit_class
class MSClass1:
    def __init__(self):
        self.num0 = 0

ms_obj = MSClass1()

@mindspore.jit
def func():
    a = hasattr(ms_obj, 'num0')
    b = hasattr(ms_obj, 'num1')
    c = hasattr(mindspore.tensor(np.array([1, 2, 3, 4])).asnumpy(), "__len__")
    return a, b, c
```

## len

功能：获取对象（字符串或者其他可迭代对象）的长度。

调用：`len(sequence)`。

入参：

- `sequence` - `Tuple`、`List`、`Dictionary`、`Tensor`、`String`以及第三方对象（例如`numpy.ndarray`）。

返回值：序列的长度，类型为`int`。当入参是`Tensor`时，返回的是`Tensor`第零维的长度。

```python
import numpy as np
import mindspore

z = mindspore.tensor(np.ones((6, 4, 5)))

@mindspore.jit
def test(w):
    x = (2, 3, 4)
    y = [2, 3, 4]
    d = {"a": 2, "b": 3}
    n = np.array([1, 2, 3, 4])
    x_len = len(x)
    y_len = len(y)
    d_len = len(d)
    z_len = len(z)
    n_len = len(n)
    w_len = len(w.asnumpy())
    return x_len, y_len, d_len, z_len, n_len, w_len
```

## isinstance

功能：判断对象是否为一个已知的类型。

调用：`isinstance(obj, type)`。

入参：

- `obj` - MindSpore支持类型的一个实例。
- `type` - `bool`、`int`、`float`、`str`、`list`、`tuple`、`dict`、`Tensor`、`Parameter`，或者第三方库的类型或包含这些类型的`tuple`。

返回值：`obj`为`type`的实例，返回`True`，否则返回`False`。

```python
import mindspore
import numpy as np

z = mindspore.tensor(np.array([[1, 2], [3, 4], [5, 6]]))

@mindspore.jit
def test(w):
    x = (2, 3, 4)
    y = [2, 3, 4]
    x_is_tuple = isinstance(x, tuple)
    y_is_list = isinstance(y, list)
    z_is_tensor = isinstance(z, mindspore.Tensor)
    w_is_ndarray = isinstance(w.asnumpy(), np.ndarray)
    return x_is_tuple, y_is_list, z_is_tensor, w_is_ndarray
```

## all

功能：判断输入中的元素是否均为真值。

调用：`all(x)`。

入参：

- `x` - 可迭代对象，支持类型包括`tuple`、`list`、`dict`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：布尔值，如果所有元素都为`True`，则返回`True`，否则返回`False`。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = all(['a', 'b', 'c', 'd'])
    b = all(['a', 'b', '', 'd'])
    c = all([0, 1, 2, 3])
    d = all(('a', 'b', 'c', 'd'))
    e = all(('a', 'b', '', 'd'))
    f = all((0, 1, 2, 3))
    g = all([])
    h = all(())
    x = mindspore.tensor(np.array([0, 1, 2, 3]))
    i = all(x.asnumpy())
    return a, b, c, d, e, f, g, h, i
```

## any

功能：判断输入中的元素是否存在真值。

调用：`any(x)`。

入参：

- `x` - 可迭代对象，支持类型包括`tuple`、`list`、`dict`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：布尔值，如果所有元素都为`False`，则返回`False`，否则返回`True`。元素除了0、空、`False`外都算`True`。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = any(['a', 'b', 'c', 'd'])
    b = any(['a', 'b', '', 'd'])
    c = any([0, '', False])
    d = any(('a', 'b', 'c', 'd'))
    e = any(('a', 'b', '', 'd'))
    f = any((0, '', False))
    g = any([])
    h = any(())
    x = mindspore.tensor(np.array([0, 1, 2, 3]))
    i = any(x.asnumpy())
    return a, b, c, d, e, f, g, h, i
```

## round

功能：返回输入的四舍五入。

调用：`round(x, digit=0)`。

入参：

- `x` - 需要四舍五入的值，有效类型为`int`、`float`、`bool`、`Tensor`以及定义了魔术方法`__round__()`的第三方对象。
- `digit` - 表示进行四舍五入的小数点位数，默认值为0，支持`int`类型以及`None`。若`x`为`Tensor`类型，则不支持输入`digit`。

返回值：四舍五入后的值。

```python
import mindspore

@mindspore.jit
def func():
    a = round(10)
    b = round(10.123)
    c = round(10.567)
    d = round(10, 0)
    e = round(10.72, -1)
    f = round(17.12, -1)
    g = round(10.17, 1)
    h = round(10.12, 1)
    return a, b, c, d, e, f, g, h
```

## max

功能：返回给定参数的最大值。

调用：`max(*data)`。

入参：

- `*data` - 若`*data`为单输入，则会比较单个输入内的各个元素，此时`data`必须为可迭代对象。若存在多个输入，则比较每个输入。`data`有效类型为`int`、`float`、`bool`、`list`、`tuple`、`dict`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：最大值。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = max([0, 1, 2, 3])
    b = max((0, 1, 2, 3))
    c = max({1: 10, 2: 20, 3: 3})
    d = max(np.array([1, 2, 3, 4]))
    e = max(('a', 'b', 'c'))
    f = max((1, 2, 3), (1, 4))
    g = max(mindspore.tensor([1, 2, 3]))
    return a, b, c, mindspore.tensor(d), e, f, g
```

## min

功能：返回给定参数的最小值。

调用：`min(*data)`。

入参：

- `*data` - 若`*data`为单输入，则会比较单个输入内的各个元素，此时`data`必须为可迭代对象。若存在多个输入，则比较每个输入。`data`有效类型为`int`、`float`、`bool`、`list`、`tuple`、`dict`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：最小值。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = min([0, 1, 2, 3])
    b = min((0, 1, 2, 3))
    c = min({1: 10, 2: 20, 3: 3})
    d = min(np.array([1, 2, 3, 4]))
    e = min(('a', 'b', 'c'))
    f = min((1, 2, 3), (1, 4))
    g = min(mindspore.tensor([1, 2, 3]))
    return a, b, c, mindspore.tensor(d), e, f, g
```

## sum

功能：对输入序列进行求和计算。

调用：`sum(x, n=0)`。

入参：

- `x` - 表示可迭代对象，有效类型为`list`、`tuple`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。
- `n` - 表示指定相加的参数，缺省值为0。

返回值：对`x`求和后与`n`相加得到的值。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = sum([0, 1, 2])
    b = sum((0, 1, 2), 10)
    c = sum(np.array([1, 2, 3]))
    d = sum(mindspore.tensor([1, 2, 3]), 10)
    e = sum(mindspore.tensor([[1, 2], [3, 4]]))
    f = sum([1, mindspore.tensor([[1, 2], [3, 4]]), mindspore.tensor([[1, 2], [3, 4]])], mindspore.tensor([[1, 1], [1, 1]]))
    return a, b, mindspore.tensor(c), d, e, f
```

## abs

功能：返回给定参数的绝对值。

调用：`abs(x)`。

入参：

- `x` - 有效类型为`int`、`float`、`bool`、`Tensor`以及第三方对象（例如`numpy.ndarray`）。

返回值：绝对值。

```python
import mindspore

@mindspore.jit
def func():
    a = abs(-45)
    b = abs(100.12)
    c = abs(mindspore.tensor([-1, 2]).asnumpy())
    return a, b, c
```

## map

功能：根据提供的函数对一个或者多个序列做映射，由映射的结果生成一个新的序列。当前要求多个序列中的元素个数一致。

调用：`map(func, sequence, ...)`。

入参：

- `func` - 函数。
- `sequence` - 一个或多个序列（`Tuple`或者`List`）。

返回值：返回一个新的序列。

```python
import mindspore

def add(x, y):
    return x + y

@mindspore.jit
def test():
    elements_a = (1, 2, 3)
    elements_b = (4, 5, 6)
    ret1 = map(add, elements_a, elements_b)
    elements_c = [0, 1, 2]
    elements_d = [6, 7, 8]
    ret2 = map(add, elements_c, elements_d)
    return ret1, ret2
```

## zip

功能：将多个序列中对应位置的元素打包成一个个元组，然后由这些元组组成一个新序列，如果各个序列中的元素个数不一致，则生成的新序列与最短的那个长度相同。

调用：`zip(sequence, ...)`。

入参：

- `sequence` - 一个或多个序列（`Tuple`或`List`）。

返回值：返回一个新的序列。

```python
import mindspore

@mindspore.jit
def test():
    elements_a = (1, 2, 3)
    elements_b = (4, 5, 6)
    ret = zip(elements_a, elements_b)
    return ret
```

## range

功能：根据起始值、结束值和步长创建一个`Tuple`。

调用：

- `range(start, stop, step)`
- `range(start, stop)`
- `range(stop)`

入参：

- `start` - 计数起始值，类型为`int`，默认为0。
- `stop` - 计数结束值，但不包括在内，类型为`int`。
- `step` - 步长，类型为`int`，默认为1。

返回值：返回一个`Tuple`。

```python
import mindspore

@mindspore.jit
def test():
    x = range(0, 6, 2)
    y = range(0, 5)
    z = range(3)
    return x, y, z
```

## enumerate

功能：生成一个序列的索引序列，索引序列包含数据和对应下标。

调用：

- `enumerate(sequence, start=0)`
- `enumerate(sequence)`

入参：

- `sequence` - 一个序列（`Tuple`、`List`、`Tensor`）。
- `start` - 下标起始位置，类型为`int`，默认为0。

返回值：返回一个`Tuple`。

```python
import mindspore
import numpy as np

y = mindspore.tensor(np.array([[1, 2], [3, 4], [5, 6]]))

@mindspore.jit
def test():
    x = (100, 200, 300, 400)
    m = enumerate(x, 3)
    n = enumerate(y)
    return m, n
```

## super

功能：用于调用父类（超类）的一个方法，一般在`super`之后调用父类的方法。

调用：

- `super().xxx()`
- `super(type, self).xxx()`

入参：

- `type` - 类。
- `self` - 对象。

返回值：返回父类的方法。

```python
import mindspore
from mindspore import nn

class FatherNet(nn.Cell):
    def __init__(self, x):
        super(FatherNet, self).__init__(x)
        self.x = x

    def construct(self, x, y):
        return self.x * x

    def test_father(self, x):
        return self.x + x

class SingleSubNet(FatherNet):
    def __init__(self, x, z):
        super(SingleSubNet, self).__init__(x)
        self.z = z

    @mindspore.jit
    def construct(self, x, y):
        ret_father_construct = super().construct(x, y)
        ret_father_test = super(SingleSubNet, self).test_father(x)
        return ret_father_construct, ret_father_test
```

## pow

功能：求幂。

调用：`pow(x, y)`

入参：

- `x` - 底数，`Number`或`Tensor`。
- `y` - 幂指数，`Number`或`Tensor`。

返回值：返回`x`的`y`次幂，`Number`或`Tensor`。

```python
import mindspore
import numpy as np

x = mindspore.tensor(np.array([1, 2, 3]))
y = mindspore.tensor(np.array([1, 2, 3]))

@mindspore.jit
def test(x, y):
    return pow(x, y)
```

## print

功能：用于打印。

调用：`print(arg, ...)`

入参：

- `arg` - 要打印的信息（`int`、`float`、`bool`、`String`或`Tensor`，或者第三方库的数据类型）。

返回值：无返回值。

```python
import mindspore
import numpy as np

x = mindspore.tensor(np.array([1, 2, 3]), mindspore.int32)

@mindspore.jit
def test(x):
    print(x)
    return x
```

## filter

功能：根据提供的函数对一个序列的元素做判断，每个元素依次作为参数传入函数中，将返回结果不为0或False的元素组成新的序列。

调用：`filter(func, sequence)`

入参：

- `func` - 函数。
- `sequence` - 序列（`Tuple`或`List`）。

返回值：返回一个新的序列。

```python
import mindspore

def is_odd(x):
    if x % 2:
        return True
    return False

@mindspore.jit
def test():
    elements1 = (1, 2, 3, 4, 5)
    ret1 = filter(is_odd, elements1)
    elements2 = [6, 7, 8, 9, 10]
    ret2 = filter(is_odd, elements2)
    return ret1, ret2
```

## type

功能：输出入参的类型。

有效输入：Number、list、tuple、dict、numpy.ndarray、常量Tensor。

```python
import numpy as np
import mindspore

@mindspore.jit
def func():
    a = type(1)
    b = type(1.0)
    c = type([1, 2, 3])
    d = type((1, 2, 3))
    e = type({'a': 1, 'b': 2})
    f = type(np.array([1, 2, 3]))
    g = type(mindspore.tensor([1, 2, 3]))
    return a, b, c, d, e, f, g
```

注意：`type`作为Python的原生函数还有另外一种使用方法，即`type(name, bases, dict)`返回name类型的类对象。由于该用法应用场景较少，因此暂不支持。
