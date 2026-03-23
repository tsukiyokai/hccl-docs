# 数据加载与采样

## 数据加载

MindSpore的`mindspore.dataset`模块提供自定义数据加载API和常用公开数据集的加载类。

### 自定义数据集

MindSpore支持三种自定义数据集方式，通过`GeneratorDataset`接口实现：

#### 可随机访问数据集

实现`__getitem__`和`__len__`方法，允许通过索引直接访问数据样本：

```python
import numpy as np
from mindspore.dataset import GeneratorDataset

class RandomAccessDataset:
    def __init__(self):
        self._data = np.ones((5, 2))
        self._label = np.zeros((5, 1))
    def __getitem__(self, index):
        return self._data[index], self._label[index]
    def __len__(self):
        return len(self._data)

loader = RandomAccessDataset()
dataset = GeneratorDataset(source=loader, column_names=["data", "label"])
```

#### 可迭代数据集

实现`__iter__`和`__next__`方法，适用于数据库或远程服务器等场景：

```python
class IterableDataset():
    def __init__(self, start, end):
        self.start = start
        self.end = end
    def __next__(self):
        return next(self.data)
    def __iter__(self):
        self.data = iter(range(self.start, self.end))
        return self

loader = IterableDataset(1, 5)
dataset = GeneratorDataset(source=loader, column_names=["data"])
```

#### 生成器

使用Python生成器函数返回数据：

```python
def my_generator(start, end):
    yield from range(start, end)

dataset = GeneratorDataset(source=lambda: my_generator(3, 6), column_names=["data"])
```

### 常用公开数据集加载

MindSpore支持MNIST、CIFAR-10、CLUE、LJSpeech等经典数据集：

```python
from mindspore.dataset import MnistDataset

train_dataset = MnistDataset("MNIST_Data/train", shuffle=False)
```

## 采样器

采样器解决数据集过大和类别分布不均的问题。用户在加载数据集时传入采样器对象即可实现数据采样。

### RandomSampler

从索引序列中随机采样指定数目的数据，支持有放回和无放回两种模式：

```python
from mindspore.dataset import RandomSampler, NumpySlicesDataset

np_data = [1, 2, 3, 4, 5, 6, 7, 8]

# 有放回采样
sampler1 = RandomSampler(replacement=True, num_samples=5)
dataset1 = NumpySlicesDataset(np_data, column_names=["data"], sampler=sampler1)

# 无放回采样
sampler2 = RandomSampler(replacement=False, num_samples=5)
dataset2 = NumpySlicesDataset(np_data, column_names=["data"], sampler=sampler2)
```

有放回采样允许重复获取同一数据；无放回采样每条数据仅获取一次。

### WeightedRandomSampler

按指定概率列表在样本中随机采样：

```python
from mindspore.dataset import WeightedRandomSampler, Cifar10Dataset

weights = [0.8, 0.5, 0, 0, 0, 0, 0, 0, 0, 0]
sampler = WeightedRandomSampler(weights, num_samples=6)
dataset = Cifar10Dataset(DATA_DIR, sampler=sampler)
```

仅概率非零的样本有机会被采样。

### SubsetRandomSampler

从指定样本索引子序列中随机采样：

```python
from mindspore.dataset import SubsetRandomSampler

indices = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
sampler = SubsetRandomSampler(indices, num_samples=6)
dataset = Cifar10Dataset(DATA_DIR, sampler=sampler)
```

### PKSampler

在指定数据集类别P中，每种类别各采样K条数据：

```python
from mindspore.dataset import PKSampler

sampler = PKSampler(num_val=2, class_column='label', num_samples=10)
dataset = Cifar10Dataset(DATA_DIR, sampler=sampler)
```

### DistributedSampler

在分布式训练中对数据集分片采样：

```python
from mindspore.dataset import DistributedSampler

sampler = DistributedSampler(num_shards=4, shard_id=0, shuffle=False)
dataset = NumpySlicesDataset(data_source, column_names=["data"], sampler=sampler)
```

## 自定义采样器

### `__iter__`模式

继承`Sampler`基类，实现`__iter__`方法：

```python
import mindspore.dataset as ds

class MySampler(ds.Sampler):
    def __iter__(self):
        yield from range(0, 10, 2)

np_data = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l']
dataset = ds.NumpySlicesDataset(np_data, column_names=["data"], sampler=MySampler())
```

### `__getitem__`模式

定义包含`__init__`、`__getitem__`和`__len__`方法的采样器类：

```python
class MySampler():
    def __init__(self):
        self.index_ids = [3, 4, 3, 2, 0, 11, 5, 5, 5, 9, 1, 11, 11, 11, 11, 8]
    def __getitem__(self, index):
        return self.index_ids[index]
    def __len__(self):
        return len(self.index_ids)

np_data = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l']
dataset = ds.NumpySlicesDataset(np_data, column_names=["data"], sampler=MySampler())
```
