# 张量 Tensor

张量（Tensor）是一个多线性函数，可用于表示矢量、标量和其他张量之间的线性关系。常见的线性关系包括内积、外积、线性映射以及笛卡儿积等。张量的坐标在 $n$ 维空间内，有 $n^{r}$ 个分量，每个分量都是坐标的函数。在坐标变换时，这些分量也依照某些规则作线性变换。$r$ 称为该张量的秩或阶（与矩阵的秩和阶无关）。

张量是一种特殊的数据结构，与数组和矩阵非常相似。张量是MindSpore网络运算中的基本数据结构，本教程主要介绍张量的属性及用法。

```python
import numpy as np
import mindspore
from mindspore import Tensor
```

## 创建张量

张量的创建方式有多种，构造张量时，支持传入`Tensor`、`float`、`int`、`bool`、`tuple`、`list`和`numpy.ndarray`类型。

### 根据数据直接生成

可以根据数据创建张量。数据类型可以手动设置，也可以由框架自动推断。

```python
data = [1, 0, 1, 0]
x_data = mindspore.tensor(data)
print(x_data, x_data.shape, x_data.dtype)
```

```
[1 0 1 0] (4,) Int64
```

### 从NumPy数组生成

可以从NumPy数组创建张量。

```python
np_array = np.array(data)
x_from_np = mindspore.tensor(np_array)
print(x_from_np, x_from_np.shape)
```

```
[1 0 1 0] (4,)
```

### 使用init初始化器构造张量

当使用`init`初始化器对张量进行初始化时，支持传入的参数有`init`、`shape`、`dtype`。

