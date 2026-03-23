# 模型训练

模型训练一般分为四个步骤：

1. 构建数据集。
2. 定义神经网络模型。
3. 定义超参、损失函数及优化器。
4. 输入数据集进行训练与评估。

现在我们有了数据集和模型后，可以进行模型的训练与评估。

## 构建数据集

首先从[数据加载与处理](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/dataset.html)加载代码，构建数据集。

```python
import mindspore
from mindspore import nn
from mindspore.dataset import vision, transforms
from mindspore.dataset import MnistDataset

# Download data from open datasets
from download import download

url = "https://mindspore-website.obs.cn-north-4.myhuaweicloud.com/" \
      "notebook/datasets/MNIST_Data.zip"
path = download(url, "./", kind="zip", replace=True)

def datapipe(path, batch_size):
    image_transforms = [
        vision.Rescale(1.0 / 255.0, 0),
        vision.Normalize(mean=(0.1307,), std=(0.3081,)),
        vision.HWC2CHW()
    ]
    label_transform = transforms.TypeCast(mindspore.int32)

    dataset = MnistDataset(path)
    dataset = dataset.map(image_transforms, 'image')
    dataset = dataset.map(label_transform, 'label')
    dataset = dataset.batch(batch_size)
    return dataset

train_dataset = datapipe('MNIST_Data/train', batch_size=64)
test_dataset = datapipe('MNIST_Data/test', batch_size=64)
```

## 定义神经网络模型

从[网络构建](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/beginner/model.html)中加载代码，构建一个神经网络模型。

```python
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
```

## 定义超参、损失函数和优化器

### 超参

超参（Hyperparameters）是可以调整的参数，可以控制模型训练优化的过程，不同的超参数值可能会影响模型训练和收敛速度。目前深度学习模型多采用批量随机梯度下降算法进行优化，其原理如下：

$$w_{t+1}=w_{t}-\eta \frac{1}{n} \sum_{x \in \mathcal{B}} \nabla l\left(x, w_{t}\right)$$

公式中，$n$是批量大小（batch size），$\eta$是学习率（learning rate）。另外，$w_{t}$为训练轮次$t$中的权重参数，$\nabla l$为损失函数的导数。除了梯度本身，这两个因子直接决定了模型的权重更新，从优化本身来看，它们是影响模型性能收敛最重要的参数。一般会定义以下超参用于训练：

- 训练轮次（epoch）：训练时遍历数据集的次数。
- 批次大小（batch size）：数据集进行分批读取训练，设定每个批次数据的大小。batch size过小，训练时间较长，且梯度震荡严重，不利于收敛；batch size过大，不同batch的梯度方向变化小，容易陷入局部极小值。因此，选择合适的batch size，可以有效提高模型精度和全局收敛性。
- 学习率（learning rate）：学习率偏小，会导致收敛的速度变慢；学习率偏大，则可能会导致训练不收敛等不可预测的结果。梯度下降法被广泛应用在最小化模型误差的参数优化算法上。通过多次迭代，并在每一步中最小化损失函数来预估模型的参数。学习率用于控制模型在迭代过程中的学习进度。

```python
epochs = 3
batch_size = 64
learning_rate = 1e-2
```

### 损失函数

损失函数（loss function）用于评估模型的预测值（logits）和目标值（targets）之间的误差。训练模型时，随机初始化的神经网络模型开始时会预测出错误的结果。损失函数会评估预测结果与目标值的相异程度，模型训练的目标即为降低损失函数求得的误差。

常见的损失函数包括用于回归任务的[mindspore.nn.MSELoss](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.MSELoss.html)（均方误差）和用于分类的[mindspore.nn.NLLLoss](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.NLLLoss.html)（负对数似然）等。[mindspore.nn.CrossEntropyLoss](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.CrossEntropyLoss.html)结合了[mindspore.nn.LogSoftmax](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.LogSoftmax.html)和[mindspore.nn.NLLLoss](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/nn/mindspore.nn.NLLLoss.html)，可以对logits进行归一化并计算预测误差。

```python
loss_fn = nn.CrossEntropyLoss()
```

### 优化器

模型优化（Optimization）是在每个训练步骤中调整模型参数以减少模型误差的过程。MindSpore提供多种优化算法的实现，称之为优化器（Optimizer）。优化器内部定义了模型的参数优化过程（即梯度如何更新至模型参数），所有优化逻辑都封装在优化器对象中。在这里，我们使用SGD（Stochastic Gradient Descent）优化器。

