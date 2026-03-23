# 基本介绍

本节将会整体介绍昇思MindSpore和华为昇腾AI全栈，并介绍昇思MindSpore在华为昇腾AI全栈中的位置。对昇思MindSpore感兴趣的开发者，欢迎参与昇思MindSpore的[社区](https://atomgit.com/mindspore/mindspore)并一键三连（Watch/Star/Fork）。

## 昇思MindSpore介绍

### 整体架构

MindSpore整体架构如下：

1. 模型套件：为开发者提供开箱即用的模型和开发套件，如大模型套件MindSpore Transformers、MindSpore ONE以及科学计算库等热点研究领域拓展库；

2. 深度学习+科学计算：为开发者提供AI模型开发所需各类Python接口，最大化保持开发者在Python生态开发的使用习惯；

3. 核心：作为AI框架的核心，构建Tensor数据结构、基础运算算子Operator、自动求导autograd模块、并行计算Parallel模块、编译compile能力以及runtime运行时管理模块。

### 设计理念

昇思MindSpore是一个全场景深度学习框架，旨在实现易开发、高效执行、全场景统一部署三大目标。其中，易开发表现为API友好，调试难度低；高效执行包括计算效率、数据预处理效率和分布式训练效率；全场景则指框架同时支持云、边缘以及端侧场景。

## 华为昇腾AI全栈介绍

昇腾计算是基于昇腾系列处理器构建的全栈AI计算基础设施及应用，包括昇腾Ascend系列芯片、Atlas系列硬件、CANN芯片使能、MindSpore AI框架、ModelArts、MindX应用使能等。

华为Atlas人工智能计算解决方案，是基于昇腾系列AI处理器，通过模块、板卡、小站、服务器、集群等丰富的产品形态，打造面向"端、边、云"的全场景AI基础设施方案，涵盖数据中心解决方案、智能边缘解决方案，覆盖深度学习领域推理和训练全流程。

下面简单介绍每个模块的作用：

- 昇腾应用使能：华为各大产品线基于MindSpore提供的AI平台或服务能力。

- MindSpore：支持端、边、云独立和协同的统一训练和推理框架。

- CANN：昇腾芯片使能、驱动层（[了解更多](https://www.hiascend.com/zh/software/cann)）。

- 计算资源：昇腾系列化IP、芯片和服务器。

详细信息请点击[华为昇腾官网](https://e.huawei.com/cn/products/servers/ascend)。

## 参与社区

欢迎每一个开发者都加入到昇思MindSpore的社区中，为全场景AI框架昇思MindSpore添砖加瓦！

- 昇思MindSpore 官网：可以全方位了解昇思MindSpore，包括安装、教程、文档、社区、资源下载和资讯栏目等（[了解更多](https://www.mindspore.cn/)）。

- 昇思MindSpore 代码：
  - [MindSpore AtomGit](https://atomgit.com/mindspore/mindspore)：一键三连（Watch/Star/Fork）即可随时跟踪MindSpore最新进展，参与issues讨论、提交代码！
  - [MindSpore GitHub](https://github.com/mindspore-ai/mindspore)：AtomGit的MindSpore代码镜像，习惯用GitHub的开发者可以在这里学习MindSpore，查看最新代码实现！

- 昇思MindSpore 论坛：我们努力地服务好每一个开发者，在昇思MindSpore中，无论是入门开发者还是高手大咖都能找到知音，共同学习，共同成长！（[了解更多](https://discuss.mindspore.cn/)）
