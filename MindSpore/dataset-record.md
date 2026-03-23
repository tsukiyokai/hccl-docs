# MindRecord格式转换

MindSpore可以将训练数据集转换为MindSpore Record格式，便于数据的保存和加载。该格式有助于减少磁盘和网络IO开销。

## MindRecord数据格式特征

主要包括：

1. 实现数据统一存储和访问
2. 支持数据聚合存储和高效读取
3. 提供高效的数据编解码操作
4. 可灵活控制数据切分的分区大小

## Record文件结构

MindSpore Record文件由数据文件和索引文件组成：

数据文件包含三部分：

- 文件头：存储元信息、Schema信息、索引字段等
- 标量数据页：存储整型、字符串、浮点型等标量类型数据
- 块数据页：存储二进制串、NumPy数组等数据

索引文件：包含基于标量数据生成的索引信息

> 单个MindSpore Record文件建议小于20G，大数据集应分片存储为多个文件。

## 转换CV类数据示例

```python
from PIL import Image
from io import BytesIO
from mindspore.mindrecord import FileWriter

file_name = "test_vision.mindrecord"
cv_schema = {"file_name": {"type": "string"},
             "label":     {"type": "int32"},
             "data":      {"type": "bytes"}}

writer = FileWriter(file_name, shard_num=1, overwrite=True)
writer.add_schema(cv_schema, "it is a cv dataset")
writer.add_index(["file_name", "label"])

data = []
for i in range(100):
    sample = {}
    white_io = BytesIO()
    Image.new('RGB', ((i+1)*10, (i+1)*10), (255, 255, 255)).save(white_io, 'JPEG')
    sample['file_name'] = str(i+1) + ".jpg"
    sample['label'] = i+1
    sample['data'] = white_io.getvalue()
    data.append(sample)
    if i % 10 == 0:
        writer.write_raw_data(data)
        data = []

if data:
    writer.write_raw_data(data)

writer.commit()
```

## 读取MindRecord文件

```python
from mindspore.dataset import MindDataset
from mindspore.dataset.vision import Decode

data_set = MindDataset(dataset_files=file_name)
decode_op = Decode()
data_set = data_set.map(operations=decode_op, input_columns=["data"], num_parallel_workers=2)
print("Got {} samples".format(data_set.get_dataset_size()))
```

## 转换NLP类数据示例

```python
import numpy as np
from mindspore.mindrecord import FileWriter

file_name = "test_text.mindrecord"

nlp_schema = {"source_sos_ids":  {"type": "int64", "shape": [-1]},
              "source_sos_mask": {"type": "int64", "shape": [-1]},
              "source_eos_ids":  {"type": "int64", "shape": [-1]},
              "source_eos_mask": {"type": "int64", "shape": [-1]},
              "target_sos_ids":  {"type": "int64", "shape": [-1]},
              "target_sos_mask": {"type": "int64", "shape": [-1]},
              "target_eos_ids":  {"type": "int64", "shape": [-1]},
              "target_eos_mask": {"type": "int64", "shape": [-1]}}

writer = FileWriter(file_name, shard_num=1, overwrite=True)
writer.add_schema(nlp_schema, "Preprocessed nlp dataset.")

data = []
for i in range(100):
    sample = {"source_sos_ids":  np.array([i, i + 1, i + 2, i + 3, i + 4], dtype=np.int64),
              "source_sos_mask": np.array([i * 1, i * 2, i * 3, i * 4, i * 5, i * 6, i * 7], dtype=np.int64),
              "source_eos_ids":  np.array([i + 5, i + 6, i + 7, i + 8, i + 9, i + 10], dtype=np.int64),
              "source_eos_mask": np.array([19, 20, 21, 22, 23, 24, 25, 26, 27], dtype=np.int64),
              "target_sos_ids":  np.array([28, 29, 30, 31, 32], dtype=np.int64),
              "target_sos_mask": np.array([33, 34, 35, 36, 37, 38], dtype=np.int64),
              "target_eos_ids":  np.array([39, 40, 41, 42, 43, 44, 45, 46, 47], dtype=np.int64),
              "target_eos_mask": np.array([48, 49, 50, 51], dtype=np.int64)}
    data.append(sample)
    if i % 10 == 0:
        writer.write_raw_data(data)
        data = []

if data:
    writer.write_raw_data(data)

writer.commit()
```

## Dataset转存MindRecord

使用`Dataset.save`方法可将常用数据集转换为MindSpore Record格式：

```python
from mindspore.dataset import Cifar10Dataset

dataset = Cifar10Dataset("./cifar-10-batches-bin/")
dataset.save("cifar10.mindrecord")
```

然后通过MindDataset读取：

```python
from mindspore.dataset import MindDataset

data_set = MindDataset(dataset_files="cifar10.mindrecord")
print("Got {} samples".format(data_set.get_dataset_size()))
```