我们通过`model.trainable_params()`方法获得模型的可训练参数，并传入学习率超参来初始化优化器。

```python
optimizer = nn.SGD(model.trainable_params(), learning_rate=learning_rate)
```

> 在训练过程中，通过微分函数可计算获得参数对应的梯度，将其传入优化器中即可实现参数优化，具体形态如下：
>
> grads = grad_fn(inputs)
>
> optimizer(grads)

## 训练与评估

设置了超参、损失函数和优化器后，我们就可以循环输入数据来训练模型。完整遍历一次数据集称为一轮（epoch）。每轮训练包括两个步骤：

1. 训练：迭代训练数据集，并尝试收敛到最佳参数。
2. 验证/测试：迭代测试数据集，以检查模型性能是否提升。

接下来，定义用于训练的`train_loop`函数和用于测试的`test_loop`函数。

使用函数式自动微分，需先定义正向函数`forward_fn`，再使用[value_and_grad](https://www.mindspore.cn/docs/zh-CN/r2.8.0/api_python/mindspore/mindspore.value_and_grad.html)获取微分函数`grad_fn`。然后，我们将微分函数和优化器的执行封装为`train_step`函数，最后循环迭代数据集进行训练。

```python
# Define forward function
def forward_fn(data, label):
    logits = model(data)
    loss = loss_fn(logits, label)
    return loss, logits

# Get gradient function
grad_fn = mindspore.value_and_grad(forward_fn, None, optimizer.parameters, has_aux=True)

# Define function of one-step training
def train_step(data, label):
    (loss, _), grads = grad_fn(data, label)
    optimizer(grads)
    return loss

def train_loop(model, dataset):
    size = dataset.get_dataset_size()
    model.set_train()
    for batch, (data, label) in enumerate(dataset.create_tuple_iterator()):
        loss = train_step(data, label)

        if batch % 100 == 0:
            loss, current = loss.asnumpy(), batch
            print(f"loss: {loss:>7f}  [{current:>3d}/{size:>3d}]")
```

`test_loop`函数同样需循环遍历数据集，调用模型计算loss和Accuracy，并返回最终结果。

```python
def test_loop(model, dataset, loss_fn):
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

我们将实例化的损失函数和优化器传入`train_loop`和`test_loop`中。训练3轮，并输出loss和Accuracy，观察性能变化。

```python
loss_fn = nn.CrossEntropyLoss()
optimizer = nn.SGD(model.trainable_params(), learning_rate=learning_rate)

for t in range(epochs):
    print(f"Epoch {t+1}\n-------------------------------")
    train_loop(model, train_dataset)
    test_loop(model, test_dataset, loss_fn)
print("Done!")
```

```
Epoch 1
-------------------------------
loss: 2.302806  [  0/938]
loss: 2.285086  [100/938]
loss: 2.264712  [200/938]
loss: 2.174010  [300/938]
loss: 1.931853  [400/938]
loss: 1.340721  [500/938]
loss: 0.953515  [600/938]
loss: 0.756860  [700/938]
loss: 0.756263  [800/938]
loss: 0.463846  [900/938]
Test:
 Accuracy: 84.7%, Avg loss: 0.527155

Epoch 2
-------------------------------
loss: 0.479126  [  0/938]
loss: 0.437443  [100/938]
loss: 0.685504  [200/938]
loss: 0.395121  [300/938]
loss: 0.550566  [400/938]
loss: 0.459457  [500/938]
loss: 0.293049  [600/938]
loss: 0.422102  [700/938]
loss: 0.333153  [800/938]
loss: 0.412182  [900/938]
Test:
 Accuracy: 90.5%, Avg loss: 0.335083

Epoch 3
-------------------------------
loss: 0.207366  [  0/938]
loss: 0.343559  [100/938]
loss: 0.391145  [200/938]
loss: 0.317566  [300/938]
loss: 0.200746  [400/938]
loss: 0.445798  [500/938]
loss: 0.603720  [600/938]
loss: 0.170811  [700/938]
loss: 0.411954  [800/938]
loss: 0.315902  [900/938]
Test:
 Accuracy: 91.9%, Avg loss: 0.279034

Done!
```
