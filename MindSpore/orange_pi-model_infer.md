# 模型在线推理

本章节介绍在OrangePi AIpro开发板上下载昇思MindSpore在线推理案例并执行推理的方法。

> 所有操作均在HwHiAiUser用户下执行。

## 1. 下载案例

### 步骤1 打开终端并下载案例代码

使用CTRL+ALT+T快捷键或点击页面下方带有$\_的图标打开终端，执行以下命令：

```bash
cd samples/notebooks/
git clone https://github.com/mindspore-courses/orange-pi-mindspore.git
```

### 步骤2 进入案例目录

代码包位置：`/home/HwHiAiUser/samples/notebooks`

代码包包含Online和Offline两部分。Online部分基于MindSpore框架开发，细分为：
- inference（推理案例）
- training（训练案例）
- community（第三方应用案例）

推理案例的项目目录结构如下：

```
/home/HwHiAiUser/samples/notebooks/orange-pi-mindspore/applications/online/inference
├── 01_quick_start
├── 02_resnet50
├── 03_vit
├── 04_fcn
├── 05_shufflenet
├── 06_ssd
├── 07_rnn
├── 08_lstm_crf
├── 09_gan
├── 10_dcgan
├── 11_pix2pix
├── 12_diffusion
├── 13_resnet50_transfer
├── 14_qwen1_5_0_5b
├── 15_tinyllama
├── 16_dctnet
├── 17_deepseek_r1_distill_qwen_1_5b
├── 18_deepseek_janus_pro_1b
└── 19_minicpm3
```

## 2. 推理执行

### 步骤1 启动Jupyter Lab界面

```bash
cd /home/HwHiAiUser/samples/notebooks/
./start_notebook.sh
```

执行脚本后，终端会显示登录Jupyter Lab的网址链接。打开浏览器，输入该网址链接即可登录Jupyter Lab。

### 步骤2 进入案例目录

在Jupyter Lab界面中双击案例目录（以"04_fcn"为例），进入该案例的目录。其他案例操作流程类似，仅需选择对应的案例目录和.ipynb文件即可。

### 步骤3 查看案例文件

在该目录下有运行示例所需的全部资源。其中mindspore_fcn8s.ipynb是在Jupyter Lab中运行该样例的文件，双击打开即可。

文件开头说明了硬件资源（香橙派开发板）信息，以及运行样例所需的CANN和MindSpore版本。请检查环境配置，环境的检查与搭建详见[环境搭建指南](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/orange_pi/environment_setup.html)。

### 步骤4 运行样例

单击运行按钮运行样例，在弹出的对话框中单击"Restart"按钮，样例开始运行。

## 3. 环境清理

推理执行完成后，需要在Jupyter Lab界面中进行清理操作，防止内存溢出。

导航至KERNELS > Shut Down All，手动关闭已运行的kernel，防止运行其他案例时报内存不足的错误。

## 下一步建议

具体基于昇思MindSpore的案例开发详见[开发入门](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/orange_pi/dev_start.html)。
