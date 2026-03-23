# 快速入门

本节通过MindSpore的API来快速实现一个简单的深度学习模型。若想要深入了解MindSpore的使用方法，请参阅各节最后提供的参考链接。

首先导入样例所需的公共包和接口。

```python
import mindspore
from mindspore import nn
from mindspore.dataset import vision, transforms
from mindspore.dataset import MnistDataset
```

## 处理数据集

MindSpore提供基于Pipeline的[数据引擎](https://www.mindspore.cn/docs/zh-CN/r2.8.0/features/data_engine.html)，通过[数据集（Dataset）](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/dataset.html)实现高效的数据预处理。在本教程中，我们使用Mnist数据集，自动下载完成后，使用`mindspore.dataset`提供的数据变换进行预处理。

> 本章节中的示例代码依赖`download`，可使用命令`pip install download`安装。如本文档以Notebook运行时，完成安装后需要重启kernel才能执行后续代码。

```python
# Download data from open datasets
from download import download

url = "https://mindspore-website.obs.cn-north-4.myhuaweicloud.com/" \
      "notebook/datasets/MNIST_Data.zip"
path = download(url, "./", kind="zip", replace=True)
```

```
Downloading data from https://mindspore-website.obs.cn-north-4.myhuaweicloud.com/notebook/datasets/MNIST_Data.zip (10.3 MB)

file_sizes: 100%|██████████████████████████| 10.8M/10.8M [00:01<00:00, 6.73MB/s]
Extracting zip file...
Successfully downloaded / unzipped to ./
```

MNIST数据集目录结构如下：

```
MNIST_Data
└── train
    ├── train-images-idx3-ubyte (60000个训练图片)
    ├── train-labels-idx1-ubyte (60000个训练标签)
└── test
    ├── t10k-images-idx3-ubyte (10000个测试图片)
    ├── t10k-labels-idx1-ubyte (10000个测试标签)
```

数据下载完成后，获得数据集对象。

```python
train_dataset = MnistDataset('MNIST_Data/train')
test_dataset = MnistDataset('MNIST_Data/test')
```

打印数据集中包含的数据列名，用于dataset的预处理。

```python
print(train_dataset.get_col_names())
```

```
['image', 'label']
```

MindSpore的dataset使用数据处理流水线（Data Processing Pipeline），需指定map、batch、shuffle等操作。这里我们使用map对图像数据及标签进行变换处理，将输入的图像缩放为1/255，根据均值0.1307和标准差值0.3081进行归一化处理，然后将处理好的数据集打包为大小为64的batch。

```python
def datapipe(dataset, batch_size):
    image_transforms = [
        vision.Rescale(1.0 / 255.0, 0),
        vision.Normalize(mean=(0.1307,), std=(0.3081,)),
        vision.HWC2CHW()
    ]
    label_transform = transforms.TypeCast(mindspore.int32)

    dataset = dataset.map(image_transforms, 'image')
    dataset = dataset.map(label_transform, 'label')
    dataset = dataset.batch(batch_size)
    return dataset
```

```python
# Map vision transforms and batch dataset
train_dataset = datapipe(train_dataset, 64)
test_dataset = datapipe(test_dataset, 64)
```

可使用[create_tuple_iterator](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/dataset/dataset_method/iterator/mindspore.dataset.Dataset.create_tuple_iterator.html)或[create_dict_iterator](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/dataset/dataset_method/iterator/mindspore.dataset.Dataset.create_dict_iterator.html)对数据集进行迭代访问，查看数据和标签的shape和datatype。

```python
for image, label in test_dataset.create_tuple_iterator():
    print(f"Shape of image [N, C, H, W]: {image.shape} {image.dtype}")
    print(f"Shape of label: {label.shape} {label.dtype}")
    break
```

```
Shape of image [N, C, H, W]: (64, 1, 28, 28) Float32
Shape of label: (64,) Int32
```

```python
for data in test_dataset.create_dict_iterator():
    print(f"Shape of image [N, C, H, W]: {data['image'].shape} {data['image'].dtype}")
    print(f"Shape of label: {data['label'].shape} {data['label'].dtype}")
    break
```

```
Shape of image [N, C, H, W]: (64, 1, 28, 28) Float32
Shape of label: (64,) Int32
```

更多细节详见[数据加载与处理](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/dataset.html)。

## 网络构建

`mindspore.nn`类是构建所有网络的基类，也是网络的基本单元。当用户需要自定义网络时，可以继承`nn.Cell`类，并重写`__init__`方法和`construct`方法。`__init__`包含所有网络层的定义，`construct`包含数据（[Tensor](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/tensor.html)）的变换过程。

```python
# Define model
class Network(nn.Cell):
    def __init__(self):
        super().__init__()
        self.flatten = nn.Flatten()
        self.dense_relu_sequential = nn.SequentialCell(
            nn.Dense(28*28, 512),
            nn.ReLU(),
            nn.Dense(512, 512),
            nn.ReLU(),
            nn.Dense(512, 10)
        )

    def construct(self, x):
        x = self.flatten(x)
        logits = self.dense_relu_sequential(x)
        return logits

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

更多细节详见[网络构建](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/model.html)。

## 模型训练

在模型训练中，一个完整的训练过程（step）需要实现以下三步：

1. 正向计算：模型输出预测结果（logits），并与正确标签（label）计算预测损失（loss）。
2. 反向传播：利用自动微分机制，自动计算模型参数（parameters）对loss的梯度（gradients）。
3. 参数优化：将梯度更新到参数上。

MindSpore使用函数式自动微分机制，因此针对上述步骤需要实现：

1. 定义正向计算函数。
2. 使用[value_and_grad](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.value_and_grad.html)通过函数变换获得梯度计算函数。
3. 定义训练函数，使用[set_train](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.Cell.html#mindspore.nn.Cell.set_train)设置为训练模式，执行正向计算、反向传播和参数优化。

```python
# Instantiate loss function and optimizer
loss_fn = nn.CrossEntropyLoss()
optimizer = nn.SGD(model.trainable_params(), 1e-2)

# 1. Define forward function
def forward_fn(data, label):
    logits = model(data)
    loss = loss_fn(logits, label)
    return loss, logits

# 2. Get gradient function
grad_fn = mindspore.value_and_grad(forward_fn, None, optimizer.parameters, has_aux=True)

# 3. Define function of one-step training
def train_step(data, label):
    (loss, _), grads = grad_fn(data, label)
    optimizer(grads)
    return loss

def train(model, dataset):
    size = dataset.get_dataset_size()
    model.set_train()
    for batch, (data, label) in enumerate(dataset.create_tuple_iterator()):
        loss = train_step(data, label)

        if batch % 100 == 0:
            loss, current = loss.asnumpy(), batch
            print(f"loss: {loss:>7f}  [{current:>3d}/{size:>3d}]")
```

除训练外，我们定义测试函数，用来评估模型的性能。

```python
def test(model, dataset, loss_fn):
    num_batches = dataset.get_dataset_size()
    model.set_train(False)
    total, test_loss, correct = 0, 0, 0
    for data, label in dataset.create_tuple_iterator():
        pred = model(data)
        total += len(data)
        test_loss += loss_fn(pred, label).asnumpy()
        correct += (pred.argmax(1) == label).asnumpy().sum()
    test_loss /= num_batches
    correct /= total
    print(f"Test: \n Accuracy: {(100*correct):>0.1f}%, Avg loss: {test_loss:>8f} \n")
```

训练过程需多次迭代数据集，一次完整的迭代称为一轮（epoch）。在每一轮，遍历训练集进行训练，结束后使用测试集进行预测。打印每一轮的loss值和预测准确率（Accuracy），可以看到loss在不断下降，Accuracy在不断提高。

```python
epochs = 3
for t in range(epochs):
    print(f"Epoch {t+1}\n-------------------------------")
    train(model, train_dataset)
    test(model, test_dataset, loss_fn)
print("Done!")
```

```
Epoch 1
-------------------------------
loss: 2.302088  [  0/938]
loss: 2.290692  [100/938]
loss: 2.266338  [200/938]
loss: 2.205240  [300/938]
loss: 1.907198  [400/938]
loss: 1.455603  [500/938]
loss: 0.861103  [600/938]
loss: 0.767219  [700/938]
loss: 0.422253  [800/938]
loss: 0.513922  [900/938]
Test:
 Accuracy: 83.8%, Avg loss: 0.529534

Epoch 2
-------------------------------
loss: 0.580867  [  0/938]
loss: 0.479347  [100/938]
loss: 0.677991  [200/938]
loss: 0.550141  [300/938]
loss: 0.226565  [400/938]
loss: 0.314738  [500/938]
loss: 0.298739  [600/938]
loss: 0.459540  [700/938]
loss: 0.332978  [800/938]
loss: 0.406709  [900/938]
Test:
 Accuracy: 90.2%, Avg loss: 0.334828

Epoch 3
-------------------------------
loss: 0.461890  [  0/938]
loss: 0.242303  [100/938]
loss: 0.281414  [200/938]
loss: 0.207835  [300/938]
loss: 0.206000  [400/938]
loss: 0.409646  [500/938]
loss: 0.193608  [600/938]
loss: 0.217575  [700/938]
loss: 0.212817  [800/938]
loss: 0.202862  [900/938]
Test:
 Accuracy: 91.9%, Avg loss: 0.280962

Done!
```

更多细节详见[模型训练](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/train.html)。

## 保存模型

模型训练完成后，需要保存其参数。

```python
# Save checkpoint
mindspore.save_checkpoint(model, "model.ckpt")
print("Saved Model to model.ckpt")
```

```
Saved Model to model.ckpt
```

## 加载模型

加载保存的权重分为两步：

1. 重新实例化模型对象，构造模型。
2. 加载模型参数，并将其加载至模型上。

```python
# Instantiate a random initialized model
model = Network()
# Load checkpoint and load parameter to model
param_dict = mindspore.load_checkpoint("model.ckpt")
param_not_load, _ = mindspore.load_param_into_net(model, param_dict)
print(param_not_load)
```

```
[]
```

> `param_not_load`是未被加载的参数列表。为空时，表示所有参数均加载成功。

加载后的模型可以直接用于预测推理。

```python
model.set_train(False)
for data, label in test_dataset:
    pred = model(data)
    predicted = pred.argmax(1)
    print(f'Predicted: "{predicted[:10]}", Actual: "{label[:10]}"')
    break
```

```
Predicted: "[3 9 6 1 6 7 4 5 2 2]", Actual: "[3 9 6 1 6 7 4 5 2 2]"
```

更多细节详见[保存与加载](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/save_load.html)。