- `init`: 支持传入[initializer](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore.common.initializer.html)的子类。如：下方示例中的[One()](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore.common.initializer.html#mindspore.common.initializer.One)和[Normal()](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore.common.initializer.html#mindspore.common.initializer.Normal)。

- `shape`: 支持传入`list`、`tuple`、`int`。

- `dtype`: 支持传入[mindspore.dtype](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.dtype.html#mindspore.dtype)。

```python
from mindspore.common.initializer import One, Normal

# Initialize a tensor with ones
tensor1 = mindspore.tensor(shape=(2, 2), dtype=mindspore.float32, init=One())
# Initialize a tensor from normal distribution
tensor2 = mindspore.tensor(shape=(2, 2), dtype=mindspore.float32, init=Normal())

print("tensor1:\n", tensor1)
print("tensor2:\n", tensor2)
```

```
tensor1:
 [[1. 1.]
 [1. 1.]]
tensor2:
 [[-0.0107513   0.00407822]
 [-0.00113699  0.00081491]]
```

`init`主要用于并行模式下的延后初始化。在正常情况下，不建议使用init对参数进行初始化。

### 继承另一个张量的属性，形成新的张量

```python
from mindspore import ops

x_ones = ops.ones_like(x_data)
print(f"Ones Tensor: \n {x_ones} \n")

x_zeros = ops.zeros_like(x_data)
print(f"Zeros Tensor: \n {x_zeros} \n")
```

```
Ones Tensor:
 [1 1 1 1]

Zeros Tensor:
 [0 0 0 0]
```

## 张量的属性

张量的属性包括形状、数据类型、转置张量、单个元素大小、占用字节数量、维数、元素个数和每一维步长。

- 形状（shape）：`Tensor`的shape，为tuple类型。
- 数据类型（dtype）：`Tensor`的dtype，是MindSpore的一个数据类型。
- 单个元素大小（itemsize）：`Tensor`中每一个元素占用字节数，为整数类型。
- 占用字节数量（nbytes）：`Tensor`占用的总字节数，为整数类型。
- 维数（ndim）：`Tensor`的秩，也就是len(tensor.shape)，为整数类型。
- 元素个数（size）：`Tensor`中所有元素的个数，为整数类型。
- 每一维步长（strides）：`Tensor`每一维所需要的字节数，为tuple类型。

```python
x = mindspore.tensor(np.array([[1, 2], [3, 4]]), mindspore.int32)

print("x_shape:", x.shape)
print("x_dtype:", x.dtype)
print("x_itemsize:", x.itemsize)
print("x_nbytes:", x.nbytes)
print("x_ndim:", x.ndim)
print("x_size:", x.size)
print("x_strides:", x.strides)
```

```
x_shape: (2, 2)
x_dtype: Int32
x_itemsize: 4
x_nbytes: 16
x_ndim: 2
x_size: 4
x_strides: (8, 4)
```

## 张量索引

Tensor索引与Numpy索引类似。索引从0开始编制，负索引表示按倒序编制。冒号`:`和`...`用于对数据进行切片。

```python
tensor = mindspore.tensor(np.array([[0, 1], [2, 3]]).astype(np.float32))

print("First row: {}".format(tensor[0]))
print("value of bottom right corner: {}".format(tensor[1, 1]))
print("Last column: {}".format(tensor[:, -1]))
print("First column: {}".format(tensor[..., 0]))
```

```
First row: [0. 1.]
value of bottom right corner: 3.0
Last column: [1. 3.]
First column: [0. 2.]
```

## 张量运算

张量之间有很多运算，包括算术、线性代数、矩阵处理（转置、标引、切片）、采样等。张量运算的使用方式和NumPy类似，下面介绍其中几种操作。

> 普通算术运算包括：加（+）、减（-）、乘（*）、除（/）、取模（%）、整除（//）。

```python
x = mindspore.tensor(np.array([1, 2, 3]), mindspore.float32)
y = mindspore.tensor(np.array([4, 5, 6]), mindspore.float32)

output_add = x + y
output_sub = x - y
output_mul = x * y
output_div = y / x
output_mod = y % x
output_floordiv = y // x

print("add:", output_add)
print("sub:", output_sub)
print("mul:", output_mul)
print("div:", output_div)
print("mod:", output_mod)
print("floordiv:", output_floordiv)
```

```
add: [5. 7. 9.]
sub: [-3. -3. -3.]
mul: [ 4. 10. 18.]
div: [4.  2.5 2. ]
mod: [0. 1. 0.]
floordiv: [4. 2. 2.]
```

[concat](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/ops/mindspore.ops.concat.html)将给定维度上的一系列张量连接起来。

```python
data1 = mindspore.tensor(np.array([[0, 1], [2, 3]]).astype(np.float32))
data2 = mindspore.tensor(np.array([[4, 5], [6, 7]]).astype(np.float32))
output = ops.concat((data1, data2), axis=0)

print(output)
print("shape:\n", output.shape)
```

```
[[0. 1.]
 [2. 3.]
 [4. 5.]
 [6. 7.]]
shape:
 (4, 2)
```

[stack](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/ops/mindspore.ops.stack.html)则是从另一个维度上将两个张量合并起来。

```python
data1 = mindspore.tensor(np.array([[0, 1], [2, 3]]).astype(np.float32))
data2 = mindspore.tensor(np.array([[4, 5], [6, 7]]).astype(np.float32))
output = ops.stack([data1, data2])

print(output)
print("shape:\n", output.shape)
```

```
[[[0. 1.]
  [2. 3.]]

 [[4. 5.]
  [6. 7.]]]
shape:
 (2, 2, 2)
```

## Tensor与NumPy转换

Tensor可以和NumPy进行互相转换。

### Tensor转换为NumPy

可以使用[Tensor.asnumpy()](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/Tensor/mindspore.Tensor.asnumpy.html)将MindSpore的Tensor转换为NumPy数据。

```python
t = mindspore.tensor([1., 1., 1., 1., 1.])
print(f"t: {t}", type(t))
n = t.asnumpy()
print(f"n: {n}", type(n))
```

```
t: [1. 1. 1. 1. 1.] <class 'mindspore.common.tensor.Tensor'>
n: [1. 1. 1. 1. 1.] <class 'numpy.ndarray'>
```

### NumPy转换为Tensor

使用[Tensor.from_numpy()](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/Tensor/mindspore.Tensor.asnumpy.html)将NumPy数据转换为MindSpore的Tensor。此方法不拷贝数据，共用数据存储地址，速度较快，但NumPy数据需为连续（用numpy.iscontiguous()判断）。

```python
n = np.ones(5)
t = Tensor.from_numpy(n)
```

```python
np.add(n, 1, out=n)
print(f"n: {n}", type(n))
print(f"t: {t}", type(t))
```

```
n: [2. 2. 2. 2. 2.] <class 'numpy.ndarray'>
t: [2. 2. 2. 2. 2.] <class 'mindspore.common.tensor.Tensor'>
```

使用[mindspore.tensor](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.tensor.html)直接创建，此方法会对数据进行拷贝。

```python
n = np.ones(5)
t = mindspore.tensor(n)
```

```python
np.add(n, 1, out=n)
print(f"n: {n}", type(n))
print(f"t: {t}", type(t))
```

```
n: [2. 2. 2. 2. 2.] <class 'numpy.ndarray'>
t: [1. 1. 1. 1. 1.] <class 'mindspore.common.tensor.Tensor'>
```
