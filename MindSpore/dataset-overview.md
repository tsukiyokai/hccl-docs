# 数据处理概述

MindSpore Dataset 提供两种数据处理能力：数据处理Pipeline模式和数据处理轻量化模式。

## 数据处理Pipeline模式

### 工作流程

用户通过API定义Dataset流水线，运行训练进程后Dataset会从数据集中循环加载数据 -> 处理 -> Batch -> 迭代器，最终用于训练。

具体步骤包括：

#### 数据集加载（Dataset）

用户可使用Dataset类加载已支持的数据集，或通过 UDF Loader + GeneratorDataset 实现自定义数据集加载。加载类方法可使用多种Sampler、数据分片、数据shuffle等功能。

数据集接口分类：

| 分类           | API列表                                              | 说明                                |
|----------------|------------------------------------------------------|-------------------------------------|
| 标准格式数据集 | MindDataset、TFRecordDataset、CSVDataset等           | MindDataset依赖MindSpore数据格式    |
| 自定义数据集   | GeneratorDataset、RandomDataset等                    | GeneratorDataset负责加载用户自定义DataLoader |
| 常用数据集     | ImageFolderDataset、Cifar10Dataset、IWSLT2017Dataset等 | 用于常用的开源数据集                |

常用参数：

- `columns_list`：过滤指定列，默认值None加载所有列
- `num_parallel_workers`：配置数据集读取并发数，默认值8
- `shuffle`：开启混洗，默认值True
- `num_shards` 和 `shard_id`：对数据集进行分片

#### 数据集操作（filter/skip）

用户通过数据集对象方法实现进一步处理：

- `.shuffle()`：混洗数据
- `.filter()`：过滤数据
- `.skip()`：跳过数据
- `.split()`：数据集切分
- `.take()`：获取前n条数据

#### 数据集样本变换操作（map）

用户可将数据变换操作添加到map操作中执行。数据预处理过程中可定义多个map操作，用于执行不同变换操作。数据变换操作也可是用户自定义变换的PyFunc。

#### 批（batch）

用户在样本完成变换后，使用 `.batch()` 操作将多个样本组织成batch，也可通过batch的参数 `per_batch_map` 自定义batch逻辑。

#### 迭代器（create_dict_iterator）

用户通过 `.create_dict_iterator()` 或 `.create_tuple_iterator()` 创建迭代器，将预处理完成的数据循环输出。

### 数据变换

#### 普通数据变换

- `.map()`：变换操作
- `.filter()`：过滤操作
- `.project()`：对多列进行排序和过滤
- `.rename()`：对指定列重命名
- `.shuffle()`：对数据进行缓存区大小的混洗
- `.skip()`：跳过数据集的前n条
- `.take()`：只读数据集的前n条样本

#### 自动数据增强

Dataset提供自动数据变换方式，可基于特定策略自动对图像进行数据变换处理。

### 数据batch

Dataset提供 `.batch()` 操作的两种使用方式：

1. 默认 `.batch()` 操作，将batch_size个样本组织成shape为 (batch_size, …)的数据
2. 自定义 `.batch(..., per_batch_map, ...)` 操作，支持用户按自定义逻辑组织batch

### 数据集迭代器

用户在定义完成 `数据集加载 -> 数据处理 -> 数据batch` Dataset流水线后，可通过 `.create_dict_iterator()` 或 `.create_tuple_iterator()` 循环输出数据。

### 性能优化

#### 数据处理性能优化

针对数据处理Pipeline性能不足的场景，可参考数据处理性能优化来进一步优化性能。

#### 单节点数据缓存

对于推理场景，可使用单节点数据缓存将数据集缓存于本地内存中，加速数据集的读取和预处理。

## 数据处理轻量化模式

用户可直接使用数据变换操作处理一条数据，返回值即是数据变换的结果。

数据变换操作可像调用普通函数一样直接使用。常见用法是：先初始化数据变换对象，然后调用数据变换操作方法，传入需要处理的数据，最后得到处理的结果。

## 其他特性

### 数据处理管道支持Python对象

数据处理管道中的特定操作（如自定义数据集GeneratorDataset、自定义map增强操作、自定义batch）支持任意Python类型对象作为输入。
