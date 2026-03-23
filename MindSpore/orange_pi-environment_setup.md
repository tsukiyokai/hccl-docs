# 环境搭建指南

## 1. 镜像烧录（以Windows系统为例）

镜像烧录可以在任何操作系统内执行。这里将以Windows系统为例，演示使用相应版本的balenaEtcher工具，快速烧录镜像至Micro SD卡。

### 1.1 制卡前准备

步骤1 将Micro SD卡插入读卡器，并将读卡器插入PC。

### 1.2 下载Ubuntu镜像

步骤1 点击[此链接](http://www.orangepi.cn/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-AIpro.html)进入镜像下载页面。

> 此处仅做示意，不同算力开发板镜像下载地址不同，详细请查看[此链接](http://www.orangepi.cn/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-AIpro(20T).html)。

步骤2 点击图片中箭头图标，跳转百度网盘下载页面。

步骤3 选择桌面版本下载，建议下载0318版本环境。

步骤4 备选下载方式。

如果百度网盘下载过慢，可以使用[此链接](https://obs-9be7.obs.cn-east-2.myhuaweicloud.com/OrangePi/20240318/opiaipro_ubuntu22.04_desktop_aarch64_20240318.img.xz)直接下载。

### 1.3 下载制卡工具

有两种制卡工具balenaEtcher、Rufus，可根据自己电脑情况任选一种工具进行烧录。

#### balenaEtcher制卡工具

步骤1 下载balenaEtcher。

点击[此链接](https://etcher.balena.io/)可跳转到软件官网，点击绿色的下载按钮会跳到软件下载的地方。

步骤2 选择下载Portable版本。

Portable版本无需安装，双击打开即可使用。

步骤3 备选下载方式。

如果官方网站下载过慢，可以使用[此链接](https://obs-9be7.obs.cn-east-2.myhuaweicloud.com/OrangePi/balenaEtcher/balenaEtcher-Setup-1.18.4.exe)直接下载balenaEtcher-Setup-1.18.4软件。

步骤4 打开balenaEtcher。

#### Rufus制卡工具

步骤1 Rufus下载。

点击[此链接](https://github.com/pbatard/rufus/releases/download/v4.5/rufus-4.5.exe)，进行下载、安装。

### 1.4 选择和烧录镜像

#### balenaEtcher烧录镜像

步骤1 选择镜像、TF卡，启动烧录。

1. 选择要烧录的镜像文件（上文1.2下载Ubuntu镜像的保存路径）。
2. 选择TF卡的盘符。
3. 点击开始烧录。

烧录和验证大概需要20分钟左右，请耐心等待。

步骤2 烧录完成。

烧录完成后，balenaEtcher的显示界面会出现绿色的指示图标，说明镜像烧录成功。此时就可以退出balenaEtcher，拔出TF卡，插入到开发板的TF卡槽中使用。

#### Rufus烧录镜像

步骤1 选择镜像、TF卡，烧录镜像。

SD卡插入读卡器，读卡器插入电脑、选择镜像与SD卡，点击"开始"。

步骤2 烧录完成。

等待结束后直接拔出读卡器。

## 2. 启动开发板和网络连接

### 2.1 启动开发板

将烧录好镜像的TF卡插入开发板对应的插槽中，参考[香橙派官方资料](http://www.orangepi.cn/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-AIpro.html)中的《用户手册》，具体可参考"2.8 启动开发板的步骤"。

### 2.2 网络连接

为方便后续环境升级、下载软件包和运行代码，开发板需要完成网络连接。具体可参考《用户手册》"3.5 网络连接测试"。

### 2.3 SSH远程登录开发板（可选）

如果需要通过SSH远程方式登录开发板，具体可参考《用户手册》"3.6 SSH远程登录开发板"。

### 2.4 设置Swap内存（可选）

虽然开发板有8GB或16GB的大内存，但有些应用需要的内存大于8GB或16GB，此时可以通过Swap内存来扩展系统能使用的最大内存容量。具体可参考《用户手册》"3.20 设置Swap内存的方法"。

## 3. CANN升级

### 3.1 CANN版本查询

步骤1 使用`CTRL+ALT+T`快捷键或点击页面下方带有`$_`的图标打开终端，保持HwHiAiUser用户登录状态。

步骤2 执行以下命令获取版本信息。

```bash
(base) HwHiAiUser@orangepiaipro:~$ cat /usr/local/Ascend/ascend-toolkit/latest/aarch64-linux/ascend_toolkit_install.info
```

### 3.2 CANN升级（以CANN 8.5.0为例）

若当前CANN版本不满足开发需求，可按照如下内容对CANN版本进行升级。

#### 3.2.1 Toolkit升级

步骤1 使用`CTRL+ALT+T`快捷键或点击页面下方带有`$_`的图标打开终端，并切换root用户。

切换root用户，root用户密码：Mind@123。

```bash
# 打开开发板的一个终端，运行如下命令
(base) HwHiAiUser@orangepiaipro:~$ su - root
Password:
(base) root@orangepiaipro: ~#
```

步骤2 删除已安装CANN包释放磁盘空间，防止安装新的CANN包时报错磁盘空间不足。

```bash
(base) root@orangepiaipro: ~# cd /usr/local/Ascend/ascend-toolkit
(base) root@orangepiaipro: /usr/local/Ascend/ascend-toolkit # rm -rf *
```

步骤3 打开昇腾CANN官网访问社区版资源[下载地址](https://www.hiascend.com/developer/download/community/result?module=cann)，下载所需版本的toolkit包，该处以8.5.0版本为例。

> 执行如下命令，根据环境实际输出信息选择aarch64或是x86_64对应的软件包。

```bash
uname -a
```

步骤4 进入Toolkit包下载目录。

```bash
(base) root@orangepiaipro: /usr/local/Ascend/ascend-toolkit# cd /home/HwHiAiUser/Downloads
```

> OrangePi AIpro浏览器文件默认下载目录：/home/HwHiAiUser/Downloads，用户在更换保存路径时请同步修改上述命令中的路径。

步骤5 给CANN包添加执行权限。

```bash
(base) root@orangepiaipro: /home/HwHiAiUser/Downloads# chmod +x ./Ascend-cann-toolkit_8.5.0_linux-aarch64.run
```

步骤6 执行以下命令升级软件。

```bash
(base) root@orangepiaipro: /home/HwHiAiUser/Downloads# ./Ascend-cann-toolkit_8.5.0_linux-aarch64.run --install
```

安装时弹出此提示后输入Y，然后按回车键继续安装。该过程约需要10-15分钟，请耐心等待。

升级完成后，若显示如下信息，则说明软件升级成功：

```
xxx install success
```

- xxx表示升级的实际软件包名。
- 安装升级后的路径（以root用户默认安装升级路径为例）：/usr/local/Ascend/ascend-toolkit/

> 如果输出不正确，需要按照以上步骤重新升级Toolkit。

步骤7 加载环境变量。

```bash
(base) root@orangepiaipro: /home/HwHiAiUser/Downloads # source ~/.bashrc
```

#### 3.2.2 Ops升级

> 二进制算子包ops依赖CANN软件包Toolkit，执行升级时，当前环境需已安装配套版本的Toolkit，并使用root用户安装。

步骤1 打开终端，并切换root用户。

root用户密码：Mind@123。

```bash
# 打开开发板的一个终端，运行如下命令
(base) HwHiAiUser@orangepiaipro:~$ su - root
Password:
(base) root@orangepiaipro: ~#
```

步骤2 执行如下命令，获取开发板NPU型号。

```bash
npu-smi info
```

步骤3 打开昇腾CANN官网访问社区版资源[下载地址](https://www.hiascend.com/developer/download/community/result?module=cann)，下载与CANN包版本一致，并且匹配NPU型号的ops包。

步骤4 进入ops包下载目录。

```bash
(base) root@orangepiaipro: /usr/local/Ascend/ascend-toolkit# cd /home/HwHiAiUser/Downloads
```

> OrangePi AIpro浏览器文件默认下载目录：/home/HwHiAiUser/Downloads

步骤5 给ops包添加执行权限。

```bash
(base) root@orangepiaipro: /home/HwHiAiUser/Downloads# chmod +x ./Ascend-cann-310b-ops_8.5.0_linux-aarch64.run
```

步骤6 执行以下命令升级软件。

```bash
(base) root@orangepiaipro: /home/HwHiAiUser/Downloads# ./Ascend-cann-310b-ops_8.5.0_linux-aarch64.run --install
```

升级完成后，若显示如下信息，则说明软件升级成功：

```
xxx install success
```

- xxx表示升级的实际软件包名。

> 如果输出不正确，需要按照以上步骤重新升级ops。

## 4. MindSpore升级

### 4.1 MindSpore版本查询

步骤1 使用CTRL+ALT+T快捷键或点击页面下方带有$_的图标打开终端，保持HwHiAiUser用户登录状态。

步骤2 执行如下命令，查询当前MindSpore版本。

```bash
(base) HwHiAiUser@orangepiaipro:~$ pip show mindspore
```

若当前MindSpore版本不满足开发需求，可按照如下内容对MindSpore版本进行升级。

### 4.2 升级MindSpore（以MindSpore 2.8.0为例）

#### 4.2.1 升级MindSpore

请参考MindSpore官网[安装教程](https://www.mindspore.cn/install)安装MindSpore 2.8.0。

#### 4.2.2 验证是否成功安装

执行以下命令：

```bash
(base) HwHiAiUser@orangepiaipro:~$ python -c "import mindspore;mindspore.set_context(device_target='Ascend');mindspore.run_check()"
```

如果输出如下，说明MindSpore安装成功了。

```
MindSpore version: 2.8.0
The result of multiplication calculation is correct, MindSpore has been installed on platform [Ascend] successfully!
```

> 如果输出不正确，需要按照4.2.1 MindSpore升级方式重新升级。

## 下一步建议

此时香橙派开发板环境搭建已经完成，可以在开发板上体验基于昇思MindSpore开发的[模型在线推理](https://www.mindspore.cn/tutorials/zh-CN/r2.8.0/orange_pi/model_infer.html)。
