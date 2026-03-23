# 数据操作/数据变换

## 数据操作

`mindspore.dataset` 提供了一系列数据集操作，用户可通过这些操作实现数据集的混洗、过滤、跳过、批处理组合等功能。

常用数据变换操作包括：

- `.filter(...)`：通过指定条件，对数据进行过滤，保留满足预期条件的样本。
- `.project(...)`：对多个数据列进行排序，或删除不需要的数据列。
- `.rename(...)`：对指定数据列进行重命名，便于标记数据特性。
- `.shuffle(...)`：划分一个数据缓冲区，对缓冲区内的数据进行混洗。
- `.skip(...)`：跳过数据集的前n条样本。
- `.take(...)`：只获取数据集的前n条样本。
- `.map(...)`：通过自定义方法对每个样本进行变换和增强。
- `.batch(...)`：对 `batch_size` 条数据进行组合。

### 基础示例

```python
from mindspore.dataset import GeneratorDataset

class MyDataset:
    def __init__(self):
        self._data = [1, 2, 3, 4, 5, 6]
    def __getitem__(self, index):
        return self._data[index]
    def __len__(self):
        return len(self._data)

loader = MyDataset()

# find sampler which value < 4
dataset = GeneratorDataset(source=loader, column_names=["data"], shuffle=False)
filtered_dataset = dataset.filter(lambda x: x < 4, input_columns=["data"])
print("filtered_dataset", list(filtered_dataset))

# skip first 3 samples
dataset = GeneratorDataset(source=loader, column_names=["data"], shuffle=False)
skipped_dataset = dataset.skip(3)
print("skipped_dataset", list(skipped_dataset))

# batch the dataset by batch_size=2
dataset = GeneratorDataset(source=loader, column_names=["data"], shuffle=False)
batched_dataset = dataset.batch(2, num_parallel_workers=1)
print("batched_dataset", list(batched_dataset))
```

### 数据集组合

数据集组合可以将多个数据集以串联/并联的方式组合起来，形成一个全新的dataset对象。

```python
import mindspore.dataset as ds

ds.config.set_seed(1234)

# concat same column of two datasets
data = [1, 2, 3]
dataset1 = ds.NumpySlicesDataset(data=data, column_names=["column_1"])

data = [4, 5, 6]
dataset2 = ds.NumpySlicesDataset(data=data, column_names=["column_1"])

dataset = dataset1.concat(dataset2)
for item in dataset.create_dict_iterator():
    print("concated dataset", item)

# zip different columns of two datasets
data = [1, 2, 3]
dataset1 = ds.NumpySlicesDataset(data=data, column_names=["column_1"])

data = [4, 5, 6]
dataset2 = ds.NumpySlicesDataset(data=data, column_names=["column_2"])

dataset = dataset1.zip(dataset2)
for item in dataset.create_dict_iterator():
    print("zipped dataset", item)
```

### 数据集切分

将数据集切分成训练集和验证集，分别用于训练和验证过程。

```python
import mindspore.dataset as ds

data = [1, 2, 3, 4, 5, 6]
dataset = ds.NumpySlicesDataset(data=data, column_names=["column_1"], shuffle=False)

train_dataset, eval_dataset = dataset.split([4, 2])

print(">>>> train dataset >>>>")
for item in train_dataset.create_dict_iterator():
    print(item)

print(">>>> eval dataset >>>>")
for item in eval_dataset.create_dict_iterator():
    print(item)
```

### 数据集保存

将数据集重新保存为MindRecord数据格式。

```python
import os
import mindspore.dataset as ds

ds.config.set_seed(1234)

data = [1, 2, 3, 4, 5, 6]
dataset = ds.NumpySlicesDataset(data=data, column_names=["column_1"])

if os.path.exists("./train_dataset.mindrecord"):
    os.remove("./train_dataset.mindrecord")
if os.path.exists("./train_dataset.mindrecord.db"):
    os.remove("./train_dataset.mindrecord.db")

dataset.save("./train_dataset.mindrecord")
```

## 数据变换

通常情况下，直接加载的原始数据无法直接用于神经网络训练，此时需要对其进行数据预处理。MindSpore提供多种数据变换（Transforms）操作，可结合数据处理Pipeline实现预处理。

### 基于map数据操作进行数据变换

