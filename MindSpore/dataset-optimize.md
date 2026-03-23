# 数据处理性能优化

## 数据加载性能优化

MindSpore支持多种数据加载方式，性能存在差异：

| 数据集类型                 | 底层实现 | 性能 |
|---------------------------|----------|------|
| 常用数据集                 | C++      | 高   |
| 标准格式（MindRecord等）   | C++      | 高   |
| 用户自定义                 | Python   | 中   |

### 优化建议

- 使用官方数据集加载接口：对已提供加载接口的常用数据集，优先使用MindSpore提供的接口
- 增加并行工作线程：通过提高 `num_parallel_workers` 参数（默认值：8）来提升性能
- 转换为MindRecord格式：将不支持的数据集格式转换为MindRecord再加载
- GeneratorDataset优化：
  - 增大 `num_parallel_workers`（默认值：1）
  - 配置 `python_multiprocessing` 参数（True为多进程，False为多线程）
  - 调整 `max_rowsize` 参数改进进程间数据传递效率

### 代码示例

```python
import mindspore.dataset as ds

cifar10_path = "./datasets/cifar-10-batches-bin/train"
cifar10_dataset = ds.Cifar10Dataset(cifar10_path, num_parallel_workers=4)
print(next(cifar10_dataset.create_dict_iterator()))
```

## Shuffle性能优化

### 方案选择

基于内存缓存的 `shuffle()` 函数不如直接在数据集加载接口中设置 `shuffle=True` 参数高效。

### 优化建议

- 直接使用数据集加载接口的 `shuffle=True` 参数进行混洗
- 若使用 `shuffle()` 函数，根据需求调整 `buffer_size` 参数

### 代码示例

```python
# 方法一：加载时混洗
cifar10_dataset = ds.Cifar10Dataset(cifar10_path, shuffle=True)

# 方法二：使用shuffle函数
ds2 = ds1.shuffle(buffer_size=3)
```

## 数据增强性能优化

### 实现方式对比

| 实现方式 | 性能 | 特点         |
|---------|------|-------------|
| C++     | 高   | 内置增强操作 |
| Python  | 较低 | 自定义灵活   |

### 优化建议

- 优先使用C++实现的MindSpore增强操作
- 多线程优化：增大 `map()` 的 `num_parallel_workers` 参数
- 融合算子优化：设置环境变量 `export OPTIMIZE=true` 来启用算子融合
- Compose优化：通过单个map操作接收多个增强操作，降低CPU竞争

### 代码示例

```python
import mindspore.dataset.vision as vision

cifar10_dataset = ds.Cifar10Dataset(cifar10_path, num_parallel_workers=4)
transforms = vision.RandomResizedCrop((800, 800))
cifar10_dataset = cifar10_dataset.map(
    operations=transforms,
    input_columns="image",
    num_parallel_workers=4
)
```

## Batch操作性能优化

### 优化建议

- 仅配置 `batch_size` 和 `drop_remainder` 时，增大 `num_parallel_workers` 参数
- 使用 `per_batch_map` 功能时：
  - 增大 `num_parallel_workers` 参数
  - 配置 `python_multiprocessing` 参数
  - 调整 `max_rowsize` 参数

## 操作系统性能优化

### 1. 存储设备

- 使用固态硬盘存储大数据集
- 利用"单节点缓存技术"手动缓存增强后的数据

### 2. NUMA架构

使用命令绑定进程与节点：

```bash
numactl --cpubind=0 --membind=0 python train.py
export DATASET_ENABLE_NUMA=True
```

### 3. CPU计算资源

计算资源分配：

```bash
numactl --cpubind=0 python train.py
```

CPU频率设置：

```bash
cpupower frequency-set -g performance
```

多线程竞争解决：

```python
import cv2
import numpy as np
import numba

cv2.setNumThreads(2)
# export OPENBLAS_NUM_THREADS=1
# numba.set_num_threads(1)
```

降低CPU/内存占用：

```python
import mindspore.dataset as ds

ds.config.set_prefetch_size(2)  # 降低预取大小
dataset = ds.Cifar10Dataset(path, num_parallel_workers=1)
```

## 单卡 vs 多卡训练建议

- 各操作的 `num_parallel_workers` 之和不超过CPU最大线程数
- 使用MindSpore Profiler分析各操作性能，优先提升瓶颈操作的并行度
- 多卡训练时谨慎增加该参数，避免资源竞争导致性能下降

## 自动优化工具

MindSpore提供Dataset AutoTune工具，可在训练中自动调整管道并行度；同时支持"数据异构加速"技术，将运算分配到不同异构硬件。
