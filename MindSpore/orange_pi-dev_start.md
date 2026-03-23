# 开发入门

因开发者可能会在OrangePi AIpro（下称：香橙派开发板）进行自定义模型和案例开发，本章节通过基于MindSpore的手写数字识别案例，说明香橙派开发板中的开发注意事项。

## 环境准备

开发者拿到香橙派开发板后，首先需要进行硬件资源确认、镜像烧录以及CANN和MindSpore版本的升级，才可运行该案例，具体如下：

| 香橙派AIpro | 镜像 | CANN Toolkit/Ops | MindSpore |
|-----------|------|-----------------|-----------|
| 8T 16G | Ubuntu | 8.5.0 | 2.8.0 |

### 镜像烧录

运行该案例需要烧录香橙派官网Ubuntu镜像，参考镜像烧录章节。

### CANN升级

参考CANN升级章节。

### MindSpore升级

参考MindSpore升级章节。

## 数据集准备与加载

MindSpore提供基于Pipeline的数据引擎，通过数据集（Dataset）实现高效的数据预处理。在本案例中，使用MNIST数据集，自动下载完成后，使用`mindspore.dataset`提供的数据变换进行预处理。

```python
# install download
!pip install download
```

```python
# Download data from open datasets
from download import download

url = "https://mindspore-website.obs.cn-north-4.myhuaweicloud.com/" \
      "notebook/datasets/MNIST_Data.zip"
path = download(url, "./", kind="zip", replace=True)
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

输出：`['image', 'label']`

MindSpore的dataset使用数据处理流水线（Data Processing Pipeline），需指定map、batch、shuffle等操作。这里使用map对图像数据及标签进行变换处理，将输入的图像缩放为1/255，根据均值0.1307和标准差值0.3081进行归一化处理，然后将处理好的数据集打包为大小为64的batch。

```python
def datapipe(dataset, batch_size):
    image_transforms = [
        vision.Rescale(1.0 / 255.0, 0),
        vision.Normalize(mean=(0.1307,), std=(0.3081,)),
        vision.HWC2CHW(),
        transforms.TypeCast(mindspore.float16)
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

可使用create_tuple_iterator或create_dict_iterator对数据集进行迭代访问，查看数据和标签的shape和datatype。

```python
for image, label in test_dataset.create_tuple_iterator():
    print(f"Shape of image [N, C, H, W]: {image.shape} {image.dtype}")
    print(f"Shape of label: {label.shape} {label.dtype}")
    break
```

输出：

```
Shape of image [N, C, H, W]: (64, 1, 28, 28) Float16
Shape of label: (64,) Int32
```

```python
for data in test_dataset.create_dict_iterator():
    print(f"Shape of image [N, C, H, W]: {data['image'].shape} {data['image'].dtype}")
    print(f"Shape of label: {data['label'].shape} {data['label'].dtype}")
    break
```

输出：

```
Shape of image [N, C, H, W]: (64, 1, 28, 28) Float16
Shape of label: (64,) Int32
```

## 模型构建

```python
# Define model
class Network(Cell):
    def __init__(self):
        super().__init__()
        self.flatten = mint.flatten
        self.dense1 = nn.Linear(28*28, 512, dtype=mindspore.float16)
        self.dense2 = nn.Linear(512, 512, dtype=mindspore.float16)
        self.dense3 = nn.Linear(512, 10, dtype=mindspore.float16)
        self.relu = nn.ReLU()

    def construct(self, x):
        x = self.flatten(x, start_dim=1)
        x = self.dense1(x)
        x = self.relu(x)
        x = self.dense2(x)
        x = self.relu(x)
        logits = self.dense3(x)
        return logits

model = Network()
print(model)
```

## 模型训练

在模型训练中，一个完整的训练过程（step）需要实现以下三步：

1. 正向计算：模型预测结果（logits），并与正确标签（label）计算预测损失（loss）。

2. 反向传播：利用自动微分机制，自动求模型参数（parameters）对于loss的梯度（gradients）。

3. 参数优化：将梯度更新到参数上。

MindSpore使用函数式自动微分机制，因此针对上述步骤需要实现：

1. 定义正向计算函数。

2. 使用value_and_grad通过函数变换获得梯度计算函数。

3. 定义训练函数，使用set_train设置为训练模式，执行正向计算、反向传播和参数优化。

```python
# Instantiate loss function and optimizer
loss_fn = nn.CrossEntropyLoss()
optimizer = SGD(model.trainable_params(), 1e-2)

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

除训练外，定义测试函数，用来评估模型的性能。

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

训练过程需多次迭代数据集，一次完整的迭代称为一轮（epoch）。在每一轮中，遍历训练集进行训练，结束后使用测试集进行预测。打印每一轮的loss值和预测准确率（Accuracy），可以看到loss在不断下降，Accuracy在不断提高。

```python
epochs = 3
for t in range(epochs):
    print(f"Epoch {t+1}\n-------------------------------")
    train(model, train_dataset)
    test(model, test_dataset, loss_fn)
print("Done!")
```

## 保存模型

模型训练完成后，需要保存其参数。

```python
# Save checkpoint
mindspore.save_checkpoint(model, "model.ckpt")
print("Saved Model to model.ckpt")
```

## 权重加载

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

`param_not_load`是未被加载的参数列表，为空时代表所有参数均加载成功。

## 模型推理

加载后的模型可以直接用于预测推理。

```python
import matplotlib.pyplot as plt

model.set_train(False)
for data, label in test_dataset:
    pred = model(data)
    predicted = pred.argmax(1)
    print(f'Predicted: "{predicted[:6]}", Actual: "{label[:6]}"')

    # 显示数字及其预测值
    plt.figure()
    for i in range(6):
        plt.subplot(2, 3, i + 1)
        # 若预测正确，显示为蓝色；若预测错误，显示为红色
        color = 'blue' if predicted[i] == label[i] else 'red'
        plt.title(f'Predicted:{predicted[i]}', color=color)
        plt.imshow(data.asnumpy()[i][0], interpolation="None", cmap="gray")
        plt.axis('off')
    plt.show()
    break
```

本案例已同步上线GitHub仓库，更多案例可参考该仓库。

本案例运行所需环境：

| 香橙派AIpro | 镜像 | CANN Toolkit/Ops | MindSpore |
|-----------|------|-----------------|-----------|
| 8T 16G | Ubuntu | 8.5.0 | 2.8.0 |