`mindspore.dataset` 提供面向图像、文本、音频等不同数据类型的内置数据变换操作，所有变换均可传递给 `map` 操作，通过 `map` 方法自动对每个样本进行变换。除了内置的数据变换外，`map` 操作也支持执行用户自定义的变换操作。

```python
from mindspore.dataset import MnistDataset
import mindspore.dataset.vision as vision

# create MNIST loader
train_dataset = MnistDataset("MNIST_Data/train", shuffle=False)

# resize samples to (64, 64) using built-in transformation
train_dataset = train_dataset.map(operations=[vision.Resize((64, 64))],
                                  input_columns=['image'])

for data in train_dataset:
    print(data[0].shape, data[0].dtype)
    break
```

自定义变换示例：

```python
train_dataset = MnistDataset("MNIST_Data/train", shuffle=False)

def transform(img):
    img = img / 255.0
    return img

# apply normalize using customized transformation
train_dataset = train_dataset.map(operations=[transform],
                                  input_columns=['image'])

for data in train_dataset:
    print(data[0].shape, data[0].dtype)
    break
```

### 轻量化数据变换（Eager模式）

MindSpore提供一种轻量化的数据处理方式，称为Eager模式。在Eager模式下，Transforms以函数式调用的方式执行，代码更为简洁，且能立即获得运行结果。推荐在小型数据变换实验、模型推理等轻量化场景中使用。

MindSpore目前支持在Eager模式执行各种Transform，具体如下所示：

- vision模块：基于OpenCV/Pillow实现的数据变换。
- text模块：基于Jieba/ICU4C等库实现的数据变换。
- audio模块：基于C++实现的数据变换。
- transforms模块：基于C++/Python/NumPy实现的通用数据变换。

#### vision示例

Vision Transform的Eager模式支持 `numpy.array` 或 `PIL.Image` 类型数据作为入参。

```python
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
import mindspore.dataset.vision as vision

img_ori = Image.open("banana.jpg").convert("RGB")
print("Image.type: {}, Image.shape: {}".format(type(img_ori), img_ori.size))

# Apply Resize to input immediately
op1 = vision.Resize(size=(320))
img = op1(img_ori)
print("Image.type: {}, Image.shape: {}".format(type(img), img.size))

# Apply CenterCrop to input immediately
op2 = vision.CenterCrop((280, 280))
img = op2(img)
print("Image.type: {}, Image.shape: {}".format(type(img), img.size))

# Apply Pad to input immediately
op3 = vision.Pad(40)
img = op3(img)
print("Image.type: {}, Image.shape: {}".format(type(img), img.size))

# Show the result
plt.subplot(1, 2, 1)
plt.imshow(img_ori)
plt.title("original image")
plt.subplot(1, 2, 2)
plt.imshow(img)
plt.title("transformed image")
plt.show()
```

#### text示例

Text Transforms的Eager模式支持 `numpy.array` 类型数据作为入参。

```python
import mindspore.dataset.text.transforms as text
import mindspore as ms

# Apply UnicodeCharTokenizer to input immediately
txt = "Welcome to Beijing !"
txt = text.UnicodeCharTokenizer()(txt)
print("Tokenize result: {}".format(txt))

# Apply ToNumber to input immediately
txt = ["123456"]
to_number = text.ToNumber(ms.int32)
txt = to_number(txt)
print("ToNumber result: {}, type: {}".format(txt, txt[0].dtype))
```

#### audio示例

Audio Transforms的Eager模式支持 `numpy.array` 类型数据作为入参。

```python
import numpy as np
import matplotlib.pyplot as plt
import scipy.io.wavfile as wavfile
import mindspore.dataset as ds
import mindspore.dataset.audio as audio

ds.config.set_seed(5)

sample_rate, waveform = wavfile.read(wav_file)

bass_biquad = audio.BassBiquad(sample_rate, 10.0)
transformed_waveform = bass_biquad(waveform.astype(np.float32))
```

#### transforms示例

通用Transform的Eager模式支持 `numpy.array` 类型数据作为入参。

```python
import numpy as np
import mindspore.dataset.transforms as trans

# Apply Fill to input immediately
data = np.array([1, 2, 3, 4, 5])
fill = trans.Fill(0)
data = fill(data)
print("Fill result: ", data)

# Apply OneHot to input immediately
label = np.array(2)
onehot = trans.OneHot(num_classes=5)
label = onehot(label)
print("OneHot result: ", label)
```
