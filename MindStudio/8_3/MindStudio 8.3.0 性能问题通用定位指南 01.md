# MindStudio8.3.0性能问题通用定位指南

文档版本 01  
发布日期 2026-03-03

![](images/20eaff4b4955ea9a205e9cfe24be8477b9d108c09d001ee5347203574d456bc7.jpg)

版权所有 $\circledcirc$ 华为技术有限公司 2026。 保留一切权利。

非经本公司书面许可，任何单位和个人不得擅自摘抄、复制本文档内容的部分或全部，并不得以任何形式传播。

# 商标声明

和其他华为商标均为华为技术有限公司的商标。  
本文档提及的其他所有商标或注册商标，由各自的所有人拥有。

# 注意

您购买的产品、服务或特性等应受华为公司商业合同和条款的约束，本文档中描述的全部或部分产品、服务或特性可能不在您的购买或使用范围之内。除非合同另有约定，华为公司对本文档内容不做任何明示或暗示的声明或保证。

由于产品版本升级或其他原因，本文档内容会不定期进行更新。除非另有约定，本文档仅作为使用指导，本文档中的所有陈述、信息和建议不构成任何明示或暗示的担保。

# 安全声明

# 产品生命周期政策

华为公司对产品生命周期的规定以“产品生命周期终止政策”为准，该政策的详细内容请参见如下网址：https://support.huawei.com/ecolumnsweb/zh/warranty-policy

# 漏洞处理流程

华为公司对产品漏洞管理的规定以“漏洞处理流程”为准，该流程的详细内容请参见如下网址：  
https://www.huawei.com/cn/psirt/vul-response-process  
如企业客户须获取漏洞信息，请参见如下网址：  
https://securitybulletin.huawei.com/enterprise/cn/security-advisory

# 华为初始证书权责说明

华为公司对随设备出厂的初始数字证书，发布了“华为设备初始数字证书权责说明”，该说明的详细内容请参见如下网址：https://support.huawei.com/enterprise/zh/bulletins-service/ENEWS2000015766

# 华为企业业务最终用户许可协议(EULA)

本最终用户许可协议是最终用户（个人、公司或其他任何实体）与华为公司就华为软件的使用所缔结的协议。最终用户对华为软件的使用受本协议约束，该协议的详细内容请参见如下网址：  
https://e.huawei.com/cn/about/eula

# 产品资料生命周期策略

华为公司针对随产品版本发布的售后客户资料（产品资料），发布了“产品资料生命周期策略”，该策略的详细内容请参见如下网址：https://support.huawei.com/enterprise/zh/bulletins-website/ENEWS2000017760

# 目 录

#

# 1 文档简介....

2 概述.....

# 3 性能问题的定位流程... 4

3.1 整体定位流程.. 4

3.2 问题信息收集. 4

3.3 排查思路介绍.

3.3.1 快速排查.. 6

3.3.2 详细排查. 6

# 4 性能工具的使用. 11

4.1 性能工具总体介绍. 11  
4.2 模型调优工具. 14  
4.2.1 使用步骤. 14  
4.2.2 模型调优性能采集工具. .14  
4.2.3 模型调优快速分析（msprof-analyze 命令行工具） 16  
4.2.4 模型调优深入分析（MindStudio Insight） 18  
4.2.4.1 总体介绍.. 19  
4.2.4.2 获取软件包. 19  
4.2.4.3 集群性能分析. .20  
4.2.4.3.1 概览（Summary） 20  
4.2.4.3.2 通信（Communication） .23  
4.2.4.4 单卡性能分析. .26  
4.2.4.4.1 时间线（Timeline） . 26  
4.2.4.4.2 内存（Memory）. 30  
4.2.4.4.3 算子（Operator） 31  
4.3 服务化工具.. 32

# 5 TopN 性能问题的解决方案.. .34

5.1 简介... 34

5.2 通信问题优化方案. 35

5.2.1 总体介绍.. . 35

5.2.2 快慢卡问题定位方法.. 37

5.2.2.1 总体介绍.. 37

5.2.2.2 快慢卡定点精确分析法. 37

目 录  
5.2.2.3 快慢卡定位 Timeline 操作案例. ..... 40  
5.2.2.4 快慢卡定位算子比对操作案例. . 43  
5.2.2.5 快慢卡案例补充... ... 44  
5.2.2.5.1 案例一：集群保存 checkpoint 后性能持续劣化.. .. 44  
5.2.2.5.2 案例二：集群 AllReduce 通信算子等待时间过长. . 46  
5.2.3 通信重传... .... 49  
5.2.3.1 基于集群数据分析是否出现通信重传.. ... 50  
5.2.3.2 通信重传典型案例.. .. 51  
5.2.4 通信小包... .. 55  
5.2.4.1 案例：ZeRO3 模式导致通信小包.. .. 55  
5.2.5 源地址与目的地址字节未对齐.. .. 56  
5.2.6 计算通信带宽抢占...... . 58  
5.2.7 Profiling 采集性能膨胀.. ... 58  
5.3 算子性能问题优化方案.. .. 58  
5.3.1 算子性能问题定位方法.. .. 59  
5.3.2 亲和算子优化策略.. . 60  
5.3.3 模型代码优化策略.. .... 64  
5.4 Host Bound 问题定位及解决方法.. . 67  
5.4.1 Host Bound 问题分类.. . 67  
5.4.2 流水优化... ....... 69  
5.4.3 绑核优化.. . 69  
5.4.4 编译优化... ....... 70  
5.4.4.1 编译优化包获取.. .. 70  
5.4.4.2 毕昇编译器安装.. .. 70  
5.4.4.3 Python 编译优化..... ..... 71  
5.4.4.3.1 安装依赖.. .. 71  
5.4.4.3.2 编译 Python.... ..... 71  
5.4.4.3.3 配置软链接.. .. 72  
5.4.4.3.4 注意事项.. . 72  
5.4.4.4 torch 及 torch_npu 编译优化.. . 72  
5.4.4.4.1 前置准备... . 72  
5.4.4.4.2 首次编译（插桩编译） ...... 73  
5.4.4.4.3 采集 Profiling 数据.. . 73  
5.4.4.4.4 二次编译（优化编译） .... 73  
5.4.5 下发异常分析... .. 74  
5.4.5.1 问题背景.. . 74  
5.4.5.2 前置检查.. .. 74  
5.4.5.3 正式分析....... .... 75  
5.4.5.3.1 CPU 运行状态... ....... 75  
5.4.5.3.2 异常堆栈捕获.. . 78  
5.5 集群性能异常波动问题定位方法论. . 78  
5.5.1 集群性能劣化分析思路.. ... 79  
5.5.2 定位实操技巧... . 80  
5.6 ONNX 离线推理场景解决方案. .80  
5.6.1 信息收集. .. 81  
5.6.2 问题分析.. . 82  
5.6.2.1 分析性能数据文件.. . 82  
5.6.2.2 ATC 转换日志分析.. . 84  
5.6.2.3 OM 和 ONNX 文件.. 84  
5.6.3 优化方法. . 85  
5.6.3.1 模型压缩量化.. . 85  
5.6.3.2 ONNX 模型调优.. .86  
5.6.3.2.1 简化 ONNX 文件. 86  
5.6.3.2.2 ATC 调优.. . 87  
5.6.3.2.3 AOE 调优.. . 87  
5.6.3.2.4 NCS 调优.. 88  
5.6.3.3 msit debug surgeon 自动优化 ONNX.. 91  
5.6.3.4 升级 CANN 版本. . 92  
5.6.3.5 改图优化.. 93  
5.7 MindIE 推理性能解决方案. . 94  
5.7.1 整体思路.. .. 94  
5.7.2 纯模型性能调优.. . 95  
5.7.3 服务化性能调优方法论. . 96  
5.7.3.1 明确优化目标.. .... 96  
5.7.3.2 针对目标进行调优.. . 98  
5.7.4 服务化性能调优定位案例.. . 99  
5.7.4.1 框架调度耗时导致性能劣化.. . 100  
5.7.4.2 模型执行耗时优化案例. . 103  
5.7.4.3 显存瓶颈分析案例. .104  
5.7.4.4 服务化参数优化案例. 106  
5.7.5 DeepSeek 进阶调优.. . 108  
5.8 版本升级性能劣化定位方法论.. . 109  
5.8.1 确认主要性能瓶颈.. 109  
5.8.2 算子性能劣化对比分析. . 109  
5.8.3 通信时间（未被掩盖的）劣化对比分析. . 110  
5.8.4 内存劣化对比分析.. . 111  
5.8.5 空闲时间劣化对比分析. 111

# 文档简介

本文档旨在为昇腾开发者提供一套系统化的性能优化方法与工具链的使用指南，通过性能问题的认知、 定位分析 、工具使用及典型案例四个环节，实现全流程的闭环管理。

表 1-1 性能问题闭环管理  

<table><tr><td rowspan=1 colspan=1>环节名称</td><td rowspan=1 colspan=1>核心目标</td></tr><tr><td rowspan=1 colspan=1>2概述</td><td rowspan=1 colspan=1>建立昇腾平台性能问题的整体认知。</td></tr><tr><td rowspan=1 colspan=1>3性能问题的定位流程</td><td rowspan=1 colspan=1>构建昇腾平台性能问题的系统化分析流程。</td></tr><tr><td rowspan=1 colspan=1>4 性能工具的使用</td><td rowspan=1 colspan=1>掌握性能数据采集与分析的工具链。</td></tr><tr><td rowspan=1 colspan=1>5TopN性能问题的解决方案</td><td rowspan=1 colspan=1>针对高频出现的性能瓶颈，提出切实可行的优化方案。</td></tr></table>

# 2 概述

随着人工智能模型规模的不断扩大及应用场景的日益复杂，在昇腾AI计算平台上进行训练和部署时，面临着Host-Device协同效率不高、重要算子性能下滑、通信延迟增大、模型下发效率低等诸多挑战。因此，深度学习系统的性能瓶颈也从计算能力提升，转变为对硬件平台、软件栈、通信机制以及模型结构之间的协同效率进行全面优化的需求。

在此背景下，迫切需要构建一套系统性的性能分析与优化框架，涵盖性能数据采集、算子层面优化、调度策略调整、通信机制改善以及模型编译下发等多个环节。性能优化不仅成为提高昇腾产品竞争力和用户满意度的重要途径，也是推动昇腾软硬件平台不断进步的关键引擎。通过对训练与推理任务进行全流程、系统化的性能优化，可显著提升各种典型模型的执行效率，加快开发和迭代周期。

# 性能优化原则

性能优化原则需遵循算子优先、昇腾亲和优化策略、模型设计策略，具体请参见表2-1。

表 2-1 性能优化原则  

<table><tr><td rowspan=1 colspan=1>原则名称</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>算子优先</td><td rowspan=1 colspan=1>算子能力是基础，唯有优秀的算子能力才能保障单机和集群性能的卓越表现。</td></tr><tr><td rowspan=1 colspan=1>昇腾亲和优化策略</td><td rowspan=1 colspan=1>昇腾AI处理器基于高度并行的架构设计，在指令级并行和数据搬运效率方面进行了深度优化。例如，在数据访问单元设计上，昇腾的Cache Line大小达到512字节，显著高于业界常见的32字节水平，能够有效提升大粒度数据传输的带宽利用率，降低访存延迟。因此，在编程与算子优化过程中，建议尽可能对齐硬件特性，提升数据局部性，使每次内存操作处理更大规模的数据，从而充分发挥高带宽、大吞吐的优势。</td></tr><tr><td rowspan=1 colspan=1>模型设计策略</td><td rowspan=1 colspan=1>模型应尽可能多地利用矩阵运算，并充分复用AICore（矩阵计算单元），以提升整体效率。</td></tr></table>

# 性能优化方向

性能优化目标可以从计算、通信、下发以及服务化调度四个维度进行优化，具体请参见表2-2。

表 2-2 性能优化方向  

<table><tr><td rowspan=1 colspan=1>维度</td><td rowspan=1 colspan=1>优化方向</td></tr><tr><td rowspan=4 colspan=1>计算</td><td rowspan=1 colspan=1>算子性能符合预期(矩阵乘计算利用率，MTE搬运流水利用率等)。</td></tr><tr><td rowspan=1 colspan=1>计算集中于Al Core，充分利用Cube资源。</td></tr><tr><td rowspan=1 colspan=1>消除AICPU算子和非亲和算子及算法逻辑优化。</td></tr><tr><td rowspan=1 colspan=1>充分利用融合算子。</td></tr><tr><td rowspan=3 colspan=1>通信</td><td rowspan=1 colspan=1>通信带宽符合预期，未发生通信重传等问题。</td></tr><tr><td rowspan=1 colspan=1>各卡通信时间相对均衡，无明显快慢卡问题。</td></tr><tr><td rowspan=1 colspan=1>通算并行，计算尽可能掩盖通信时间。</td></tr><tr><td rowspan=3 colspan=1>下发</td><td rowspan=1 colspan=1>Free占比尽可能小。</td></tr><tr><td rowspan=1 colspan=1>计算掩盖调度时间。</td></tr><tr><td rowspan=1 colspan=1>IO/内存问题消除。</td></tr><tr><td rowspan=3 colspan=1>服务化推理</td><td rowspan=1 colspan=1>模型推理部分时延逼近纯模型性能。</td></tr><tr><td rowspan=1 colspan=1>最小化Batch间CPU任务占比。</td></tr><tr><td rowspan=1 colspan=1>调优调度参数和batch上限在时延约束下显存占满最大化吞吐。</td></tr></table>

# 3 性能问题的定位流程

整体定位流程问题信息收集排查思路介绍

# 3.1 整体定位流程

性能优化总体思路围绕性能优化方向展开，具体步骤如下。

# 说明

性能优化的前提是不造成精度劣化，特殊情况下，需要对齐精度劣化是否能接受。

1. 问题明确：请参考3.2 问题信息收集章节，收集必要信息。

2. 性能问题排查：排查思路请参见3.3 排查思路介绍。

3. 实验验证：在进行多次实验时，应严格控制变量，确保除了改变的策略外，其他参数和数据保持一致。

# 3.2 问题信息收集

在定位问题之前，务必收集准确的问题信息，具体请参考表3-1。

表 3-1 问题信息收集模板  

<table><tr><td colspan="1" rowspan="1">类别</td><td colspan="1" rowspan="1">主要信息</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="4">基本信息</td><td colspan="1" rowspan="1">模型类型</td><td colspan="1" rowspan="1">了解模型结构（类Llama、类GPT、是否是MoE等）。</td></tr><tr><td colspan="1" rowspan="1">作业规模</td><td colspan="1" rowspan="1">卡数、机器数。</td></tr><tr><td colspan="1" rowspan="1">并行策略</td><td colspan="1" rowspan="1">具体并行参数配置。</td></tr><tr><td colspan="1" rowspan="1">框架和版本</td><td colspan="1" rowspan="1">·明确CANN、MindSpore或PyTorch的版本。：需确认近期是否有版本变更，以确定问题出现在版本变更之前还是之后。</td></tr><tr><td colspan="1" rowspan="2">关键问题描述</td><td colspan="1" rowspan="1">问题场景</td><td colspan="1" rowspan="1">在模型训练或推理过程中，其表现未达到预期标准或竞品水平，或者出现了性能下降等异常情况。O 性能不及预期，一般出现在模型迁移后，相较于竞品性能不及预期。性能出现抖动，模型长稳训练过程中，随机或伴随特定事件出现性能波动。集群线性度不足，集群规模扩大后，模型性能没有按照预期增长。纯模型性能差，相同配置下纯模型性能异常，参考训练问题解决。·服务化调度调优。</td></tr><tr><td colspan="1" rowspan="1">当前性能指标</td><td colspan="1" rowspan="1">明确当前性能问题的状况，计算性能指标的优先级排序请参考《PyTorch训练模型迁移调优指南》中的“性能概述&gt;性能指标&gt;性能指标介绍”章节。</td></tr><tr><td colspan="1" rowspan="1">优化目标</td><td colspan="1" rowspan="1">性能优化的目标</td><td colspan="1" rowspan="1">明确优化目标及来源，例如是否基于竞品对比，还是通过线性度推算等。说明若优化中涉及增大batchsize等方法，尽量避免使用单step时间等指标进行衡量，需参考《PyTorch训练模型迁移调优指南》中的“性能概述&gt;性能指标&gt;性能指标介绍”章节，选取合理指标替换。</td></tr></table>

# 3.3 排查思路介绍

训练场景和推理场景中纯模型问题排查策略相同，可以通过3.3.1 快速排查初步定位，若无法定位问题，再使用3.3.2 详细排查具体定位。

推理场景中服务化问题可参考4.3 服务化工具、5.7.3 服务化性能调优方法论中的问题。

# 说明

推理和训练中的大规模集群场景，均优先考虑通过集群分析工具、N分集群等方式，将集群问题转为小规模问题，找到问题复现最小单元后再按照本节方法进行定位。具体过程可参考5.5 集群性能异常波动问题定位方法论。

![](images/28b4019a15a148b2d55ca1ad5892522c2340c1ea96d8a00a89bfb6c8c32b5883.jpg)  
图 3-1 排查思路

# 3.3.1 快速排查

快速排查分为开箱和长稳两种场景，具体介绍如下：

1. 开箱场景：开箱性能问题场景一般指首次加载模型。此时需要明确性能优化目标，如果存在竞品性能参考，则可以通过Profiling等工具详细对比二者差异。建议在正式拉起任务前，先考虑并行策略寻优，确认最优并行策略。若无法解决问题，参考长稳场景定位方案。

2. 长稳场景：长期稳定性下的性能问题通常指的是系统在一段时间内表现良好且没有性能问题的情况下，突然出现性能下降或性能问题。

a. 变更排查：明确近期是否进行过变更，包括但不限于集群重新规划、版本变更等。若性能问题在变更后出现，条件允许的情况下，可以尝试撤销近期变更，若确认问题由变更引起，建议优先考虑版本问题或变更涉及操作（如重启）等对集群可能的影响，具体请参见5.8 版本升级性能劣化定位方法论。

b. 硬件排查：当性能出现波动时，应检查相应时间点是否有硬件问题，例如NPU降频、网络丢包等硬件告警。注意，这里的硬件排查仅指初步排查，主要关注硬件告警。若无硬件告警或如网络丢包等关键事件，则参考3.3.2 详细排查进行定位

# 3.3.2 详细排查

详细排查主要围绕下发、通信、计算三类常见问题开展，性能工具的使用请参见4.2 模型调优工具。

# 说明

● 使用msprof-analyze工具初步分析，粗粒度定位性能问题，并为后续的深入分析提供明确的方向，具体请参见4.2.3 模型调优快速分析（msprof-analyze命令行工具）

通过MindStudio Insight工具进一步识别瓶颈点，深入剖析问题根源。具体请参见4.2.4 模型调优深入分析（MindStudio Insight）。

![](images/5db4095eafe34da6ea007999daa9e220ce37ae4ef317f936a60326d07deda81e.jpg)  
图 3-2 详细排查流程图

# 下发问题

下发问题是指算子下发过程耗时异常。下发指Graph Engine将算子执行请求下发至Runtime，Runtime判断算子的Task类型，根据类型将算子执行请求下发到Device上执行，详细说明请参考《TBE&AI CPU算子开发指南》中的“背景知识 > 算子编译运行流程”章节。

理想情况下，NPU侧的计算流水不停运转，不会出现NPU等待CPU的场景，一旦出现下发延迟，将导致流水线阻塞，从而影响AI Core的算力利用率。此时即判定存在下发问题。下发问题主要通过MindStudio Insight工具中4.2.4.4.1 时间线（Timeline）进行观察，若存在下列任一现象，请参考5.4.5 下发异常分析进行分析。

覆盖Free Time占比过大：Overlap Analysis中Free Time占比远超Computing和 Communication。理想的Free Time占比应在10%以内。

![](images/f33357473d80d989942a4f899e777d796acbe88341d0df0cd1ccbe2d4ecb9ab4.jpg)  
图 3-3 查看 Free

红框中的HostToDevice连线接近垂直，且蓝框中的Device上相对空闲。

![](images/cd0d6cb3d701787875bfb774fe5314f344a201743bfb551243d17bef86f41b2d.jpg)  
图 3-4 查看连线

HostToDevice频繁拷贝打断异步流水，造成下发瓶颈。

![](images/a4c730a4fe3aa43cdfdf42cc3cabef79ba8e3b18e420c66d98c1c1538c574052.jpg)  
图 3-5 寻找下发瓶颈

通信问题一般指NPU卡间通信存在异常，典型表现为快慢卡或通信带宽远不及预期，具体情况请参见MindStudio Insight的4.2.4.3 集群性能分析，解决方法请参考5.2 通信问题优化方案。

# 说明

在大集群场景中，全量集群的Profiling可能会数据量过大，不利于分析，建议使用4.2.3 模型调优快速分析（msprof-analyze命令行工具）中的cluster_analyze（集群分析）工具先对全量Profiling做一遍集群分析，将交付件cluster_analysis_output目录导入MindStudio Insight，观察是否存在明显快慢卡或通信传输问题，再挑选部分感兴趣卡的Profiling，从单卡维度分析。

图3-6为MindStudio Insight通信（Communication）界面中的“通信耗时分析”功能，其中每一个色块代表一个集合通信算子，其长度代表通信算子的执行时间。如果对于某个集合通信算子，不同卡间通信算子执行时间差距非常大，说明通信算子耗时最短的那张卡是慢卡（其他卡都在等此慢卡）。

![](images/8127b30ccf3c13f575a698c312ecf826c6774d6ae399386f93a7505f9d012f57.jpg)  
图 3-6 利用 MindStudio Insight 的通信耗时分析查找慢卡

在Timeline中可以看到明显的卡间等待，如图3-7中红框所示，快卡Rank6已经计算完毕，在等待慢卡Rank5完成计算。

![](images/14e2eb47b35c806c8d2e76e69bbc2828a45596918f6cbe48bc0febebddab495e.jpg)  
图 3-7 Timeline 对比快慢卡 Profiling（Ascend Hardware 层）

如图3-8所示，Communication泳道可观察到，快卡Rank6的hcom_allGather通信算子时长远大于慢卡Rank5的通信算子，且主要时长来源于同步等待。

![](images/fb68b4339330a8210c08e8c1369c3f1e2ca544819ad489a570fe7e6314299a6a.jpg)  
图 3-8 Timeline 对比快慢卡 Profiling（Communication 泳道）

# 计算问题

计算问题，即算子性能问题，是深度学习模型中的一个关键挑战，具体表现为部分基础计算单元的执行效率低下，从而影响整个模型的运行速度并造成资源浪费。这类问题需要借助专门的分析工具和代码优化技术来解决。例如，在评估融合算子性能时，可以通过对比不同配置下的计算时间、内存使用量等指标来进行综合判断。具体请参考5.3 算子性能问题优化方案来定位与解决。

![](images/4bb4295c6ec6e7a5b5e8fff82b7ff16acf68958bcd7b7518c881aaba4b8e34da.jpg)  
图 3-9 算子性能问题定位

# 性能工具的使用

性能工具总体介绍模型调优工具服务化工具

# 4.1 性能工具总体介绍

本章将系统介绍如何在训练和推理任务中高效利用调优工具链，实现从性能数据收集到问题定位的完整闭环流程。训练场景主要聚焦于模型调优，推理场景同时包括模型调优与服务化调优。本章节将重点围绕模型调优和服务化调优两个方面进行介绍。

表 4-1 性能工具简介  

<table><tr><td colspan="1" rowspan="1">调优维度</td><td colspan="1" rowspan="1">调优步骤</td><td colspan="1" rowspan="1">使用工具</td><td colspan="1" rowspan="1">详细说明</td></tr><tr><td colspan="1" rowspan="1">模型调优</td><td colspan="1" rowspan="1">性能数据采集</td><td colspan="1" rowspan="1">按照使能方式可以分为msprof命令行采集和Al框架Profiler接口采集两大类。具体介绍请参见4.2.2模型调优性能采集工具·Al框架Profiler接口采集-PyTorch场景:Ascend PyTorchProfilerMindSpore场景：MindSporeProfilermsprof命令行采集说明msprof命令行工具无AI框架层数据。</td><td colspan="1" rowspan="1">选择合适的性能数据采集工具，记录模型运行过程中必要的AI框架与昇腾软硬件数据。详细介绍请参见4.2.2模型调优性能采集工具。msprof命令行工具主要采集CANN层与NPU层性能数据，是其它采集接口的底座。Al框架Profiler接口封装了msprof命令行工具，进一步增加了对AI框架层性能数据的采集与解析，是训练或在线推理场景中最常用的采集方式。AI框架Profiler接口按照功能特性，可进一步分为常规采集（即静态采集）、动态采集、在线监测三种模式。此外，部分训练或推理套件会对AI框架Profiler接口进行进一步封装，可直接通过套件内接口调用，例如MindSpeed-MM、MindFormers等。</td></tr><tr><td colspan="2" rowspan="2">性能数据分析</td><td colspan="1" rowspan="1">模型调优快速分析工具：·集群分析（cluster_analyze）·专家建议（advisor）·性能拆解比对（compare）详细介绍请参见4.2.3模型调优快速分析（msprof-analyze命令行工具）</td><td colspan="1" rowspan="1">初步快速分析，可使用msprof-analyze提供的一系列功能，其中：·cluster_analyze:集群场景下，如干卡、万卡等无法直接分析全部数据的场景，可通过此工具来提取集群迭代耗时和通信数据，快速定位慢卡、慢节点以及慢链路问题。建议结合MindStudio Insight可视化工具的Summary（概览）与Communication（通信）页签使用，具体使用方法请参考MindStudio Insight的4.2.4.3集群性能分析章节。advisor：将常见问题的识别和优化建议工具化，快速定界、定位典型性能问题，或为下一步分析指明方向。compare：通过对训练耗时和内存占用的比对分析，定位到具体劣化的算子/API，帮助用户提升性能调优的效率；支持比较GPU与NPU之间、NPU与NPU之间的单卡性能差异，更推荐在GPU迁移NPU性能劣化、性能抖动等有基线比对数据的场景使用。</td></tr><tr><td colspan="1" rowspan="1">模型调优深入分析工具，具体分析请参见4.2.4模型调优深入分析（MindStudioInsight）。</td><td colspan="1" rowspan="1">MindStudio Insight可视化工具能够将以图形化方式展示完整的Profiling数据，助力用户深入理解和准确定位问题根源。该工具采用Top-Down分析方法，即从宏观到微观、从整个集群到单一节点逐步深入。若要详细了解其使用策略和具体操作，请参考4.2.4模型调优深入分析（MindStudio Insight ）。</td></tr><tr><td colspan="1" rowspan="1">服务化调优说明仅推理场景涉及服务化调优，具体服务化调优工具的使用请参见4.3服务化工具。</td><td colspan="1" rowspan="1">环境预检</td><td colspan="1" rowspan="1">预检工具（msprechecker)</td><td colspan="1" rowspan="1">属于广泛的性能调优范畴，来检查是否因系统、环境变量或配置文件等问题影响了服务的整体性能。</td></tr><tr><td rowspan="2"></td><td>快速 分析</td><td>·服务化专家建议工 具 （msservice_adviso r） 自动寻优工具 （modelevalstate）</td><td>服务化专家建议工具适用于快 速提升服务化性能的场景，但 不支持细粒度调优。 自动寻优工具适用于针对性提 升服务性能，能够实现接近手 工调优最佳性能的95%效果。</td></tr><tr><td>深入 分析</td><td>服务化调优工具 （msServiceProfiler）</td><td>进行深入分析，适合具备丰富服 务化运作经验的用户。</td></tr></table>

# 4.2 模型调优工具

# 4.2.1 使用步骤

模型调优可以分为性能采集和性能分析两个步骤，具体介绍如下：

1. 采集性能日志：建议先分析Profiling日志，工具使用方法请参见4.2.2 模型调优性能采集工具。

# 说明

首次采集性能数据时，建议仅采集L1、不开启堆栈（即with_stack为False），可以设置warmup为1，并设置active为2，采集两个step的数据。  
若涉及竞品分析，建议在相同条件下（包括但不限于采集的step数、超参数设置以及使用数据等）同步收集竞品的性能指标。

2. 分析性能瓶颈：收集到Profiling日志后，可使用性能分析工具进行瓶颈分析。

a. 使用msprof-analyze工具初步分析，粗粒度定位性能问题，并为后续的深入分析提供明确的方向，具体请参见4.2.3 模型调优快速分析（msprof-analyze命令行工具）。  
b. 通过MindStudio Insight工具进一步识别瓶颈点，深入剖析问题根源。具体请参见4.2.4 模型调优深入分析（MindStudio Insight）。

# 4.2.2 模型调优性能采集工具

MindStudio提供多种灵活的系统级性能数据采集方式，可根据实际需求选择合适的方案，以精准定位性能瓶颈并提升训练效率。

按照使能方式可以分为命令行采集（msprof）和AI框架Profiler接口采集（AscendPyTorch Profiler，MindSpore Profiler）两大类。

msprof命令行工具主要采集CANN层与NPU层性能数据，是其它采集接口的底座。

# 说明

msprof命令行工具无AI框架层数据。

AI框架Profiler接口封装了msprof命令行工具，进一步增加了对AI框架层性能数据的采集与解析，是最常用的采集方式。AI框架Profiler接口按照功能特性，可进一步分为常规采集（即静态采集）、动态采集、在线监测三种模式。

此外，部分训练或推理套件会对AI框架Profiler接口进行进一步封装，可直接通过套件内接口调用，例如MindSpeed-MM、MindFormers等。

![](images/ea113f47363a31f6f04ace7099ba677832f8f5a9449a94a1d2c1865e559518bf.jpg)  
图 4-1 性能采集框架

表 4-2 采集方式说明  

<table><tr><td rowspan=1 colspan=1>采集方式</td><td rowspan=1 colspan=1>特点</td><td rowspan=1 colspan=1>推荐使用场景</td><td rowspan=1 colspan=1>文档参考链接</td></tr><tr><td rowspan=1 colspan=1>msprof命令行采集</td><td rowspan=1 colspan=1>msprof命令行工具提供了AI任务运行性能数据、昇腾AI处理器系统数据等性能数据的采集和解析能力。说明msprof命令行工具无AI框架层数据。</td><td rowspan=1 colspan=1>训练、推理场景下均可使用。</td><td rowspan=1 colspan=1>《性能调优工具用户指南》中的“msprof模型调优工具&gt;性能数据采集&gt;msprof采集通用命令”章节。</td></tr><tr><td rowspan=1 colspan=1>Ascend PyTorchProfiler接口采集</td><td rowspan=1 colspan=1>完全对标PyTorch-GPU场景下的使用方式，支持采集PyTorch框架和昇腾软硬件数据。</td><td rowspan=1 colspan=1>基于PyTorch的常规性能分析。</td><td rowspan=1 colspan=1>《性能调优工具用户指南》中的“ Ascend PyTorch调优工具”章节。</td></tr><tr><td rowspan=1 colspan=1>MindSporeProfiler接口采集</td><td rowspan=1 colspan=1>支持采集MindSpore框架和昇腾软硬件数据。</td><td rowspan=1 colspan=1>基于MindSpore的常规性能分析。</td><td rowspan=1 colspan=1>《性能调优工具用户指南》中的MindSpore调优工具章节。</td></tr></table>

使用AI框架Profiler采集时，可以参考表4-3信息配置参数。

表 4-3 参数配置  

<table><tr><td>使用场景</td><td>配置参数</td></tr><tr><td rowspan="4">常规性能分析场景</td><td>·profiler_level=Level1</td></tr><tr><td>aic_metrics保持默认，取值PipeUtilization</td></tr><tr><td>activities设置采集CPU和NPU .</td></tr><tr><td>其他开关根据需要开启</td></tr><tr><td colspan="1" rowspan="1">NPU/GPU对比场景</td><td colspan="1" rowspan="1">该配置用于对比NPU和GPU端到端耗时：•profiler_level=Level0·activities设置仅采集NPU或CPU和NPU（根据需要）·其他开关关闭</td></tr><tr><td colspan="1" rowspan="1">定位代码位置场景</td><td colspan="1" rowspan="1">如果需要定位异常算子代码位置，可在常规场景下开启with_stack或with_modules开关，用以开启调用栈（非必要不开启，性能会膨胀）。</td></tr><tr><td colspan="1" rowspan="1">分析算子NPU片上内存申请情况</td><td colspan="1" rowspan="1">profile_memory=True</td></tr><tr><td colspan="1" rowspan="1">分析集群通信情况</td><td colspan="1" rowspan="1">profiler_level=Level1</td></tr></table>

按照功能特性区分，可以区分为常规采集、动态采集、轻量化性能数据在线监测三大类，如表4-4所示。

表 4-4 采集分类  

<table><tr><td rowspan=1 colspan=1>采集方式</td><td rowspan=1 colspan=1>特点</td><td rowspan=1 colspan=1>推荐使用场景</td><td rowspan=1 colspan=1>使用指导</td></tr><tr><td rowspan=1 colspan=1>常规采集</td><td rowspan=1 colspan=1>预先设置采集周期或全量采集，落盘详细性能数据。</td><td rowspan=1 colspan=1>常规性能分析</td><td rowspan=2 colspan=1>•Ascend PyTorch Profiler:请参考《性能调优工具用户指南》中的“AscendPyTorch调优工具”章节。MindSpore Profiler:请参考《性能调优工具用户指南》中的MindSpore调优工具章节。msprof：请参考《性能调优工具用户指南》中的“msprof模型调优工具&gt;性能数据采集&gt;msprof采集通用命令”章节。</td></tr><tr><td rowspan=1 colspan=1>dynamic_profile动态采集</td><td rowspan=1 colspan=1>在执行模型训练过程中可以随时开启采集进程、动态修改配置采集项，无需频繁改动脚本代码。</td><td rowspan=1 colspan=1>启停成本高的场景（如超大规模训练）</td></tr></table>

# 4.2.3 模型调优快速分析（msprof-analyze 命令行工具）

针对AI作业中的性能瓶颈，msprof-analyze工具提供快速分析的命令行工具，包含三个核心能力，如表4-5所示。

表 4-5 msprof-analyze 工具的核心能力  

<table><tr><td rowspan=1 colspan=1>工具名称</td><td rowspan=1 colspan=1>功能介绍</td></tr><tr><td rowspan=1 colspan=1>cluster_analyze（集群分析）</td><td rowspan=1 colspan=1>提供慢节点、慢卡、慢链路定位的能力，可结合MindStudio Insight可视化工具使用。</td></tr><tr><td rowspan=1 colspan=1>compare（性能拆解比对）</td><td rowspan=1 colspan=1>提供NPU与GPU以及两个NPU之间，算子在时间和内存维度上的比对能力，帮助用户快速定位问题算子。</td></tr><tr><td rowspan=1 colspan=1>advisor（专家建议）</td><td rowspan=1 colspan=1>结合性能调优专家经验和昇腾软硬件对算子的亲和适配，提供自动化调优能力，帮助用户识别性能瓶颈，并给出优化建议。</td></tr></table>

cluster_analyze（集群分析）

cluster_analyze（集群分析）结果通过MindStudio Insight可视化工具展示，辅助进行通信矩阵与通信耗时分析。

![](images/6cce54673113da280969c7ff45f40bbe70e73d77e86f0dc1306ab9ce0f470f3a.jpg)  
图 4-2 利用 MindStudio Insight 可视化集群分析结果示意图

compare性能比对

compare工具将耗时拆解为算子执行、通信（非计算掩盖部分）、调度开销、内存占用四大核心维度，帮助精准定位性能瓶颈。

图 4-3 compare 工具分析结果报告示意图  

<table><tr><td colspan="9">BaseProfiing:/home/data_space/ubuntu_1378765_202502qComparisonProfiling:/home/data_space/ubuntu_13787582</td></tr><tr><td></td><td></td><td>DurationRatio</td><td></td><td>Duration(ms)</td><td>DurationRatio</td><td>Number</td><td>Diff Duration(ms)</td><td>Diff Ratio</td></tr><tr><td>Index Computing Time</td><td>Duration(ms) 1,520.75</td><td>44.90%</td><td>Number 20,836</td><td>1,511.48</td><td>44.64%</td><td>20,986</td><td>-9.27</td><td>99.39%</td></tr><tr><td>Flash Attention (Forward)</td><td>347.89</td><td>10.27%</td><td>2.048</td><td>347.74</td><td>10.27%</td><td>2.048</td><td>-0.14</td><td>99.96%</td></tr><tr><td></td><td>317.02</td><td>9.36%</td><td>256</td><td>316.84</td><td>9.36%</td><td>256</td><td>0.18</td><td>99.94%</td></tr><tr><td>FlashAttention (Forward) (Cube)</td><td>30.86</td><td>0.91%</td><td>1792</td><td>30.90</td><td>0.91%</td><td>1.792</td><td>0.04</td><td></td></tr><tr><td>FlashAttention (Forward) (Vector)</td><td></td><td></td><td></td><td></td><td></td><td>256</td><td></td><td>100.12%</td></tr><tr><td>FlashAttention (Backward)</td><td>679.30</td><td>20.06%</td><td>256</td><td>680.71</td><td>20.11%</td><td></td><td>1.41</td><td>100.21%</td></tr><tr><td>Flash Attention (Backward) (Cube)</td><td>679.30</td><td>20.06%</td><td>256</td><td>680.71</td><td>20.11%</td><td>256</td><td>1.41</td><td>100.21%</td></tr><tr><td>Matmul</td><td>253.32</td><td>7.48%</td><td>3.072</td><td>249.51</td><td>7.37%</td><td>3,072</td><td>-3.80</td><td>98.50%</td></tr><tr><td>Matmul (Cube)</td><td>253.32</td><td>7.48%</td><td>3.072</td><td>249.51</td><td>7.37%</td><td>3.072</td><td>-3.80</td><td>98.50%</td></tr><tr><td>Vector</td><td>239.36</td><td>7.07%</td><td>15,360</td><td>232.57</td><td>6.87%</td><td>15,510</td><td>-6.79</td><td>97.16%</td></tr><tr><td>Vector (Trans)</td><td>19.81</td><td>0.58%</td><td>3.528</td><td>20.33</td><td>0.60%</td><td>3,528</td><td>0.52</td><td>102.63%</td></tr><tr><td>Vector (No Trans)</td><td>219.56</td><td>6.48%</td><td>11,832</td><td>212.24</td><td>6.27%</td><td>11,982</td><td>-7.31</td><td>96.67%</td></tr><tr><td>SDMA (Tensor Move)</td><td>0.89</td><td>0.03%</td><td>100</td><td>0.94</td><td>0.03%</td><td>100</td><td>0.05</td><td>106.07%</td></tr><tr><td>Uncovered Communication Time</td><td>1.798.60</td><td>53.10%</td><td></td><td>1,706.90</td><td>50.42%</td><td></td><td>-91.70</td><td>94.90%</td></tr><tr><td>tp: Communication</td><td>976.61</td><td>28.83%</td><td></td><td>606.84</td><td>17.92%</td><td></td><td>-369.76</td><td>62.14%</td></tr><tr><td>Wait</td><td>19.10</td><td>0.56%</td><td></td><td>0.52</td><td>0.02%</td><td></td><td>-18.58</td><td>2.70%</td></tr><tr><td>Transmit</td><td>957.51</td><td>28.27%</td><td></td><td>606.33</td><td>17.91%</td><td></td><td>-351.18</td><td>63.32%</td></tr><tr><td>dp_cp:Group10.174.216.241%enp189s0f1_510.10</td><td></td><td>0.30%</td><td></td><td>0.00</td><td></td><td></td><td>-10.10</td><td>0.00%</td></tr><tr><td>Transmit</td><td>10.10</td><td>0.30%</td><td></td><td>0.00</td><td>1000%</td><td></td><td>-10.10</td><td>0.00%</td></tr><tr><td>pp:Group10.174.216.241%enp189s0f1_5500881.93</td><td></td><td>26.04%</td><td></td><td>0.00</td><td>0.00%</td><td></td><td>-881.93</td><td>0.00%</td></tr><tr><td>Wait</td><td>1.379.84</td><td>40.74%</td><td></td><td>0.00</td><td>0.00%</td><td></td><td>1.379.84</td><td>0.00%</td></tr><tr><td>Transmit</td><td>497.91</td><td>-14.70%</td><td></td><td>0.00</td><td>0.00%</td><td></td><td>497.91</td><td>0.00%</td></tr><tr><td>default_group:Group10.174.216.241%enp1891,456.76</td><td></td><td>43.01%</td><td></td><td>1,641.48</td><td>48.48%</td><td></td><td>184.72</td><td>112.68%</td></tr><tr><td>Wait</td><td>0.00</td><td>0.00%</td><td></td><td>0.10</td><td>0.00%</td><td></td><td>0.10</td><td>119.05%</td></tr><tr><td>Transmit</td><td>1,456.76</td><td>43.01%</td><td></td><td>1641.38</td><td>48.48%</td><td></td><td>184.62</td><td>112.67%</td></tr><tr><td>embd:Group10.174.216.241%enp189s0f1_555.62</td><td></td><td>0.17%</td><td></td><td>0.00</td><td>0.00%</td><td></td><td>-5.62</td><td>0.00%</td></tr><tr><td>Transmit</td><td>5.62</td><td>0.17%</td><td></td><td>0.00</td><td>0.00%</td><td></td><td>-5.62</td><td>0.00%</td></tr><tr><td>mp:Group 10.174.216.241%enp189s0f1_55005.86</td><td></td><td>0.17%</td><td></td><td>0.36</td><td>0.01%</td><td></td><td>-5.50</td><td>6.17%</td></tr><tr><td>Wait</td><td>5.81</td><td>0.17%</td><td></td><td>0.02</td><td>0.00%</td><td></td><td>-5.79</td><td>0.42%</td></tr><tr><td>Transmit</td><td>0.05</td><td>0.00%</td><td></td><td>0.34</td><td>0.01%</td><td></td><td>0.29</td><td>681.27%</td></tr><tr><td>Group10.174.216.241%enp189s0f1_55000_00.00</td><td></td><td>0.00%</td><td></td><td>940.65</td><td>27.78%</td><td></td><td>940.65</td><td>INF</td></tr><tr><td>Wait</td><td>0.00</td><td>0.00%</td><td></td><td>1390.74</td><td>41.08%</td><td></td><td>1.390.74</td><td>INF</td></tr><tr><td>Transmit</td><td>0.00</td><td>0.00%</td><td></td><td>-450.09</td><td>-13.29%</td><td></td><td>-450.09</td><td>INF</td></tr><tr><td>Group10.174.216.241%enp189s0f1_550000.00</td><td></td><td>0.00%</td><td></td><td>10.17</td><td>0.30%</td><td></td><td>10.17</td><td>INF</td></tr><tr><td>Transmit</td><td>0.00</td><td>0.00%</td><td></td><td>10.17</td><td>0.30%</td><td></td><td>10.17</td><td>INF</td></tr><tr><td>Group10.174.216.241%enp189s0f1_55000_00.00</td><td></td><td>0.00%</td><td></td><td>5.63</td><td>0.17%</td><td></td><td>5.63</td><td>INF</td></tr><tr><td>Transmit</td><td>0.00</td><td>0.00%</td><td></td><td>5.63</td><td>0.17%</td><td></td><td>5.63</td><td>INF</td></tr><tr><td>Free Time</td><td>67.64</td><td>2.00%</td><td></td><td>167.31</td><td>4.94%</td><td></td><td>99.67</td><td>247.37%</td></tr><tr><td></td><td>67.64</td><td>2.00%</td><td></td><td>167.31</td><td>4.94%</td><td></td><td>99.67</td><td>247.37%</td></tr><tr><td>Free E2ETime</td><td>3.386.99</td><td>100.00%</td><td></td><td>3.385.69</td><td>100.00%</td><td></td><td>-1.30</td><td>99.96%</td></tr></table>

# advisor专家建议

advisor专家建议工具自动化识别性能瓶颈点，并给出优化建议。覆盖集群和单卡场景的下发、计算、通信等维度，端到端帮助用户分析Profiling数据。

![](images/ecfc65675ae2e2b069ae95a4cc060356df2ec2a9bd6bcb0881443e8f9e3caf05.jpg)  
图 4-4 advisor 专家建议工具主要功能

advisor工具将优化建议按紧急程度分级，其中红色标注代表最高优先级，需优先处理。

![](images/e3e5b552387f622262d76f4ba95ae6b436820d24d1721cabb538f459524e26bd.jpg)  
图 4-5 advisor 专家建议工具分析结果报告示意图

# 4.2.4 模型调优深入分析（MindStudio Insight）

# 4.2.4.1 总体介绍

MindStudio Insight工具能够将全量Profiling数据进行可视化呈现，帮助用户进一步分析并确认问题。

使用MindStudio Insight工具分析问题的流程如图4-6所示。

![](images/41f191360d5e940d926e122c53ffcf1a26ace19fcecf5567d0a6b1ae34ee4c59.jpg)  
图 4-6 使用 MindStudio Insight 工具分析流程图

# 1. 使用集群分析功能初步定界

a. 先进入概览（Summary）界面，通过多卡计算、通信、调度比对，初步确认问题分类。具体定界过程，详见4.2.4.3.1 概览（Summary） 。  
b. 再进入通信（Communication）界面，按通信域拆解，进一步定位慢卡或慢链路问题；确认异常卡或异常链路后，可直接根据通信算子跳转至时间线（Timeline）具体定位，详见4.2.4.3.2 通信（Communication）

# 说明

● 卡数较少时，可直接导入原始性能数据，自动生成集群分析结果（由可视化工具调用msprof-analyze命令行工具）。  
● 卡数过多、全量性能数据过于笨重时，推荐您以命令行方式手动调用msprof-analyze工具，用MindStudio Insight打开cluster_analysis_output集群分析结果交付件，更加轻便快捷。

2. 初步定界后，选择所需卡，从单卡维度进一步分析

时间线（Timeline）界面：将训练或推理过程中Host、Device上的运行详细情况平铺在时间轴上，直观呈现Host侧的API耗时以及Device侧的Task耗时，具体使用思路详见4.2.4.4.1 时间线（Timeline） 。  
内存（Memory）界面：以内存折线图呈现整体内存趋势，可以框选放大折线图中峰值区域，精准定位到内存消耗大的进程或算子，具体使用思路详见4.2.4.4.2 内存（Memory）   
算子（Operator）界面：计算算子和通信算子的耗时统计，可按类型、Shape统计，同时支持两卡间比对功能，可更直观的查看算子详情，具体使用思路详见4.2.4.4.3 算子（Operator）

# 4.2.4.2 获取软件包

MindStudio Insight工具的软件包及资料获取方式如表4-6所示，请根据需求选择合适的版本下载。

表 4-6 软件包及资料获取方式  

<table><tr><td>类别</td><td>获取链接</td></tr><tr><td>软件包</td><td>·社区版： 获取链接 商发版：获取链接</td></tr><tr><td></td><td>. POC版本（非商用版本，不定期发布，可抢先体验最新功 能）：获取链接</td></tr><tr><td>资料</td><td>获取链接</td></tr></table>

# 4.2.4.3 集群性能分析

# 4.2.4.3.1 概览（Summary）

概览（Summary）界面常用功能有并行策略分析，流水并行分析，多卡计算、通信、调度比对，MoE大模型专家负载均衡分析等。

# 初步定性

首先通过多卡计算、通信、调度时间比对，确认是否有哪一部分占比过高，或者存在严重卡间不同步、各卡间通信时间波动较大现象（快慢卡问题），初步定性问题。概览界面展示如图4-7所示。

![](images/a3ef7a646448958bd6fbe902c8758bc9afc254d749a1f6ea327fdfc2fb2d4c41.jpg)  
图 4-7 概览界面

常用操作步骤如下：

1. 配置正确的并行策略，即保证并行策略参数值与模型实际训练/推理时的并行参数配置一致。具体的并行参数，可与模型开发人员确认。  
2. 卡数较少时，推荐选择全展开维度，即 $\mathsf { \Omega } ^ { 4 6 } \mathsf { D } \mathsf { P } + \mathsf { P } \mathsf { P } + \mathsf { T } \mathsf { P } ^ { 3 5 }$ 维度。  
3. 可选择感兴趣的性能指标进行热力图渲染，快速横向比对性能指标。快慢卡分析时，一般关注特定并行域下的通信时间。  
4. 查看并行策略排布图，可通过热力渲染效果快速横向比对  
5. 并行策略正确配置时，可得到慢卡专家建议分析。

6. 在下方的”计算/通信概览”页面，查看各卡计算、通信、调度耗时对比，初步确认是否存在计算、通信、下发问题和快慢卡问题。

# 典型案例

典型Case1：如图4-8所示，可以看到各卡间通信时间波动较大，存在严重卡间不同步，且计算、空闲（即下发）与通信所占比例成反比。通信时间占比低，计算、空闲（即下发）时间占比高的卡即为慢卡。可以初步确认此集群存在快慢卡问题。进一步定位快慢卡问题，可参考5.2.2.3 快慢卡定位Timeline操作案例。

![](images/84e228e4a20194dea54cedb69a8d52909c4b7e9899a9c6fda6c24a8004497d89.jpg)  
图 4-8 典型 Case1

典型Case2：如图4-9所示，可以看到空闲时间占比较高，说明此集群存在较高的下发瓶颈，可参考5.4 Host Bound问题定位及解决方法进一步定位优化；通信时间占比也较高，且各卡通信时间有波动，可参考5.2 通信问题优化方案进一步定位优化。

![](images/9ac369d05e5f027aee09ebd13d948945416f828e14dda5404566549d83a2894a.jpg)  
图 4-9 典型 Case2

卡数较多时，呈现的全量数据过多，不利于查看分析，如图4-10所示。需要通过合理的方式，精简拆分数据，让分析方向更加明确。

![](images/81a8d584b44bcf75b65fd54da401ee2f6157c2aa50474702ec48aecf7deb56fd.jpg)  
图 4-10 ”计算/通信概览”页面全量数据展示

精简方式1：左键点击通信域连线（例如下图中 $\textcircled{1}$ ），单独查看某个通信域，得到按通信域拆解后的概览图。点击框也有类似效果（同一根连线代表同一个通信域、同一个框内代表并行分组）。

![](images/41e7c47442c295b7ac192e0787e52b9728261516adc81693377ae9806397af0e.jpg)  
图 4-11 按通信域拆解后的计算/通信概览  
计算/通信概览③

![](images/c04841a3c4ec1b57b29d75a1c25f4dbcc1de3424d8bdf5eb7605f8974a9ce1e2.jpg)  
专家建议：Communicatiohassomeissues,because themaxdiferenceofCommunication(NotOverlapped)"hasreached379600us

精简方式2：先查看折叠视图，由整体至局部逐渐定位。以一个并行策略为DP8、PP8、TP8的512卡集群为例，其全展开维度（即“DP + PP+ TP”维度）有512张卡，折叠TP维度（即“DP + PP”维度）后，每8张TP域卡合并为1个节点，即折叠为64个节点。可先选择“DP + PP”维度，初步定位慢分组，再进一步进入“DP + PP+ TP”维度定位慢卡。

a. 在“DP + PP”维度下，“性能指标”选择“DP-平均通信时间”，观察到dpIndex=4、7存在慢分组，如图4-12所示。

![](images/572915c4c040e4a61fb757203b19c32b7b419ccf162fcbf52cc0f567c500913e.jpg)  
图 4-12 DP+PP 维度（TP 被折叠）

b. 以dpIndex=4这一并行分组为例，在4上单击鼠标右键，选择“展开”，跳转至“DP + PP + TP”维度下，如图4-13所示。此时，可将“性能指标”切换至“TP-通信时间”，看到慢卡为38卡，即38卡影响了TP域（32-39），进而影响了图4-12中的dpIndex0-dpIndex7。

![](images/dfe2fcb69f7b0808e7bdedc0b5c7a262fe839f945c4dbb7790a362f15573e348.jpg)  
图 4-13 DP+PP+TP 维度（全展开维度）

c. 定位到慢卡后，可右键点击感兴趣的通信域连线（如此处的绿色连线，代表TP通信域），查看通信耗时分析，前往通信（Communication）界面进一步分析慢卡的通信过程，如图4-14和图4-15所示。

![](images/a64cf6e9c8d9706cab31eea97713d9b9615751429061e66ea4e2adda24e63489.jpg)  
图 4-14 右键连线，查看通信耗时分析

![](images/be18194b101efb36854dc639ea25988d937fb10c8f515bf011a20d695175b683.jpg)  
图 4-15 通信（Communication）界面-通信耗时分析

# 4.2.4.3.2 通信（Communication）

通信（Communication）页签将通信指标按通信域拆解。如果在概览（Summary）界面中显示通信时间过长，可在通信（Communication）界面进一步确认是否存在慢卡或慢链路问题。可在图4-16红框所示位置切换通信矩阵视图或通信耗时分析视图。

![](images/8bdb29b6cf05c0883bead0336f8e576a7791551065ea6fb328e996de5a2a13c5.jpg)  
图 4-16 切换通信矩阵或通信耗时分析视图

# 操作步骤

步骤1 在通信（Communication）界面，选择“通信耗时分析”，查看“通信时长”图，确认传输时间在通信时间中的占比是否过高，如图4-17所示。

其中传输时间过长为慢链路问题，同步时间过长为慢卡问题。

![](images/3661f0354ab9b1ba5773fb3bd97a859b1b8290d3f4cdd547a6705fa27e1cb19f.jpg)  
图4-17 通信时长

步骤2 如果传输时间占比过高，再选择“通信矩阵”，查看通信矩阵图，分析传输带宽是否远低于经验带宽，如图4-18所示。在传输数据量足够的前提下，如果带宽明显低于预期带宽, 可以认为存在优化空间。常见慢链路原因包括通信重传、通信小包、数据包字节未对齐等，具体可参考5.2 通信问题优化方案中相关案例。

![](images/43002b314d009a039532126484f488c3c59c0c1ad2a52efb259452a06738f2b1.jpg)  
图 4-18 通信矩阵

步骤3 如果传输时间占比低，等待或同步时间占比高，则为“快慢卡问题”，需选择“通信耗时分析”。通过通信算子横向平铺，查看“通信算子缩略图”锁定慢卡，如图4-19所示。针对绿色的hcom allGather集合通信算子，时长较短的4卡、5卡、13卡为慢卡，而时长较长的卡（例如11卡、14卡等）为相对的快卡。接下来需要分析慢卡在空白时间做什么？需前往时间线（Timeline）界面，查看具体差异点，详细可参考5.2.2.3 快慢卡定位Timeline操作案例。

![](images/cd9836d033137d941f82653b25ecf2701b57f53704d2ea2c458c98612f971477.jpg)  
图4-19 锁定慢卡

# ----结束

# 通信与时间线相互跳转

支持通信（Communication）界面与时间线（Timeline）界面，根据通信算子互相跳转，如图4-20和图4-21所示。

若在通信界面定位到异常卡与异常通信算子，可跳转至时间线视图，进一步确认问题根因；若在时间线界面观察到耗时异常长的通信算子，也可跳转至通信界面，寻找是否有同一通信域内的慢卡拖累了此卡，导致长时间的等待耗时。

![](images/e946bd393e1351eef24e98d149c627d09939690c12c63e6e8e97c2802ec104e9.jpg)  
图 4-20 通信界面算子跳转至时间线界面

![](images/b72907c277abfa94be53ac5ba37653520c4924ff6021df54c0a1c49c7f219f22.jpg)  
图 4-21 时间线界面通信算子跳转至通信界面

# 4.2.4.4 单卡性能分析

# 4.2.4.4.1 时间线（Timeline）

时间线（Timeline）是将训练或推理过程中的Host、Device上的运行详细情况平铺在时间轴上，直观呈现Host侧的API耗时以及Device侧的Task耗时。常用泳道与界面如图4-22所示，界面信息说明如表4-7所示。

![](images/53c8c7131828194333228c75f383611493f5c03d160595c2e5f14a94e9bc2591.jpg)  
图 4-22 时间线常用泳道与界面

表 4-7 时间线常用泳道与界面信息说明  

<table><tr><td colspan="1" rowspan="1">序号</td><td colspan="1" rowspan="1">名称</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">1</td><td colspan="1" rowspan="1">Python泳道（一级流水）</td><td colspan="1" rowspan="1">查看Python层代码，采集时开启withstack开关可查看代码调用栈。</td></tr><tr><td colspan="1" rowspan="1">2</td><td colspan="1" rowspan="1">CANN泳道（二级流水）</td><td colspan="1" rowspan="1">收集ACL接口执行、GE融合、Runtime等数据。Python侧算子从一级流水下发至此二级流水，任务从二级流水出队后被下发至NPU层。</td></tr><tr><td colspan="1" rowspan="1">3</td><td colspan="1" rowspan="1">Ascend Hardware（NPU层）</td><td colspan="1" rowspan="1">也称Device侧，记录发生在NPU上计算、通信等任务的执行时序。</td></tr><tr><td colspan="1" rowspan="1">4</td><td colspan="1" rowspan="1">Al Core Freq （AlCore频率）</td><td colspan="1" rowspan="1">AI Core频率，可用于观察降频问题。</td></tr><tr><td colspan="1" rowspan="1">5</td><td colspan="1" rowspan="1">Communication（通信）</td><td colspan="1" rowspan="1">旧称HCCL泳道。记录NPU层通信事件，与AscendHardware的通信子泳道一一对应，此处由HCCL等组件上报。定位通信细节时可查看此泳道。</td></tr><tr><td colspan="1" rowspan="1">6</td><td colspan="1" rowspan="1">Overlap Analysis（覆盖分析）</td><td colspan="1" rowspan="1">将Ascend Hardware（NPU层）的计算、通信任务垂直投影至此，得到计算、通信、空闲时间的拆分。常用于快速比对不同卡间计算、通信、空闲差异来源。</td></tr><tr><td colspan="1" rowspan="1">7</td><td colspan="1" rowspan="1">Stats System View（统计视图）</td><td colspan="1" rowspan="1">单卡维度统计汇总信息，可通过左侧“卡序号”下拉框切换不同卡。</td></tr></table>

此处列举了定位过程中Timeline最常用的泳道与界面，每条泳道可展开查看具体细节，如图4-23所示。完整界面介绍请参见《MindStudio Insight工具用户指南》的“系统调优 $>$ 时间线（Timeline）”章节。

![](images/87c5823b55a67015b57a00675828064c59e0b0f24cd404a9f00c252519187bea.jpg)  
图 4-23 泳道展开查看具体细节

# 常用操作

想要快速查看或了解当前所有快捷键操作，可单击界面右上角的问号按钮，在下拉菜单中选择“键盘快捷键”，即可打开快捷键说明的弹窗。

Timeline常用操作包括通信与时间线相互跳转、置顶比对、覆盖分析、旗帜标记重点区域、框选统计、下发连线关系查看等功能。详情请参见《MindStudio Insight工具用户指南》的“系统调优 > 时间线（Timeline）”章节。

# 定位快慢卡具体差异来源

时间线（Timeline）常用于进一步定位快慢卡具体的差异来源。理想情况下，每张卡的计算用时相对接近，不应存在某张卡提前完成计算，长时间等待另一张卡的情况。当出现某些卡存在时长较长的通信算子，且通信算子主要时长来源于等待（例如Notify Wait事件）时，优先考虑是否出现了快慢卡问题。

快慢卡是一个现象，背后原因多种多样，需要通过比对快慢卡在时间线上的差异，确认具体原因。慢卡常见原因包括负载不均衡、计算慢、下发慢、数据加载慢（存储问题）。具体定位过程如下：

步骤1 在通信（Communication）界面的通信算子缩略图中，查看差异较大的通信算子，并跳转至时间线界面。

步骤2 通过覆盖分析、置顶比对，确认Ascend Hardware层（NPU层）差异来源。

步骤3 在时间线（Timeline）界面选择async_npu下发连线，通过连线关系，由NPU层向上寻找，确认Python层差异来源。确认差异来源于某处Python层代码后，可凭借此信息，与模型开发或运维人员进一步确认问题根因。

# 说明

时间线（Timeline）界面功能比较多，可以通过一个实际案例来具体感受下各种功能的作用，详细案例请参见5.2.2.3 快慢卡定位Timeline操作案例。

# ----结束

# 观察下发瓶颈

时间线（Timeline）是观察下发问题的有力工具，理想情况下NPU侧的计算流水线能不停运转，不会出现NPU等CPU的场景。一旦下发慢，将导致流水线无法运转，AICore算力利用率降低。

# 说明

理想Free Time占比约为10%以内。

下发瓶颈在时间线（Timeline）典型表现分别如下所示。具体定位方法论与优化思路请参考5.4.5 下发异常分析。

覆盖分析Free Time占比远超Computing和Communication，如图4-24和图4-25所示。

![](images/3a8a94e47562c14aa9a46d2d4e493b1b3bf7dbc22afe2a97fe4ec5178ab3c6bb.jpg)  
图 4-24 下发瓶颈典型表现 1

![](images/69c4c3471ffa4e267d500107d9aeb0ffe8539721ceb09349a6fd50a2d3e4ec27.jpg)  
图 4-25 下发瓶颈典型表现 2

HostToDevice连线接近垂直，如图4-26所示。

![](images/047a9f5c97fe32969c6d8728b01dfdfda0a098dabdd476a462d9b19fa3e25f2f.jpg)  
图 4-26 下发瓶颈典型表现 3

频繁HostToDevice拷贝打断异步流水，造成下发瓶颈，如图4-27所示。

![](images/4c832ef9bf355b0226f41a6a6a70922616e00430bf0aac302b2c1b41496d5352.jpg)  
图 4-27 下发瓶颈典型表现 4

# 单卡维度统计与搜索算子

如果想在时间线（Timeline）中查看算子具体位置，可在底部数据窗格中选择“系统视图”，选择“统计系统视图”和相应“卡序号”，单击“算子详情”，可查看所有

算子，并可按照名称、类型、加速器核、输入/输出Shapes过滤，按耗时排序，选择相应算子，单击“点击跳转Timeline”列的“点击”，可跳转至时间线的具体位置，如图4-28所示。此操作比全局搜索定位更快。

![](images/ed08ce55e7ac4ada5c4622fbabb126b11d71f038dfb5d3c1b3165a21585ccc8c.jpg)

# 4.2.4.4.2 内存（Memory）

以内存折线图呈现整体内存趋势，可以框选放大折线图中峰值区域，精准定位到内存消耗大的进程或算子。针对内存申请、释放异常的算子，跳转至时间线（Timeline），定位至具体代码。

# 说明

内存优化思路：尽量增大Batchsize，最大化利用NPU内存。观察内存趋势，消除尖刺，削峰填谷。

查看图4-29，可以看到NPU利用率不足，观察到存在内存尖刺。

![](images/9648c81bccd06facc7ba84b7bb81bd4ddd365c254513c703a5e1bd5647d65ed3.jpg)  
图 4-29 典型 Case

通过内存尖刺的时间点，框选锁定尖刺时间区域内算子。在内存申请/释放详情中，按内存申请大小降序排列，根据内存申请排名第一的算子跳转至时间线，定位至具体代码。如图4-30所示。随后，根据代码位置，和模型开发人员沟通确认有无优化空间。

![](images/36bce741bca26552b460a8352bdd89d35109bc7a38b00abf904a70df82e14fa8.jpg)  
图 4-30 跳转至时间线

内存（Memory）界面还支持两卡间的比对功能，详情请参见《MindStudio Insight工具用户指南》的“系统调优 $>$ 内存（Memory） > 使用说明”章节。

# 4.2.4.4.3 算子（Operator）

算子（Operator）界面展示计算算子和通信算子耗时统计，常用功能如下：

按类型统计，用于观察Top耗时算子占比，尤其是转换类等低效算子是否占比过高。● 按加速器核分组统计，观察AI CPU类算子与vector类算子是否耗时占比过高。● 按输入Shape统计计算算子，观察算子是否会在特定Shape下劣化。● 可切换展示耗时Top15或全部算子，如图4-31所示。

具体定位与优化方法，可参考5.3 算子性能问题优化方案。

![](images/f837a84dc2bb487195d806a6738eabe1d863241718acfe244a373abb99695421.jpg)  
图 4-31 算子界面

算子（Operator）界面还支持两卡间的比对功能，详情请参见《MindStudio Insight工具用户指南》的“系统调优 $>$ 内存（Memory） $>$ 使用说明”章节。

# 典型案例：利用算子比对功能快速定位计算性能劣化原因

问题背景：相同模型在不同机器上部署，计算性能劣化（每step约80ms），需定位原因。

步骤1 将两卡的性能数据移动至同一父目录，使用MindStudio Insight打开该父目录。

步骤2 参见《MindStudio Insight工具用户指南》的“系统调优 $>$ 算子（Operator） > 使用说明”章节设置两卡间数据比对。

步骤3 已知慢卡相比快卡单Step计算耗时多出约80ms。开启比对模式后，在“算子详情”中，按照“总耗时”进行排序，如图4-32所示。观察可发现，计算时间差异主要来源是Matmul算子，在数量一致的前提下（数量差为0），总耗时相差约74ms，为主要耗时差异来源。

![](images/e5218c30b01c753f68716623825ce9f0aa2a3090f49bd196f14725000ee1b18b.jpg)  
图 4-32 算子卡间比对

步骤4 进一步按相同Shape对比，可以看到，相同类型（MatmulV3）不同Shape算子有不同程度劣化，各个Shape下，慢卡Matmul稳定劣化于快卡，如图4-33所示。

![](images/b1fe8c49d9e8473ad6db4eaedebb6078cc76ef0aad0f2501fb4f037ec9b94ca8.jpg)  
图 4-33 按 Shape 比对

步骤5 经排查，最终确认是不同机器片上内存颗粒差异，Matmul是访存密集型算子，在不同机器上计算通信带宽抢占程度不同导致。

----结束

# 4.3 服务化工具

本章节主要介绍服务化工具使用场景和定位思路，具体问题的定位请参考 5.7.4 服务化性能调优定位案例。

1. 广泛性调优：通过预检工具（msprechecker）对系统、环境变量及配置文件等方面进行检查，识别可能影响服务化性能的潜在问题。  
2. 针对性地调优：通过调整当前需求的配置项（输入输出长度、并发数等）优化服务化的调度，有以下两种方法：a. 服务化专家建议工具（msservice_advisor）：此工具能够快速提升服务化的性能，但不足以进行精细化的调优。

b. 服务化自动寻优工具（modelevalstate）：适用于有针对性地提升服务化性能，能够达到人工调优所能实现的最佳性能的 $9 5 \%$ 。但该方法耗时较长，需要持续搜索参数以接近最优解。

3. 如果仍未达到预期，可以使用服务化调优工具（msServiceProfiler）进行深入分析，此工具适用于熟悉整个服务化运作方式的用户。

表 4-8 服务化性能工具介绍  

<table><tr><td rowspan=1 colspan=1>工具名称</td><td rowspan=1 colspan=1>工具简介</td></tr><tr><td rowspan=1 colspan=1>推理预检工具(msprechecker)</td><td rowspan=1 colspan=1>支持推理前、推理中和推理后的全流程检测。·推理前，提供一键式预检功能，全面排查环境变量、系统内核、配置文件等可能导致服务部署失败或性能下降的问题。·推理过程中，支持将环境相关的所有数据完整落盘。·推理结束后，支持对落盘文件进行比对，帮助识别差异点，便于复现基线环境。</td></tr><tr><td rowspan=1 colspan=1>服务化专家建议</td><td rowspan=1 colspan=1>根据当前的Benchmark输出结果及MindIEservice的config.json配置，结合理论分析性能上限，提出提升首令牌生成时间（TFTT）和吞吐量（Throughput）等关键指标的优化建议。</td></tr><tr><td rowspan=1 colspan=1>服务化自动寻优</td><td rowspan=1 colspan=1>提供MindIE服务化和vLLM服务化的参数自动优化功能。利用先进的检索算法，在参数空间中高效寻找最优解，实现自动化调优。该功能同时支持轻量化设计，部署快速便捷，确保搜索结果更加准确。</td></tr><tr><td rowspan=1 colspan=1>服务化调优工具( msServiceProfiler )</td><td rowspan=1 colspan=1>提供推理服务化性能数据采集接口的解析和拆解能力。此接口专为服务化调优设计，能够采集关键流程的起止时间点，识别并记录关键函数调用、关键事件、服务化调度等信息，同时支持采集算子信息，助力快速定位性能问题。如果想了解更多请参见《性能调优工具用户指南》的“msServiceProfiler服务化调优工具”章节。</td></tr></table>

# 5 TopN 性能问题的解决方案

# 简介

通信问题优化方案  
算子性能问题优化方案  
Host Bound问题定位及解决方法  
集群性能异常波动问题定位方法论  
ONNX离线推理场景解决方案  
MindIE推理性能解决方案  
版本升级性能劣化定位方法论

# 5.1 简介

本章节聚焦于性能问题根源的精确定位方法，阐述了从广泛的问题领域逐步深入至具体问题点的“由面及点”策略。在高性能计算与大规模分布式训练中，面对复杂多变的性能瓶颈，仅依靠单一维度的分析往往难以迅速、精准地锁定问题所在。为此，本章节针对七类常见性能问题，详细介绍了从宏观层面的趋势识别到微观层面的全面排查的技术手段。

表 5-1 性能问题一览表  

<table><tr><td>问题类型</td><td>问题介绍</td></tr><tr><td>通信问题</td><td>是指在分布式训练过程中，由于节点间数据传输效率低下 或快慢卡问题所引发的集群整体性能下降现象。首先通过 MindStudio Insight工具的4.2.4.3集群性能分析功能对 Profiling结果做初步分析，确认是快慢卡问题还是通信传 输问题，针对快慢卡问题，进一步对比快慢卡时间线 （Timeline）差异确认问题根因；针对通信传输效率低下 的问题，按不同原因（如通信小包、通信重传、字节未对 齐等）进行针对性优化。</td></tr><tr><td colspan="1" rowspan="1">算子性能问题</td><td colspan="1" rowspan="1">是指在训练过程中某些算子执行效率低、占用资源多，成为整体性能瓶颈的问题。利用advisor与MindStudioInsight工具自动识别高耗时或低效算子，结合调用栈与代码逻辑分析，提出算子替换、融合或优化的改进路径。</td></tr><tr><td colspan="1" rowspan="1">下发异常问题</td><td colspan="1" rowspan="1">指的是在模型训练过程中，由于某些卡的算子下发延迟或任务分配不均衡，导致整体训练效率下降的情况。该问题可通过从系统环境、任务分配及绑核策略等前置条件进行检查，并结合CPU运行状态与堆栈信息，定位由下发延迟引起的性能劣化。</td></tr><tr><td colspan="1" rowspan="1">集群性能问题</td><td colspan="1" rowspan="1">在大规模集群环境中，由于节点数目庞大且通信复杂，经常会遇到整体训练性能下降的问题。与小型集群相比，这种情况下生成的性能文件数据较大，使得问题根源的定位变得困难。通过利用msprof-analyze的clusteranalyse工具，可以高效地识别出表现异常的节点，进而将大规模集群的问题简化为多个小型集群或多卡系统的问题，便于进-步的详细分析和解决。</td></tr><tr><td colspan="1" rowspan="1">Atlas 200I/500 A2 推理产品场景</td><td colspan="1" rowspan="1">Atlas 200I/500A2推理产品推理性能受限于数据搬运瓶颈的问题，MTE2/MTE3类指令耗时占比高、模型推理延时大、吞吐率低。通过模型压缩量化、ONNX简化、AOE调优、msit debug surgeon自动优化、CANN版本升级等多种手段，减少冗余计算、优化内存布局、提升算子执行效率，从而缓解数据搬运压力，提升整体推理性能。</td></tr><tr><td colspan="1" rowspan="1">MindIE推理场景</td><td colspan="1" rowspan="1">服务化推理性能受调度机制与模型推理能力共同影响，且因测试场景多样（如并发、序列长度、时延、吞吐等需求），需灵活调整服务化参数并关注多维度指标。本节从性能调优出发，介绍调优思路与问题定位方法：先通过纯模型测试评估推理上限，识别瓶颈所在；再基于此结果结合调优目标修改服务化调度配置；提供使用预检、msServiceProfiler等工具定位性能问题的案例；提供针对DeepSeek模型的调优建议。</td></tr><tr><td colspan="1" rowspan="1">版本升级</td><td colspan="1" rowspan="1">适用于快速排查，用于确认近期是否有进行过任何变更，例如集群的重新规划或版本更新等。</td></tr></table>

# 5.2 通信问题优化方案

# 5.2.1 总体介绍

通信问题最直观的表现是，集群通信时间较长，通信时长占比远远大于计算，如图5-1所示；或者存在明显超长时间的通信算子，如图5-2所示，可以看到图中标注①与标注$\textcircled{2}$ 是两个时间远大于计算流的reduceScatter算子。

![](images/6670b8f01b126d95859edc1a12092c26df219fac62d3926a1ef6b786a44a051b.jpg)  
图 5-1 通信问题表现 1：通信时长远大于计算时长

![](images/a47f1dbb4ad46575b73459c63dcee0d9fef1dd1796414654d9bdbe08073852cc.jpg)  
图 5-2 通信问题典型表现 2：存在明显超长时间的通信算子

这里需要注意的是，某张卡的通信问题不一定是通信传输本身造成的，也可能是由于等待其它慢卡造成的，即快慢卡问题。

要排查某张卡的通信时长过长是通信传输本身的问题还是快慢卡问题，可通过如下方式：

按照4.2.4.3.2 通信（Communication）介绍，进入MindStudio Insight的通信耗时分析页签，若该卡的传输时间占比较高，则可认为是通信传输本身存在问题；同步时间占比较高，则可认为是快慢卡问题。

![](images/81b656537010796914402169ccfa2ebc2addbbe7c4b1600a8208b68a0441c002.jpg)  
图 5-3 通信时长分析

![](images/39ce3f5386d0f96898b5f36fb9af45ae5c585755c5c0a5b234e38098a8b5be55.jpg)

可通过4.2.4.3.1 概览（Summary）页签对比多卡计算、通信、空闲时间，观察是否是快慢卡问题。以图5-4为例，若观察到各卡的空闲时间和通信时间呈负相关

（即空闲时间长的，通信时间短；空闲时间短的，通信时间长），则有较大概率可以判断，该集群存在下发性能波动导致的快慢卡问题。同理，也存在计算时间和通信时间呈负相关的计算快慢卡问题。

![](images/3f86afb92605013b97fbe1544b421836a3b78e3547c06098b087ff3aaf6967e1.jpg)  
图 5-4 下发快慢卡问题

若确认为快慢卡问题，可参考5.2.2 快慢卡问题定位方法进一步定位造成快慢卡差异的原因。

若确认不是快慢卡问题，则关注通信本身，可能原因包括通信小包、通信重传、源地址与目标地址不对齐、Profiling性能膨胀、计算通信带宽抢占等原因。

# 5.2.2 快慢卡问题定位方法

# 5.2.2.1 总体介绍

快慢卡的快卡和慢卡是相对概念，快卡是在集群中首先完成计算任务的卡，慢卡是在集群任务中较慢完成计算任务的卡。在集群中往往需要完成集合通信，若不同卡完成任务的时间不同，则容易造成快卡等待慢卡完成计算的情况，从而使整个集群的性能劣化。

快慢卡是一种现象，背后的原因多种多样。通用的定位思路是，使用定点精确分析法，比对快卡和慢卡在MindStudio Insight时间线（Timeline）页签上的差异，确认具体原因。

形成慢卡的常见原因包括负载不均衡、计算性能波动、Host侧下发性能波动、数据加载性能波动等原因。

本节主要内容如下：

5.2.2.2 快慢卡定点精确分析法：介绍了快慢卡分析的通用思路。  
5.2.2.3 快慢卡定位Timeline操作案例：演示了如何利用MindStudio Insight的时间线（Timeline）页签定位快慢卡问题。  
5.2.2.4 快慢卡定位算子比对操作案例：演示了如何利用算子比对功能定位计算快慢卡问题。  
5.2.2.5 快慢卡案例补充：展示更多快慢卡典型案例。

# 5.2.2.2 快慢卡定点精确分析法

可以从MindStudio Insight时间线（Timeline）页签，快速定位快慢卡产生差异的根源。

![](images/8eaf64e9eb5f18a51d3b005ee8c7723136ed5edefef05674c1bb857df76c052a.jpg)  
图 5-5 定点精确分析方法

# 定位点

首先要找到产生快慢卡的问题点，进入通信页签的通信算子缩略图，迭代ID选择靠后的ID，并且首先观察差异最大的集合通信算子，从快慢卡影响最大的地方入手，右键跳转到对应的通信位置。以绿色算子为例，如图5-6所示。

![](images/8572bffc9b38050d4fb2cfd674c95cc31e87b00ffd28eeeffa9b39b12e84da46.jpg)  
图 5-6 定位点

1. 放大通信算子。找到需要对比的卡（找到耗时差距最大的卡，如0和3卡），如图5-7所示。

![](images/1fb5682c1c3bb8a7d1a19bb1a1884b6185446cd81a0d25d8ae1ed678c8d96c0b.jpg)  
图 5-7 定区间 1

2. 在通信算子上单击鼠标右键，选择“跳转至时间线视图”，跳转到时间线（Timeline）的对应位置。建议在两张卡HCCL通信算子所对应Python侧下发API的开头做一个旗子标记。

![](images/3e8b4209f783974e9ed87bc9fdf34e25fce408f7d166ea60d3e18fac4f03d341.jpg)  
图 5-8 定区间 2

3. 找到开始产生快慢卡问题的地方（白线位置），作为对比区域的开始，以两张卡的旗子标记分别作为两张卡对比区间的结束，分别得到快卡区域和慢卡区域，如图5-9所示。

![](images/cf8f3270e7d7b0ceb40acd931323cd820be68ea7e5e0402a2abf7c8bf7f6f825.jpg)  
图 5-9 查找产生快慢卡的地方

分别在快卡区域和慢卡区域对比不同部分，从而进一步找到产生差异的具体原因。  
以图5-10为例，造成快慢卡的地方主要有三块，分别是1和4、2和5以及3和6区域。

![](images/d65f685ea991745a148a22ced09d171d658e08170c27f4af8d86d09ebf9da8a1.jpg)  
图 5-10 示例图

至此已经定位到快慢卡的问题点，下一步就需要根据算子和代码定位快慢卡的根因。

# 5.2.2.3 快慢卡定位 Timeline 操作案例

本节介绍利用MindStudio Insight进一步定位快慢卡问题的具体操作案例。

若我们通过4.2.4.3.1 概览（Summary）已经初步锁定了集群中存在快慢卡不同步问题，并且通过4.2.4.3.2 通信（Communication）确认，集群通信时间里，传输时间占比低，等待或同步时间占比高，则确认当前集群存在“快慢卡问题”，需查看“通信耗时分析”。通过通信算子横向平铺，确认慢卡具体慢在哪里。

如图5-11所示，其中针对绿色的hcom allGather集合通信算子，时长较短的4卡、5卡、13卡为慢卡，而时长较长的卡（例如11卡、14卡等）为相对的快卡。

下一步定位目标为：分析慢卡在空白时间做什么？需前往时间线（Timeline）模块，查看具体差异点。

![](images/ef9d1e54d92af7d20438db186653668fd8384c8f5b1954721ce6750ebb27a6c9.jpg)  
图 5-11 通信算子平铺图

步骤1 选择通信算子，单击鼠标右键，选择“跳转至时间线视图”，由通信算子跳转至时间线视图，用旗帜标记大致范围，方便后续定位。

![](images/b647b0e6f5a945268c847b70ea2eb4acda4bdf90937152aeb2eba851af9622e5.jpg)  
图 5-12 由通信算子跳转至时间线视图

![](images/c5350706c56bd73c16bd5a83aa8c2c265008f7765601ad97b255aced959e45cc.jpg)  
图 5-13 用旗帜标记大致范围

步骤2 选择一个Step（按Step整屏显示），置顶对比慢卡（13卡）与快卡（14卡）的Overlap Analysis泳道，确认差异来源（Overlap Analysis泳道将Ascend Hardware层计算、通信任务统一投影，比对更清晰）。

![](images/3afdfc0bd0e91b202b08479c780b7a127a1fd1620f8afff5fdb421058de34f99.jpg)  
图 5-14 将研究区域限定在一个 Step 内（按 Step 整屏显示）

![](images/a26324409c8311aca541ce2b5166cccd5e7392dddf37c87d934988ed30e69f0f.jpg)  
图 5-15 置顶对比慢卡（13 卡）与快卡（14 卡）的覆盖分析泳道，确认差异来源

步骤3 框选统计可以看到，慢卡泳道计算任务次数明显多于快卡。

![](images/b9e4dbf687b436036ed2b27db679dfa79cd840a4ae50ed6a98c8a0367517a369.jpg)  
图5-16 框选统计

差异主要来源于后半段，说明是负载不均导致了快慢卡问题。

![](images/fd41306d34801cb0e4d35afe6b549a02f5a1b663eaea41f0e0a19879765bd93a.jpg)  
图 5-17 算子数差异对比

步骤4 根据async_npu下发连线，确认多出的算子由哪个Python API下发（锁定框选区域，可按照锁定区域展示连线，而不展示全量连线）。

![](images/6e891cc6ab3771474f6f5bc856217f5da4ef1fb0e5bc84e0e9ca95095c2109c4.jpg)  
图 5-18 async_npu 下发连线

对Ascend Hardware层框选统计得出，同样的Python侧某API，快卡下发1218个计算算子，慢卡下发3303个计算算子，所以是由于该Python侧某API负载不均导致快慢卡。

步骤5 和模型开发人员确认，该负载不均能否规避。----结束

# 5.2.2.4 快慢卡定位算子比对操作案例

若在MindStudio Insight的概览界面发现集群的快慢卡是计算时间波动导致的，除了使用5.2.2.2 快慢卡定点精确分析法中提到的定点精确分析法以外，还可以尝试比对快慢卡算子耗时，快速锁定差异来源。

# 从概览页确定快慢卡

在MindStudio Insight的概览界面查看计算/通信概览区域，可以看到，0-7卡为计算慢卡（计算时间长，通信时间短），8-15卡为计算快卡（计算时间短，通信时间长）后者通信时间长是等待前者所导致的。

![](images/85f080c1e9cf7a3a3c65c26f5e996413141bc5b3a74720aa715d31204f9ab473.jpg)  
图 5-19 计算快慢卡概览页

# 对比算子差异

按照4.2.4.4.3 算子（Operator）中描述，可以快速锁定造成耗时差异的算子，如图5-20所示，首先设置7、8两卡进入卡间比对模式，随后按总耗时升序排列。若快慢卡存在较大算子数量差异，说明存在计算负载任务不均衡的问题，可与模型开发人员确认，该负载不均能否规避；若某类算子数量一致，但平均耗时存在差异，可求助相关算子开发负责人，或者结合5.2.2.2 快慢卡定点精确分析法中方法，通过时间线（Timeline）进一步确认问题根因。

![](images/03016fa6dafa165c9ccb4362e1b1a43ac6152ca6f53e514a588c3e0936c85de3.jpg)  
图 5-20 算子卡间比对

同理，也可使用4.2.3 模型调优快速分析（msprof-analyze命令行工具）工具中的compare工具，进入KernelCompare比对页，分析算子差异。

图 5-21 compare 性能拆解比对工具 KernelCompare 比对页  

<table><tr><td></td><td>B</td><td></td><td>C</td><td></td><td></td><td></td><td>G H</td><td></td><td>J</td><td></td><td>R</td><td>L</td><td>M X</td><td></td><td>0</td></tr><tr><td></td><td></td><td></td><td>Base Profiling:</td><td></td><td></td><td></td><td>_ascend_pt</td><td>ComparisonProfiling:</td><td></td><td></td><td></td><td>ascend_pt</td><td></td><td></td><td></td></tr><tr><td>2 Order Id</td><td>Kemel</td><td>Input Shape</td><td>Total Duration(us)</td><td>Avg Duration(us)</td><td>Max Duration(us)</td><td>Min Duretion(us)</td><td>Calls</td><td>Total Duration(us)</td><td>Avg Duration(us)</td><td>Max Duration(us)</td><td>Min Duration(us)</td><td>Calls</td><td></td><td>Diff Total Ratio</td><td>Diff Avg Ratic</td></tr><tr><td>4</td><td>ZerosLike</td><td>*1627345920&quot;</td><td>3.913.10</td><td>3.913.10</td><td>3.913.10</td><td>3,913.10</td><td>1</td><td>3,910.00</td><td>3.910.00</td><td>3,910.00</td><td>3.910.00</td><td></td><td>1.00</td><td>1.00</td><td></td></tr><tr><td></td><td>ZerosLike</td><td>8.24</td><td>1.08 95.32</td><td>1.08</td><td>1.08 4.44</td><td>1.08</td><td>24 1</td><td>1.26 95.52</td><td>1.26 3.98</td><td>1.26 4.38</td><td>1.26</td><td>1</td><td>116.67%</td><td></td><td>116.67%</td></tr><tr><td>6 3 4</td><td>Cast</td><td>*2,4096* *2,4096,1&quot;</td><td>42.44</td><td>3.97 2.65</td><td>2.82</td><td>3.48 2.52</td><td>16</td><td>42.64</td><td>2.67</td><td>2.88</td><td>3.46 2.56</td><td>24 16</td><td>100.47% 100.21%</td><td></td><td>100.25%</td></tr><tr><td>5</td><td>Cast</td><td>4096.1.1,128*</td><td>35.320.38</td><td>9.20</td><td>13.24</td><td>7.94</td><td>3.840</td><td>35.478.60</td><td>9.24</td><td>13.20</td><td>7.98</td><td>3.840</td><td>100.459</td><td></td><td>100.75% 100.43%</td></tr><tr><td>9</td><td>cast Cast</td><td>4096,2,4000°</td><td>856.84</td><td>53.55</td><td>58.60</td><td>50.34</td><td>16</td><td>848.52</td><td>53.03</td><td>56.18</td><td>50.54</td><td>16</td><td>0.99</td><td></td><td></td></tr><tr><td></td><td>Cast</td><td>*4096.2°</td><td>113.76</td><td>3.56</td><td>4.40</td><td>2.56</td><td>32</td><td>113.52</td><td>3.55</td><td>4.24</td><td>2.42</td><td>32</td><td>1.00</td><td></td><td>0.99 1.00</td></tr><tr><td>11 8</td><td>Cast</td><td>8192*</td><td>20.44</td><td>2.56</td><td>2.74</td><td>2.42</td><td>8</td><td>20.42</td><td>2.55</td><td>2.60</td><td>2.48</td><td>8</td><td>1.00</td><td></td><td>1.00</td></tr><tr><td>12</td><td>Cast</td><td>*4000,5120*</td><td>1.846.82</td><td>71.03</td><td>139.26</td><td>30.32</td><td>26</td><td>1,809.16</td><td>69.58</td><td>139.52</td><td>30.46</td><td>28</td><td>0.98</td><td>0.90</td><td></td></tr><tr><td>1310</td><td>Cast</td><td>5120*</td><td>2.591.84</td><td>1.88</td><td>2.26</td><td>1.32</td><td>1.377</td><td>2.595.64</td><td>1.88</td><td>2.26</td><td>1.30</td><td>1.377</td><td>100.15%</td><td>1.00</td><td></td></tr><tr><td>1411</td><td>Cast</td><td>*5120,1728*</td><td>7,295.46</td><td>20.27</td><td>52.12</td><td>16.66</td><td>360</td><td>7.313.74</td><td>20.32</td><td>50.92</td><td>16.66</td><td></td><td>100.25%</td><td></td><td>100.25%</td></tr><tr><td>12 15 16 13</td><td>Cast</td><td>*3456,5120*</td><td>12,098.15</td><td>33.61</td><td>90.66</td><td>25.10</td><td>360 360</td><td>12.079.69 4.195.06</td><td>33.55</td><td>90.88 26.16</td><td>24.88</td><td></td><td>1.00</td><td></td><td>1.00</td></tr><tr><td>17 14</td><td>Cast Cast</td><td>5120,640* *1920,5120*</td><td>4.190.86 10,646.93</td><td>11.64 29.57</td><td>27.35 70.82</td><td>10.26 16.92</td><td>360</td><td>10.642.15</td><td>11.65 29.56</td><td>70.42</td><td>10.22 16.88</td><td></td><td>100.10%</td><td></td><td></td></tr><tr><td>1815</td><td>Cast</td><td>*2,4096,5120*</td><td>1.543.83</td><td>192.98</td><td>194.78</td><td>191.60</td><td>8</td><td>1,531.63</td><td>191.44</td><td>192.14</td><td>190.04</td><td></td><td>1.00 0.99</td><td></td><td>1.00</td></tr><tr><td>1916</td><td>Less</td><td>*2.4096:*</td><td>21.34</td><td>2.67</td><td>2.72</td><td>2.56</td><td>8</td><td>21.22</td><td>2.65</td><td>2.70</td><td>2.58</td><td></td><td>0.99</td><td></td><td>0.99 0.99</td></tr><tr><td>20 17</td><td>Less</td><td>4096,2.*</td><td>20.40</td><td>2.55</td><td>2.66</td><td>2.42</td><td>8</td><td>20.84</td><td>2.60</td><td>2.84</td><td>2.42</td><td></td><td>102.16%</td><td></td><td></td></tr><tr><td>21 18 22 19</td><td>GreaterEqual</td><td>*2.4096;*</td><td>40.74</td><td>5.09</td><td>5.38 4.98</td><td>4.94</td><td>B</td><td>40.02 39.46</td><td>5.00</td><td>5.20</td><td>4.84</td><td></td><td>0.98</td><td></td><td>0.98</td></tr><tr><td>23 20</td><td>GreaterEqual</td><td>*4096.2.&quot; *2,4096;2,4096*</td><td>39.32 13.36</td><td>4.92 1.67</td><td>1.82</td><td>4.82</td><td>8 8</td><td>13.28</td><td>4.93 1.66</td><td>5.10 1.78</td><td>4.78 1.56</td><td></td><td></td><td></td><td></td></tr><tr><td>24 21</td><td>LogicalOr LoglcalOr</td><td>*4096.2:4096.2*</td><td>14.44</td><td>1.80</td><td>1.84</td><td>1.56 1.76</td><td>B</td><td>14.44</td><td>1.80</td><td>1.84</td><td>1.76</td><td></td><td>0.99</td><td></td><td>0.99</td></tr><tr><td>25 22</td><td>Sub</td><td>&quot;2.4096:&quot;</td><td>40.72</td><td>5.09</td><td>5.42</td><td>4.88</td><td>8</td><td>40.12</td><td>5.01</td><td>5.14</td><td>4.96</td><td></td><td>1.00 0.99</td><td></td><td>1.00</td></tr><tr><td>26 23</td><td>sub</td><td></td><td>4096.24000496.2.16</td><td>100.02</td><td>101.12</td><td>99.10</td><td>8</td><td>796.88</td><td>99.61</td><td>100.76</td><td>98.72</td><td></td><td>1.00</td><td></td><td>1.00 0.98</td></tr><tr><td>27 24</td><td>Sub</td><td>*4096,2:*</td><td>39.44</td><td>4.93</td><td>5.04</td><td>4.84</td><td>8</td><td>39.82</td><td>4.98</td><td>5.18</td><td>4.82</td><td></td><td>100.9</td><td></td><td></td></tr><tr><td>28 25 29 26</td><td>Sub Sub</td><td>4096,2;4096,2*</td><td>23.06</td><td>2.88</td><td>3.12</td><td>2.70</td><td>8 8</td><td>22.80 21.00</td><td>2.85</td><td>3.02</td><td>2.68</td><td></td><td>0.99</td><td></td><td>0.99</td></tr><tr><td>30 27</td><td>sub</td><td>:8192* *8192:8192*</td><td>06</td><td>2.63 2.52</td><td>2.70 2.58</td><td>2.48</td><td>8</td><td>20.06</td><td></td><td>2</td><td>1248</td><td></td><td>1.00</td><td></td><td>1.00</td></tr><tr><td>31 28</td><td>LogicalNot</td><td>*2.4096*</td><td>14.68</td><td>1.83</td><td>1.96</td><td>2.48 1.66</td><td>8</td><td>14.62</td><td>1.83</td><td>1.92</td><td>1.70</td><td></td><td>0.99 1.00</td><td></td><td>1.00 1.00</td></tr><tr><td>32 29 33 30</td><td>LogicalNot LogicalNot</td><td>2.4096,1* *4096.2*</td><td>14.26 27.60</td><td>1.78 1.73</td><td>1.82 1.88</td><td>1.70 1.56</td><td>B 16</td><td>14.56 27.50</td><td>1.82 1.72</td><td>1.88 1.90</td><td>1.76 1.52</td><td></td></table>

# 5.2.2.5 快慢卡案例补充

# 5.2.2.5.1 案例一：集群保存 checkpoint 后性能持续劣化

问题描述

4机64卡，保存checkpoint后性能持续劣化。

# 问题分析

首先，如图5-22所示，利用4.2.3 模型调优快速分析（msprof-analyze命令行工具）中的msprof-analyze工具进行集群分析后，发现communication time和free time卡间波动趋势呈负相关（即同一张Rank中，communication耗时长的Free耗时短，communication耗时短的Free耗时长），推测出Free Time时间最短的0卡最先结束，等待其他卡，为快卡；Free Time时间最长的1卡最后结束，为慢卡。即此案例为Host侧下发性能波动导致的快慢卡问题。

![](images/dde0c6351e24ff98b48d998930a885b7c571cbc7d2d3b72b9c61d0cfeb07a53e.jpg)

进入时间线（Timeline）视图，如图5-23与图5-24所示，发现0卡通信等待发生在反向传播后的梯度汇总阶段。对比快慢卡时间线（Timeline）的相近位置，发现慢卡Rank1的Step末尾存在异常空洞，在此阶段卡顿，导致了快卡Rank0的等待。

![](images/32a3cd6328b230c32d71efb3909801be9eb48dc5cd1b40345715c9fc162328ec.jpg)  
图 5-23 Timeline 视图（Rank0）

![](images/0a02f775e4bb136e5fc1475ae08b05e150593a36279645d908b9fbcdb94cc6d9.jpg)  
图 5-24 Timeline 视图（Rank1）

# 定位完成

该问题最终定位原因是代码在慢卡Step末尾存在未释放内存，手动清空后问题消失。

# 5.2.2.5.2 案例二：集群 AllReduce 通信算子等待时间过长

问题描述

256p集群allreduce通信算子等待时间长，线性度不达标。

问题分析

问题直接表现是dp通信域存在耗时较长notify wait，如图5-25所示。

![](images/9be0bb65140b368ec30ae9bb7f4b4727bdeb198695f6ce123270f3c9229aa3d2.jpg)  
图 5-25 dp 通信域 timeline 分析

在通信界面找到此通信域，可以看到存在一个很长的notify wait，Elapse Time（通信总时间）差距巨大，最快卡和最慢卡差距 $2 0 0 \mathsf { m } \mathsf { s } ^ { + }$ 。

![](images/6060347977a4eb3eed672fee0a3a6ef875183141f35891fb0a679e32a06eaddd.jpg)  
图 5-26 通信时长分析

如图5-27所示，同时打开快卡0卡和慢卡208卡的时间线（Timeline）界面，发现慢卡208主要是慢在tp通信上。慢卡208卡在一次micro的计算中tp域通信耗时494ms，快卡0卡tp域通信耗时340ms，差距较大。快慢卡的tp域通信传输速度基本一致，主要还是等待时间造成的差距。

![](images/ced48c0751e94c29c707608729862953f991d80036ffee8a4b4554660107937c.jpg)  
图 5-27 快卡 0 卡和慢卡 208 卡的 Timeline 界面

进一步观察和208卡同一个tp域的各卡，发现其中214卡是拖累整个tp域的慢卡。即214卡从tp域拖累了208卡，而208卡沿着dp域进一步拖累了0卡，214卡为其中真正的根因慢卡。如图5-28所示，对比208卡和214卡的时间线（Timeline），可以看到214卡存在Host下发较慢的问题。

![](images/aae37971860388007f0a536be0d2e8a42a1aea9c66ba6329e82c66000cda5c82.jpg)  
图 5-28 208 卡和 214 卡的 Timeline

![](images/2af444422e63d99f24c02d0016e31d386a4c3b94415b385f084fca4c68e80991.jpg)

如图5-29所示，通过时间线（Timeline）的框选统计功能，对比208卡与214卡的负责下发的CANN侧接口，可以看到214卡的CANN侧接口耗时明显高于208卡。

![](images/c798867d5cbf4920857b7e58d7e665263a793cef1a9cd0bd42a2003acec90ad9.jpg)  
图 5-29 208 卡与 214 卡 CANN 侧接口框选统计

# 定位完成

通过快慢卡时间线（Timeline）定点比对分析，可以初步确认快慢卡差异是慢卡（214卡）的Host侧下发瓶颈导致的。进一步解决Host侧下发瓶颈，可参考5.4 Host Bound问题定位及解决方法中内容。

# 5.2.3 通信重传

# 5.2.3.1 基于集群数据分析是否出现通信重传

在分布式训练中，如果发现某个通信算子耗时超过4秒（通信重传的典型阈值），在排除快慢卡问题的可能性后，可以怀疑是发生了数据包重传。仅凭单张卡的性能数据，很难直接区分这两种情况，可通过MindStudio Insight进一步排查。

通常通信重传多出现在集群间的性能抖动问题中，可以通过MindStudio Insight对比正常和异常Step的Profiling数据确认。

步骤1 在MindStudio Insight集群结果概览页，可以看到不同Step的耗时分布，如图5-30所示。

![](images/36e763ccc5f499f7d5526f3678006678bd7699623f495ab27377dab5d286ff04.jpg)  
图 5-30 MindStudio Insight 集群概览页

步骤2 对比两个Step的概览结果，可以发现各卡间的计算时间和空闲时间比较接近，说明没有明显的快慢卡问题，差异点主要体现在通信耗时（通信时间（被覆盖） $^ +$ 通信时间（未被覆盖））上。

步骤3 同时同步对比所有卡，发现Step11和Step12每一张卡的通信耗时差距都在4.7s左右。

步骤4 如图5-31所示，观察320卡的时间线（Timeline）结果，算子为LinearWithGradAccumulationAndAsyncCommunication，该算子为用于同步的通信算子，后续任务下发时（AscendCL@aclnnTopK）被阻塞。

![](images/e8ceae8fd8787737c3259875a2162a1ea750bd6ec20b78dbd025c38a43c8683d.jpg)  
图 5-31 Rank320 的 Timeline

结合各项数据分析，NPU（计算）和CPU（调度）均正常，各卡间通信时间的差异来源于点对点通信异常超时，大概率为网络异常导致的通信重传。

步骤5 发现通信重传问题后，建议排查交换机网络是否正常配置，例如，是否未添加PFC控制机制，导致PFC拥塞，进而导致通信重传。

----结束

# 5.2.3.2 通信重传典型案例

# 问题描述

LLaMA3-70B模型从4机迁移至32机集群时，发生线性度劣化。

# 问题分析

如图5-32所示，利用4.2.3 模型调优快速分析（msprof-analyze命令行工具）分别对正常4机集群与异常32机集群进行分析，对比两者结果发现，耗时差异主要在通信上，32机集群整体通信耗时更长。

![](images/9efa790ae942b34aa9eb370a4d7c5186bdae01c3889a8efbe378e2bf78bb2c0b.jpg)  
图 5-32 cluster_step_trace_time.csv 交付件，正常 4 机与异常 32 机对比

以0卡为例，对比正常集群与异常集群的0卡，发现问题出现在迭代末尾的allReduce和broadcast算子上，如图5-33所示。

![](images/c27a9a65f5bdc3181020239b3507b6dc0591586fea4b43a8e30a55c1e1f5b021.jpg)  
图 5-33 异常集群 0 卡 Timeline

根据通信算子选中详情的dst rank（目标卡，在notify wait通信事件中一般代表被等待的慢卡）不断跳转，发现是440卡拖累了同一tp通信域内的其它卡，如图5-34所示。

![](images/212656390be04b81ad5c8b24b33344f165321f58ab9ac138c1b1d7ef29e68523.jpg)  
图 5-34 异常集群 440 卡 Timeline

进入通信（Communication）页签，选择通信耗时分析，找到对应通信域。按等待时间（Wait Time）升序排列，找到该通信域内等待时间较短、传输时间较长的卡。如图5-35所示。

![](images/204b3335fd14f57163440f9a5eeaa8c74a34df891c625d6dd5709292ec74ebc6.jpg)  
图 5-35 通信（Communication）页签 $>$ 通信耗时分析

进一步查看传输时间较长卡的带宽分析，发现存在大量带宽极低的RDMA通信包，如图5-36所示。怀疑存在网络传输问题，需要排查网络配置。

![](images/62a4471c04f720306df21e08ffbbe06adf32974878f0a09816d33df12b135990.jpg)  
图 5-36 带宽分析

# 问题解决

排查网络配置后发现，交换机和计算节点服务器之间的流量走在了无PFC拥塞控制的队列上，导致相应队列上网络出现大量报文丢包，从而导致RDMA数据网络报文重传，正确配置相关环境变量后，该问题解决。

# 5.2.4 通信小包

集群通信过程中，每次通信存在固定的时间开销，用于通信域初始化、通信建链、通信域销毁等操作。若通信时存在大量小包高频次传输，那么花在固定通信操作上的时间开销将远大于传输数据本身的时间，导致传输效率降低，通信带宽下降。

# 5.2.4.1 案例：ZeRO3 模式导致通信小包

问题描述

同一模型从NPU迁移至GPU，性能不达标。

问题分析

使用4.2.3 模型调优快速分析（msprof-analyze命令行工具）中的compare性能比对工具，对比基线GPU和迁移后的NPU的数据，发现通信未掩盖时间存在较大差距，如图5-37所示。

![](images/721d331d11065f1665cd36a857718f63015bc37eb0af65eba1067269bbd27898.jpg)  
图 5-37 compare 对比 GPU 与 NPU

对比基线GPU与迁移后NPU的时间线（Timeline），发现差异主要来源于Step结尾的allGather通信部分，NPU耗时100ms左右，GPU耗时仅4ms左右，如图5-38所示。

![](images/5f45f4eecadc753719b9ecb1b0bfb5bd91ab6f0ffcfe0b7696bff0e136e52152.jpg)  
图 5-38 GPU 与 NPU 的 Timeline 对比

NPU这部分通信算子带宽仅0.2GB/s，观察到存在大量allGather通信算子，传输数据量仅256byte，通信建链占据大部分耗时，如图5-39所示。这部分算子主要在进行ZeRO3操作。

![](images/fb8bab912b5231003e085f4c2a48432c716ee35d796c2e1f6d37982f53fbaada.jpg)  
图 5-39 allGather 通信小包算子

ZeRO（Zero Redundancy Optimizer，零冗余优化器）模式是一种节省内存的方法，ZeRO模式会带来通信策略的变化。ZeRO的主要原理是将优化器状态、梯度、权重等数据进行切分，在需要使用时再通过集合通信进行同步，来减少显存的使用峰值，是一种典型的时间换空间的办法。

# 问题解决

将ZeRO3算法修改为ZeRO2算法，即不再对模型权重进行切分，虽然会增加显存占用，但是减少了设备间的通信开销。替换为ZeRO2后，总体性能提升2.7%。

# 5.2.5 源地址与目的地址字节未对齐

字节未对齐，指的是在集群进行通信时，如果源地址或目的地址没有按照特定的字节对齐（例如512字节对齐），会导致传输带宽显著下降，进而影响通信性能。这种未对齐问题常见于SDMA传输（即节点内传输），多出现于ZeRO算法中。解决方法通常包括对数据进行适当的填充（padding），以确保地址对齐，从而优化通信性能。

# 问题描述

单机八卡性能低于预期。通过MindStudio Insight的通信矩阵分析，发现平均带宽很低，只有1-2GB/s，远低于经验值，如图5-40所示。

![](images/b282c74c25d84f2e47e5681573bf0da411d58f2ee9bb72b56ef2c6009e784f83.jpg)  
图 5-40 集群通信矩阵分析

# 问题分析

进一步定位具体算子，发现是一个allGather算子带宽特别低，耗时 $3 0 0 + \mathsf { m s }$ ，带宽只有0.1GB/，如图5-41所示。

![](images/91723aa25482d79bed9449addfa312e491f13248b8d0bc966b52cbb91ac37d75.jpg)  
图 5-41 allGather 算子 Timeline 选中详情

经HCCL专家分析，是DP通信域的allGather通信算子源地址与目标地址无法对齐，导致通信性能劣化严重。

针对DP通信域的allGather入桶做字节对齐的padding，对齐字节。当前AscendSpeed已适配针对DP通信域的allGather入桶做了字节对齐的padding，DeepSpeed和Megatron还未修改，需要去DeepSpeed源码里改nccl_start_alignment_factor，如图5-42所示，修改后，allgather耗时从350ms变成了50ms。

![](images/fb87570bec4b36946341b147699771fde27ae96efc84bdb33b15d00d5a10df64.jpg)  
图 5-42 修改 nccl_start_alignment_factor 令字节对齐

# 5.2.6 计算通信带宽抢占

MatMul、FA等算子属于访存密集型算子，容易发生mte bound。此类算子与通信算子并行时，如图5-43所示，会发生计算通信对于内存带宽的抢占，导致通信传输带宽低于经验值（可能会下降1\~2倍左右，但不会特别低），如图5-44所示。

解决方法：若计算通信并行导致的带宽抢占现象较为严重，可以比较通算并行与未并行的性能数据，评估带宽抢占的影响是否超过了通算并行的收益，选择性能更优的方式。

![](images/382c5922abb29b22ba2ab640cc32294e1cdf7cc041657f02a3de5d29185a0737.jpg)  
图 5-43 Matmul 算子与 allGather 通信算子并行

![](images/4a6fbcbaf9fb07fba45ecac4da07e423d4bc69510cb4ac918e3d885090fa8baa.jpg)  
图 5-44 allGather 算子发生计算通信带宽抢占

# 5.2.7 Profiling 采集性能膨胀

在按照4.2.2 模型调优性能采集工具采集Profiling数据的过程中，若开启了较高的采集等级，或者配置了较多采集项，性能数据采集打点时，可能会对Host侧造成较大压力，导致采集的性能数据膨胀失真。Profiling采集级别设置为Level2时，会造成通信调度膨胀，具体可能表现为，通信算子Host侧下发被阻塞，出现大段Free时长。

解决方法：适当降低采集等级，建议非必要不开启Level2。若一定要开启，需结合Level1数据对比分析，以排除性能膨胀对数据的干扰。

# 5.3 算子性能问题优化方案

# 5.3.1 算子性能问题定位方法

算子性能问题是深度学习模型中的一个关键挑战，具体表现为部分基础计算单元的执行效率低下，从而影响整个模型的运行速度并造成资源浪费。这类问题需要借助专门的分析工具和代码优化技术来解决。例如，在评估融合算子性能时，可以通过对比不同配置下的计算时间、内存使用量等指标来进行综合判断。

![](images/1821ea1dedcdc3c3b1ea2fee0a5de07cfd4bab448b870986a18f2509a73d26f2.jpg)  
图 5-45 算子性能问题定位

表 5-2 算子性能问题定位方法  

<table><tr><td colspan="1" rowspan="1">分析方式</td><td colspan="1" rowspan="1">分析目的</td><td colspan="1" rowspan="1">处理思路</td></tr><tr><td colspan="1" rowspan="2">advisor分析</td><td colspan="1" rowspan="1">AI CPU算子耗时占比问题。降低AI CPU算子耗时。</td><td colspan="1" rowspan="1">AICPU算子首先基于算子名在时间线（Timeline）中定位算子位置，接着基于调用栈找到代码中的位置，然后尝试相同逻辑替换，如果无法替换，可记录算子shape、type等信息并联系算子负责人，确认是否支持该case。</td></tr><tr><td colspan="1" rowspan="1">算子编译问题</td><td colspan="1" rowspan="1">可尝试在Python训练开始前添加如下代码，指定二进制模式。如果无效则记录算子shape、type等信息并联系算子负责人，确认是否支持该case。torch_npu.npu.set_compile_mode(jit_compile=False)torch_npu.npu.config.allow_internal_format = False</td></tr><tr><td colspan="1" rowspan="1">单算子分析</td><td colspan="1" rowspan="1">vector算子分析</td><td colspan="1" rowspan="1">vector算子优化方法是通过修改代码逻辑进行消减，优化逻辑如下：算子亲和优化：具体请参见5.3.2亲和算子优化策略。模型代码优化：结合算子分析，模型代码层面尝试对算子调用，如冗余消除、shape优化、等效替换等修改，具体请参见5.3.3模型代码优化策略。版本更新：联系昇腾社区反馈，确认新版本是否有优化或后续优化的计划。根据advisor可融合算子分析，排查是否有设计融合算子的必要。若有必要，开发者也可尝试自行开发融合算子进行替换。</td></tr><tr><td colspan="1" rowspan="1"></td><td colspan="1" rowspan="1">cube算子分析</td><td colspan="1" rowspan="1">基于4.2.4模型调优深入分析（MindStudioInsight）中的operator页签，查看算子占比，选取最高耗时的TopN算子，并详细分析input shape下平均aicore性能，记录异常算子及shape，并联系算子负责人确认优化计划。·MACratio：表示cube计算单元是否充分使用，一般理想情况是80%。·MTEratio：算子中内存搬运过程，若MTEratio值过高，则表示内存搬运存在瓶颈。当算子性能无法达成预期，优化步骤如下：·算子亲和优化：具体请参见5.3.2亲和算子优化策略。模型代码优化：结合算子分析，模型代码层面尝试对算子调用，如冗余消除、shape优化、等效替换等修改，具体请参见5.3.3模型代码优化策略。版本更新：联系昇腾社区反馈，确认新版本是否有优化或后续优化的计划。</td></tr><tr><td colspan="1" rowspan="1">融合算子/亲和API替换</td><td colspan="1" rowspan="1">使用融合算子/亲和API替换，可减少不必要的小算子下发，提高Al Core利用率。</td><td colspan="1" rowspan="1">advisor中AffinityAPI Issues分析器可自动识别融合算子，结合调用栈可定位代码位置，进行融合算子/亲和API替换。</td></tr><tr><td colspan="1" rowspan="1">融合算子开发</td><td colspan="1" rowspan="1">如需进一步提升模型性能，可以考虑进行融合算子开发，融合算子主要为了减少小算子下发，进而减少空闲时间占比。</td><td colspan="1" rowspan="1">advisorcsv交付件中，针对Host瓶颈和MTE瓶颈的算子序列分析结果已充分展现，并清晰标注了算子序列存在Host瓶颈或MTE瓶颈。为了进一步提升性能，建议深入剖析代码逻辑，判断能否通过算子合并等手段缓解瓶颈现象。</td></tr></table>

# 5.3.2 亲和算子优化策略

# 问题描述

在B4训练BERT模型的时候发现和A100性能差距非常明显。训练相同的step，A100耗时45s，B4耗时1000s，NPU性能约为0.045GPU。

# 案例分析

1. 查看覆盖分析表，发现整个step基本都在计算，Free占比非常低，因此首先从计算入手优化。

图 5-46 覆盖分析表  

<table><tr><td>名称 时间(%)</td><td>总时间（us)</td><td>调用数</td><td>A 平均值(us)</td><td>最小值(us)</td><td>最大值(us) A</td></tr><tr><td>Computing 96.52</td><td>1009318720.812</td><td>555324</td><td>1817.5312</td><td>0.94</td><td>173183.62</td></tr><tr><td>Free 3.48</td><td>36394469.618</td><td>555323</td><td>65.5375</td><td>0.02</td><td>1461159.314</td></tr></table>

![](images/d44a84faa0a91f1135a17a9cd34d9c0c3a4eb61255a6b566614a278b316fa9ee.jpg)  
图 5-47 查看计算算子

2. 查看算子耗时，发现IndexPutV2算子计算占了总耗时的 $7 5 \%$ ，因此需要对该算子优化，具体优化方法请参考表5-4。

# 说明

本示例以IndexPutV2算子为例，其他算子情况类似，具体请参见表5-3和表5-4。

表 5-3 Linear、Reduce_Sum、BatchMatMulV2、RepeatInterleave 及Gatherelement 算子代码优化  

<table><tr><td colspan="1" rowspan="1">算子名称</td><td colspan="1" rowspan="1">代码优化前</td><td colspan="1" rowspan="1">代码优化后</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">Linear</td><td colspan="1" rowspan="1">####修改前###model=nn.Linear(K,N) input=torch.randn(M,K)res=model(input)</td><td colspan="1" rowspan="1">###修改后###res=torch.addmm(bias,input,weight.t())#此处的input为(M,K),weight为(N,K)---weight.t()为(K,N)</td><td colspan="1" rowspan="1">不区分场景，仅提供一种优化方案。</td></tr><tr><td colspan="1" rowspan="1">Reduce_sum</td><td colspan="1" rowspan="1">mask =torch.nn.functional.one_hot(indices,num_classes=self.num_experts).sum(dim=1)</td><td colspan="1" rowspan="1">temp_mask =torch.zeros(indices.shape[0],self.num_experts,device="npu",dtype=torch.bfloat16)mask=temp_mask.scatter_(-1,indices,1.0)</td><td colspan="1" rowspan="1">mask创建方式从onehot+reducesum变成了zeros+scatter，局部计算时间从2.2ms优化到0.08ms，总时间优化306ms。</td></tr><tr><td colspan="1" rowspan="1">BatchMatMulV2</td><td colspan="1" rowspan="1">output = tf.matmul(a, b,transpose_a=True) # a: [bs, n,1], b:[n, 1]</td><td colspan="1" rowspan="1">a_ = tf.transpose(a, perm=[0,2, 1])a_=tf.reshape(a_,[-1,a_.shape[2]])output=tf.matmul(a_,b,transpose_a=True)#a_:[bs,n],b:[n,1]</td><td colspan="1" rowspan="1">该算子在输入shape:[b, n, 1]与[n,1]输出shape:[b, 1, 1]时性能会劣化，当bmm算子输出shape存在[1,1]的情况时需要规避，将b与1进行合轴，tf.matmul在两个相乘矩阵为两维自动执行MatMul算子，当输入shape有[b,1,1]时，反向传播也会执行该算子，可替换为点乘。</td></tr><tr><td colspan="1" rowspan="1">Repeatlnterleave</td><td colspan="1" rowspan="1">valid_lens =torch.repeat_interleave(valid_lens, shape[1])</td><td colspan="1" rowspan="1">valid_lens =valid_lens.unsqueeze(-1).expand(-1, shape[1]).reshape(-1)</td><td colspan="1" rowspan="1">使用改变shape从而切换高性能分支的方法优化该算子。在输入第二个维度时，不直接传入shape[1]，而是将shape[1]替换为长度2048的1维tensor，形式为torch.tensor([shape[1],shape[1],shape[1],...,shape[1]])。</td></tr><tr><td colspan="1" rowspan="1">Gatherelement</td><td colspan="1" rowspan="1">pt =logit.gather(1.target).view(-1) +epslogpt=torch.log(pt)alpha=self.alpha.to(logpt.device)alpha_class=alpha.gather(0,target.view(-1))</td><td colspan="1" rowspan="1">pt =logit[torch.arange(logit.size(0)),target.squeeze(1)] + epslogpt=torch.log(pt)alpha=self.alpha.to(logpt.devicealpha_class=torch.index_select(alpha,0,target.view(-1))</td><td colspan="1" rowspan="1">当前NPU上调用Gatherelement算子会有性能劣化，使用torch.index_select函数替换torch.gather函数后，调用算子修改为GatherV2规避，修改时需要注意修改索引。</td></tr></table>

表 5-4 其他算子优化  

<table><tr><td colspan="1" rowspan="1">算子名称</td><td colspan="1" rowspan="1">官方链接</td></tr><tr><td colspan="1" rowspan="1">IndexPutV2</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/60RC3/ptmoddevg/trainingmigrguide/performance_tuning_0033.html</td></tr><tr><td colspan="1" rowspan="1">MatMul/hcom_allReduce</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/60RC3/ptmoddevg/trainingmigrguide/performance_tuning_0026.html</td></tr><tr><td colspan="1" rowspan="1">Nonzero</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/60RC3/ptmoddevg/trainingmigrguide/performance_tuning_0034.html</td></tr><tr><td colspan="1" rowspan="1">where</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/60RC3/ptmoddevg/trainingmigrguide/performance_tuning_0o35.html</td></tr><tr><td colspan="1" rowspan="1">RotaryMul &amp; RotaryMulGrad（融合算子）</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/600/ptmoddevg/trainingmigrguide/performance_tuning_0023.html</td></tr><tr><td colspan="1" rowspan="1">RmsNorm &amp; RmsNormGrad</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/600/ptmoddevg/trainingmigrguide/performance_tuning_0o24.html</td></tr><tr><td colspan="1" rowspan="1">ScaledMaskedSoftmax &amp;ScaledMaskedSoftmaxGrad</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/600/ptmoddevg/trainingmigrguide/performance_tuning_0025.html</td></tr><tr><td colspan="1" rowspan="1">MatmulAllReduce</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/600/ptmoddevg/trainingmigrguide/performance_tuning_0026.html</td></tr><tr><td colspan="1" rowspan="1">FlashAttentionScore</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/600/ptmoddevg/trainingmigrguide/performance_tuning_0027.html</td></tr><tr><td colspan="1" rowspan="1">SwiGlu</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/ detail/zh/Pytorch/600/ptmoddevg/trainingmigrguide/performance_tuning_0100.html</td></tr><tr><td colspan="1" rowspan="1">融合优化器</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/600/ptmoddevg/trainingmigrguide/performance_tuning_0028.html</td></tr><tr><td colspan="1" rowspan="1">融合算子替换官方文档</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/60RC3/ptmoddevg/trainingmigrguide/performance_tuning_0023.html</td></tr><tr><td colspan="1" rowspan="1">亲和算子替换官方文档</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/60RC3/ptmoddevg/trainingmigrguide/performance_tuning_0033.html</td></tr><tr><td colspan="1" rowspan="1">亲和API替换官方文档</td><td colspan="1" rowspan="1">https://www.hiascend.com/document/detail/zh/Pytorch/60RC3/ptmoddevg/trainingmigrguide/performance_tuning_0036.html</td></tr></table>

# 5.3.3 模型代码优化策略

基于Profiling性能数据， 结合NPU相关的特性， 能够进一步提升模型的性能，具体优化流程请参见图5-48，常见案例请参见表5-5 。

![](images/aa832dbc2ab0b60973e451cd4cc411ccf312afc53f000eb2028718144293ab6f.jpg)  
图 5-48 模型代码优化流程

1. 获取模型pr性能数据。  
2. 定位到模型性能问题，此处一般是指单点CPU操作或算子执行的耗时过长，超出预期。  
3. 基于性能数据中调用栈的关系，找到问题代码段。  
4. 深入分析问题代码段，找出具体问题。  
5. 采用相应的优化措施，例如通过消除冗余代码或用更亲和的实现方式来替代原有代码，从而提升性能。

表 5-5 常见微调案例  

<table><tr><td colspan="1" rowspan="1">问题类型</td><td colspan="1" rowspan="1">模型问题</td><td colspan="1" rowspan="1">代码优化建议</td></tr><tr><td colspan="1" rowspan="2">格式转换</td><td colspan="1" rowspan="1">基于算子数据，若TransData算子耗时占比较高，具体请参见图5-49。</td><td colspan="1" rowspan="1">尝试禁用自动格式转换。torch.npu.config.allow_internal_format= false</td></tr><tr><td colspan="1" rowspan="1">变量x1为非连续性转换后的结果，在后续的每次调用都将引入transpose。def forward(self, x):X=self.fc1(x) x1=F.relu(x).transpose(1,2)#.contiguous()x2_1=self.fc2_1(x1)x2_2=self.fc2_2(x1)x3=torch.add(x2_1,x22)x4=self.fc3(x3)[:,0,]returnx4</td><td colspan="1" rowspan="1">消除调用产生的冗余Transpose，转换后，主动调用连续性转换函数。 x1 = F.relu(x).transpose(1,2).contiguous()</td></tr><tr><td colspan="1" rowspan="2">冗余代码</td><td colspan="1" rowspan="1">变量定义未使用，将会带来额外的内存操作开销。tasks = torch.tensor(tasks).to(self.device)# 定义后变量不使用</td><td colspan="1" rowspan="1">消除冗余代码。</td></tr><tr><td colspan="1" rowspan="1">小批量多次内存搬运导致大量的memory算子，可通过合并后搬运提升性能。tasks =torch.cat([self.task_tokenizer(x["task"]).to(self.device).unsqueeze(0) for x in batched_inputs], dim=0)</td><td colspan="1" rowspan="1">在CPU上完成操作后，统一搬运到NPU上运行。tasks =torch.cat([self.task_tokenizer(x["task"]).unsqueeze(0) for x inbatched_inputs],dim=0)tasks=tasks.to(self.device)</td></tr><tr><td colspan="1" rowspan="1">代码不亲和</td><td colspan="1" rowspan="1">算子在极端shape下，性能会发生较大的劣化，以SelectV2算子为例，具体请参见图5-50。fg_scores_mask = fg_mask[;, ;, None].repeat(1,1,self.num_classes)target_scores=torch.where(fg_scores_mask&gt;O,target_scores,0)</td><td colspan="1" rowspan="1">规避调用此算子，使用矩阵运算替换。fg_scores_mask =fg_mask.unsqueeze(-1)target_sores*=(fg_scores_mask&gt;0).float()</td></tr></table>

![](images/ebb0aa1c1d047977ad6eca2bf03c61d3f574de6d008f237632b309ce8e0d1b86.jpg)  
图 5-49 TransData 算子耗时占比高

![](images/7a4121a9a104d1131de1bf33281795302e2c9b7b1d802919dcb5514a050e353f.jpg)  
图 5-50 SelectV2 算子在极端 shape 下的性能劣化

# 5.4 Host Bound 问题定位及解决方法

# 5.4.1 Host Bound 问题分类

Host Bound 问题简介

在torch_npu训练和推理场景中，Host侧（CPU）的任务下发（如算子调度、内存分配）与Device侧（NPU）的任务执行是异步进行的。当Host侧任务下发耗时超过Device侧任务执行耗时，Device会因等待新任务而处于空闲状态，形成性能瓶颈，即Host Bound问题。

# Host Bound 问题表现

1. 算子下发连线呈现垂直密集状态，表明NPU在等待CPU下发任务。  
2. 因等待导致NPU闲置时间（Free time）占比过高。

![](images/b030851bca5b53e072aceb14e8adc1a90ef2d251558323bed50cf556f4ca1417.jpg)  
图 5-51 典型 Host Bound 场景性能数据

# Host Bound 问题优化方法

Host Bound问题主要由算子下发延迟、CPU计算负载过重两个因素导致，可采用表5-6方法进行优化。

表 5-6 Host Bound 常见优化手段  

<table><tr><td colspan="1" rowspan="1">优化方法</td><td colspan="1" rowspan="1">优势</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="2">算子下发优化</td><td colspan="1" rowspan="1">减少算子下发次数</td><td colspan="1" rowspan="1">通过5.2.2快慢卡问题定位方法识别瓶颈点后，采用逻辑优化、等价计算替换和算子融合等方法进行优化（参考5.3.2亲和算子优化策略）。</td></tr><tr><td colspan="1" rowspan="1">提升算子下发速度</td><td colspan="1" rowspan="1">·流水优化：是一种通用且高效的优化手段，其核心是将部分算子适配任务迁移至二级流水，从而均衡两级流水负载并降低任务出队唤醒耗时。·绑核优化：通过控制CPU端算子任务的处理器亲和性（即绑核），优化任务执行效率，避免跨NUMA（非统一内存访问架构）节点的内存访问，降低任务调度开销。编译优化：通过应用毕昇编译器（BiShengCompiler）的LTO（链接时优化）和PGO（配置文件引导优化）技术，对Python、PyTorch（torch）及其昇腾扩展（torch_npu/Ascend Extension forPyTorch）三个核心组件进行源码编译构建，可有效提升程序性能。</td></tr><tr><td colspan="1" rowspan="1">CPU计算优化</td><td colspan="1" rowspan="1">发挥异构算力优势</td><td colspan="1" rowspan="1">尽量减少AICPU算子的使用，优先选用亲和性更优的算子，可参考5.3.2亲和算子优化策略）。</td></tr><tr><td colspan="1" rowspan="1"></td><td colspan="1" rowspan="1">发挥并行计算优势</td><td colspan="1" rowspan="1">促进CPU与NPU的异步并行处理（例如，将数据处理逻辑置于DataLoader中），并减少流同步操作（如谨慎使用item(),cpu(),npu()等操作，尽量合并或避免使用）。</td></tr></table>

此外，存在一类属于下发异常问题，即算子下发过程因为资源抢占、操作系统调度策略冲突等因素导致耗时显著增加，参考下发异常分析。

# 5.4.2 流水优化

# 使能说明

通过设置以下环境变量启用该特性，通常适用于Host Bound问题严重的网络场景。

export TASK_QUEUE_ENABLE=2

# 详细原理

task_queue算子下发队列支持三个配置级别（以add算子为例），用户可以根据需要自行配置，详细介绍请参见《Ascend Extension for PyTorch 环境变量参考》的“TASK_QUEUE_ENABLE”章节。

# 注意事项

当ASCEND_LAUNCH_BLOCKING设置为"1"时，task_queue算子队列将被强制关闭，此时TASK_QUEUE_ENABLE配置失效，ASCEND_LAUNCH_BLOCKING的配置可参见《Ascend Extension for PyTorch 环境变量参考》的“ASCEND_LAUNCH_BLOCKING”章节。

配置 TASK_QUEUE_ENABLE=2 时，由于内存访问并发性增加，可能导致运行期间 NPU 内存峰值上升。

# 5.4.3 绑核优化

# 使能说明

通过设置以下环境变量启用该特性，适用于任务自主调度能力不足或快慢卡问题突出的网络场景。

export CPU_AFFINITY_CONF $= <$ <mode>,npu<value1>:<value2>-<value3>

<mode>：绑核模式。配置为 “0”或未配置时禁用绑核；配置为 “1” 启用粗粒度绑核；配置为 “2”启用细粒度绑核。  
npu<value1>:<value2>-<value3>：自定义绑核区间。当前仅在 <mode> 配置不为“0”时生效。<value1> 表示 NPU 卡编号，<value2>-<value3> 指定该卡进程的 CPU 核心绑定区间。

# 详细原理

● 粗粒度绑核：将单张NPU卡关联的所有线程绑定至指定CPU核心区间。细粒度绑核：将单张NPU卡关联的主要线程绑定至指定CPU核心区间，每个线程独占一个核心，实现相互隔离。

# 配置示例

步骤1 启用粗粒度绑核。export CPU_AFFINITY_CONF $^ { = 1 }$ 步骤2 启用细粒度绑核。export CPU_AFFINITY_CONF $\scriptstyle = 2$ 步骤3 启用自定义绑核。export CPU_AFFINITY_CONF $= 1$ ,npu0:0-1,npu1:2-5,npu3:6-6步骤4 执行上述配置后，NPU卡绑定详情如下所示。

● NPU 0 卡进程绑定至CPU核心 0-1。  
● NPU 1 卡进程绑定至CPU核心 2-5。  
● NPU 3 卡进程绑定至CPU核心 6。  
● 其他NPU卡进程使用默认核心区间。

----结束

# 5.4.4 编译优化

# 5.4.4.1 编译优化包获取

编译优化环境配置复杂，完整流程耗时较长，为提升部署效率，实现开箱即用的性能收益，用户可直接获取预构建的泛化编译优化包，包含Python编译优化后的压缩包，以及torch、torch_npu的whl安装包。Python压缩包可以直接配置软连接使用，torch、torch_npu的whl安装包则是基于典型场景的模型数据优化生成，具有一定泛化性能收益，软件包获取链接请参见表5-7。

表 5-7 vLLM 场景软件包获取  

<table><tr><td rowspan=1 colspan=1>名称</td><td rowspan=1 colspan=1>获取链接</td></tr><tr><td rowspan=1 colspan=1>Python压缩包</td><td rowspan=1 colspan=1> https://repo.oepkgs.net/ascend/pytorch/vllm/python/</td></tr><tr><td rowspan=1 colspan=1>torch、torch_npuwhl安装包</td><td rowspan=1 colspan=1>https://repo.oepkgs.net/ascend/pytorch/vllm/torch/</td></tr><tr><td rowspan=1 colspan=1>运行时依赖so</td><td rowspan=1 colspan=1>https://repo.oepkgs.net/ascend/pytorch/vllm/lib/</td></tr></table>

# 5.4.4.2 毕昇编译器安装

Python编译优化以及torch、torch_npu的编译优化均需预先安装毕昇编译器。

步骤1 请从鲲鹏社区官网获取该编译器的安装包。以4.1.0版本为例，单击获取链接下载。

步骤2 下载后执行以下安装及配置命令。

# 解压毕昇编译器安装包 tar -xvf BiShengCompiler-4.1.0-aarch64-linux.tar.gz

# 配置环境变量   
export PATH=\$(pwd)/BiShengCompiler-4.1.0-aarch64-linux/bin:\$PATH   
export LD_LIBRARY_PATH $\lvert = \$ 5$ (pwd)/BiShengCompiler-4.1.0-aarch64-linux/lib:\$LD_LIBRARY_PATH

步骤3 配置完成后，执行以下命令验证安装。

步骤4 若显示如下版本信息，则表明配置成功。

BiSheng Enterprise 4.1.0.B007 clang version 17.0.6 (ac00fd0f3aa8)   
Target: aarch64-unknown-linux-gnu   
Thread model: posix   
InstalledDir: /BiShengCompiler-4.1.0-aarch64-linux/bin   
Found candidate GCc installation: /usr/lib/gcc/aarch64-linux-gnu/7.3.0   
Selected Gcc installation: /usr/lib/gcc/aarch64-linux-gnu/7.3.0   
Candidate multilib:.;@m64   
Selected multilib: .;@m64   
Found CuDA installation: /usr/local/cuda， version 11.4

----结束

# 5.4.4.3 Python 编译优化

# 5.4.4.3.1 安装依赖

Python源码编译过程会尝试调用系统库。若编译时相关系统头文件缺失，虽可完成编译，但运行时调用到相应组件将触发错误。

# Fedora/RHEL/CentOS (dnf-based系统):

sudo dnf install gcc gcc-c++ gdb lzma glibc-devel libstdc $^ { + + }$ -devel openssl-devel \   
readline-devel zlib-devel libffi-devel bzip2-devel xz-devel \   
sqlite sqlite-devel sqlite-libs libuuid-devel gdbm-libs perf \   
expat expat-devel mpdecimal python3-pip

# ● Debian/Ubuntu (apt-based系统):

sudo apt-get install build-essential gdb lcov pkg-config \ libbz2-dev libffi-dev libgdbm-dev libgdbm-compat-dev liblzma-dev \ libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev \ lzma lzma-dev tk-dev uuid-dev zlib1g-dev libmpdec-dev

# 5.4.4.3.2 编译 Python

根据需求从以下地址下载对应版本的Python源码并解压：https://www.python.org/downloads/source/。

以Python 3.8.17为例，参考以下命令编译安装。

# 解压源码文件并进入目录 tar -xvf Python-3.8.17.tgz cd Python-3.8.17

# 配置编译环境（需预装毕昇编译器）  
export CC=clang  
export CXX=clang $^ { + + }$   
# 编译安装（<install_path>为Python安装目标绝对路径）  
mkdir -p <install_path>  
./configure --prefix=<install_path> --with-lto --enable-optimizations  
make -j  
make install

编译完成后，<install_path>/bin目录下将生成带有编译优化效果的python3及pip3可执行文件。

# 5.4.4.3.3 配置软链接

为了方便在环境中使能编译优化后的Python，可通过以下命令配置系统软链接。

# <default_path>为系统默认Python路径（可通过\`which python\`查询）cd <default_path>

# 备份原Python二进制文件mv python python_bak

# 创建新Python软链接ln -s <install_path>/bin/python3 python

按相同方式配置pip文件。配置完成后，建议检查pip文件首行Python路径是否正确。

vi pip

完成上述操作后，即可通过Python命令调用编译优化后的Python，并通过pip命令进行包管理。

# 5.4.4.3.4 注意事项

● 若模型运行时提示.so文件或模块缺失，需检查依赖是否安装完整。● 编译完成的Python环境可跨服务器迁移，但需注意仅支持从低版本glibc环境向高版本迁移，反向迁移不可行。

# 5.4.4.4 torch 及 torch_npu 编译优化

torch_npu编译依赖torch组件。需先完成torch编译优化，在编译环境中安装新编译的torch后，方可编译torch_npu。

# 说明

PGO技术需进行二次编译，首次编译生成插桩包，第二次编译生成优化包。

# 5.4.4.4.1 前置准备

# 下载torch源码，以2.1.0版本为例  
git clone -b v2.1.0 https://github.com/pytorch/pytorch.git pytorch-2.1.0  
cd pytorch-2.1.0  
git submodule sync  
git submodule update --init --recursive

# 安装torch依赖pip install -r requirements.txt

#下载torch_npu源码，要求和torch版本对应git clone -b v2.1.0 https://gitee.com/ascend/pytorch.git torch_npu需修改torch源码目录下的CMakeLists.txt文件，注释以下行以屏蔽告警（torch高版本已移除该行，无需注释）：

append_cxx_flag_if_supported("-Werror=cast-function-type" CMAKE_CXX_FLAGS)

![](images/35e7d5fe9a73ced9f0b6acf80b6c98f912d84383423bf9729686306de06cfe24.jpg)

# 5.4.4.4.2 首次编译（插桩编译）

# 配置环境变量（/path/to/profile为默认的性能数据存储路径，后续可通过设置LLVM_PROFILE_FILE修改）  
export CMAKE_C_FLAGS $| = "$ -flto=thin -fuse-ld=lld -fprofile-generate=/path/to/profile"  
export CMAKE_CXX_FLAGS="-flto $| =$ thin -fuse-ld=lld -fprofile-generate=/path/to/profile"  
export CC=clang  
export CXX=clang $^ { + + }$   
export USE_XNNPACK=0  
# 编译torch  
cd pytorch-2.1.0  
git clean -dfx  
python3 setup.py bdist_wheel  
# 编译torch_npu（需先安装新编译的torch）  
cd torch_npu  
git clean -dfx  
bash ci/build.sh --python $\scriptstyle 1 = 3 . 8$ --enable_lto --enable_pgo=1

# 5.4.4.4.3 采集 Profiling 数据

# 安装插桩包

pip install pytorch-2.1.0/dist/torch-\*.whl --force-reinstall --no-deps pip install torch_npu/dist/torch_npu-\*.whl --force-reinstall --no-deps

# 设置关键环境变量  
export OMP_PROC_BIND=false  
export LLVM_PROFILE_FILE=/tmp/profile/default_%m.profraw # 确保/tmp/profile目录为空

# 执行实际训练任务采集性能数据，例如: bash run_model.sh

# 5.4.4.4.4 二次编译（优化编译）

# 转换Profiling数据格式 llvm-profdata merge /tmp/profile -o default.profdata

# 配置优化编译环境  
export CMAKE_C_FLAGS $| = "$ -flto $| =$ thin -fuse- $\left. \lfloor \mathsf { d } = \right.$ lld -fprofile-use=/path/to/profile/default.profdata"  
export CMAKE_CXX_FLAGS="-flto $| =$ thin -fuse-ld=lld -fprofile-use=/path/to/profile/default.profdata"  
export $\complement { \mathsf { C } } =$ clang  
export CXX=clang $^ { + + }$   
export USE_XNNPACK=0  
# 编译优化版torch  
cd pytorch-2.1.0  
git clean -dfx  
python3 setup.py bdist_wheel  
# 编译优化版torch_npu（需复制default.profdata至torch_npu目录）  
cd torch_npu  
git clean -dfx

cp /path/to/profile/default.profdata .bash ci/build.sh --python $\scriptstyle 1 = 3 . 8$ --enable_lto --enable_pgo $^ { - 2 }$ 编译生成的torch及torch_npu即为高性能优化包。

# 5.4.5 下发异常分析

# 5.4.5.1 问题背景

当前，下发异常是导致快慢卡问题的常见原因之一，典型表现如下。

单张计算卡的特定算子执行耗时显著增加，从下图可以发现Dequeue@aclnnLogicalNot耗时明显长于左右部分的耗时（可通过MindStudioInsight的4.2.4.4.1 时间线（Timeline）观测）。

![](images/c410b7adad4c237d4baa46e1b94feeea886f72460016b05ef08a995233497505.jpg)  
图 5-54 单函数耗时异常增加

整体下发耗时延长，如图5-55所示，执行相同数量的算子，对比发现卡1的下发耗时明显更长。

![](images/ee9263989306c3fa848b0ca36eaa335fd134c958dd170c88ee9e1ffd4185d2b6.jpg)  
图 5-55 算子下发普遍变慢

此类问题通常因场景复杂而定位困难，可归类为下发异常问题。

# 5.4.5.2 前置检查

执行深入分析前，需完成以下基础检查：

额外进程检查：排查运行环境中是否存在影响CPU性能的后台进程或插件（通常由业务场景负责人确认，此因素较少成为主因）。  
任务均衡检查：通过Profiling工具分析各卡计算耗时。若各卡耗时接近且无明显快慢卡现象，可初步判定任务均衡（业务场景负责人可进一步确认）。如图5-56所示。

![](images/cf2584fd965339c09ae26ebf71b83076ec1d378cc09bbf8be28181529d2404cc.jpg)  
图 5-56 多卡计算任务相对均衡

绑核隔离检查（针对A+K场景）：在服务器调度能力有限（可能出现CPU核切换或抢占）的 $\mathsf { A } +$ K场景中，建议尝试绑核隔离任务。

具体操作：使用taskset命令，或设置环境变量export CPU_AFFINITY_CONF=1、export CPU_AFFINITY_CONF $^ { = 2 }$ 。

# 说明

环境变量CPU_AFFINITY_CONF的详细说明请参考《PyTorch 训练模型迁移调优指南》的“性能调优 $>$ 性能调优方法 $>$ 调度优化 $>$ 绑核优化”章节。

# 5.4.5.3 正式分析

# 5.4.5.3.1 CPU 运行状态

完成5.4.5.2 前置检查后，执行以下步骤检查CPU状态：

命令输入：在命令行执行top命令，实时查看系统CPU运行状态。

![](images/d6309f9acbef1dd1b82681d881aea8ff6983b13a63346b2805653094aad8c967.jpg)  
图 5-57 top 命令执行界面

进程线程查看： 如需查看特定进程（PID）的所有线程，执行：top -H -p<pid>。

![](images/13a3daab2afe6255f137a19c87a7281daa1a72c1710c360934d3a2d89939b5ad.jpg)  
图 5-58 top 查看指定进程

# 界面操作说明：

F键：进入字段选择模式

– 方向键↑/↓：浏览字段  
– D键：选定/取消显示项  
– 方向键 ：选中字段，通过方向键与其他字段交换排序位置方向键 ：退出排序操作

![](images/335307c255e4c6c928e7ca0ef7567fdc749625061788214011847200623dc89a.jpg)  
图 5-59 top 界面选择展示参数

# 结果分析与处理：

按F键并添加P（Last used CPU）字段后，可观察各进程最后运行的CPU核编号。  
若发现进程CPU核存在异常切换，可能原因包括：虚拟机或容器内CPU拓扑结构差异，或绑核范围过大。  
针对后者，可设置export CPU_AFFINITY_CONF=2，尝试细粒度的绑核优化。

![](images/86763bdc9f99021f1c39060d9c2472de755188520adfd30e7e40b5a9733655a7.jpg)  
图 5-60 显示各进程最后使用 CPU

# 5.4.5.3.2 异常堆栈捕获

当检测到性能异常时，需及时捕获并分析相关堆栈信息：

# gdb 查看

gdb -p <pid>  
#进入GDB命令行，打印进程/主线程调用栈  
bt  
#查看线程调用栈  
info threads  
thread<n>  
bt

# 5.5 集群性能异常波动问题定位方法论

在大规模集群场景，性能波动问题定位主要存在以下困难：

采集难：

若用户未开启动态性能数据采集功能，则需要停止训练任务，修改配置后进行采集，对用户环境影响较大。  
若问题现象为慢step偶然出现，为定位异常点，需要采集到慢step的性能数据，这一过程要求反复采集，在大规模集群下由于每次采集数据量大，这一过程尤其繁琐。

分析难：

由于数据量较大，全量节点直接将性能数据写入共享存储可能会造成采集结果膨胀过大，甚至可能会影响同一集群下其他任务，因此需要先将性能数据保存到本地后，再移动到共享存储。  
使用msprof-analyze等工具进行分析时耗时较长。  
出于安全考虑，往往存在着服务器无法联网，工具安装困难等问题，且部分局点性能数据不允许回传，必须在线分析，造成分析困难。

验证难：若需要在全量集群上进行实验、验证优化效果，同样涉及停止当前作业的需求，影响训练进度。且若反复进行全量集群实验，会极大程度影响性能。

本节主要聚焦集群性能异常波动问题，介绍大规模集群长稳训练时出现性能劣化的定位分析方法，对其中技巧进行总结，帮助提升定位效率。

说明：

本节主要关注长稳场景出现的性能问题，一般指作业在无性能问题稳定运行一段时间后，突然出现性能波动或在进行过某些非业务变更后（例如版本升级、集群重划分、硬件替换等），性能相较于变更前出现劣化。

# 5.5.1 集群性能劣化分析思路

整体上，集群性能问题分析思路可以参考3.3 排查思路介绍。

对于一般的长稳性能问题，即在正常训练过程中出现的性能波动，需要首先考虑近期变更，并进行硬件问题初步排查，再通过Profiling找到具体问题点进行详细定位。

为提高定位效率，建议采用粗-细两步定位法，如图5-61所示。

![](images/eeb546976c04c0fd0ff5f99e2f384b91908948b9cecdb814b061e2101cff23cf.jpg)  
图 5-61 定位流程

粗、细两步定位法的核心思路是考虑到大规模集群训练Profiling采集分析困难，先粗粒度，聚焦近期集群变动（如配置变更、组件升级）及硬件指标（CPU/内存利用率、网络吞吐量等），快速锁定异常时段与可疑模块；未发现明显异常时，采用细定位的方法，通过Profiling工具深度剖析计算、通信、下发等底层细节，进一步从线程栈、I/O延迟、锁竞争等维度定位瓶颈根因。这一方法论强调变动信息与指标的联动分析，并借助Profiling、火焰图、调用链等可视化手段，实现从宏观异常定位到微观根因分析的闭环，为集群性能优化提供精准的分层诊断路径。

另外，为解决验证难的问题，在条件允许的情况下，建议采用N分、业务单机模型测试等方法，在小规模上复现问题，方便验证。

# 1. 粗定位

粗定位一般是在无Profiling的场景下展开。粗定位的方法主要来源于过去问题的分析与经验。显著问题主要集中在以下四类：

资源累积：由于申请的资源没有及时释放，或异常申请，导致资源出现累积，影响性能。  
资源抢占：Host侧存在其他进程，或在未进行良好资源隔离的情况下，其他机器大压力读写共享存储等行为。

通信重传：由于丢包等问题产生通信重传，影响性能。

环境变更：近期进行了变更，包括但不限于升级、使用新集群、启动或停止某些服务。

针对以上问题，可以通过以下方法进行Profiling分析前的粗定位：

版本排查：收集当前使用版本，至少包括HDK、OS、CANN、框架，通过兼容性列表，确认是否版本上存在兼容性问题，同时确认是否存在版本已知性能相关问题。

变更排查：明确近期是否进行过变更，包括但不限于版本升级、集群重划分等。

环境排查：查看是否存在硬件告警、查看存储（io等）、网络（丢包等）等KPI是否正常。

配置排查：分别参考训练预检工具指南和推理预检工具指南中典型检查项是否配置正确。

粗定位的检查项来源于历史问题定位经验，可覆盖大多数常见性能波动问题。若存在粗定位无法处理的问题，则需要通过细定位详细排查。

另外，可以通过集群N分、单机模型测试等方法，最小化问题复现单元。若确认单机复现，则可以查看是否存在进程抢占、CPU利用率变化，messages或dmesg日志是否存在告警或报错等。

# 2. 细定位

细定位的过程，请参考3.3.2 详细排查，使用模型调优工具进一步定位异常点，详细分析。

将粗、细定位过程的结论相互印证，得到最终问题根因及解决方案。

# 5.5.2 定位实操技巧

1. 在大规模场景下，若以规定时间内完成训练任务为目标，可以先明确优化目标，计算定位的收益（需要对比为定位问题在生产集群上实验、采集所造成的时间损耗，与完成优化后带来的收益）。

2. 针对大规模集群问题定位，首先需要考虑将大规模问题在小规模集群、甚至是单机环境上进行复现，方便实验，从而减小对生产任务的影响。方式包括但不限于N分、单机测试、预检等方式。

3. 实践中初次采集一般采用L1不带堆栈。在大规模集群场景下若直接写共享存储，可能会导致采集膨胀过大。另外，如果未进行良好资源隔离，可能会影响集群中其他作业。因此可以考虑首先将Profiling写到本地，再通过脚本分批收集至共享存储。

4. 若条件允许，可以在模型执行训练时就启动动态性能数据采集功能。

# 5.6 ONNX 离线推理场景解决方案

昇腾产品以昇腾AI处理器的PCIe（Peripheral Component Interconnect Express）的工作模式进行区分，分为Ascend RC（Ascend Root Complex）和Ascend EP（AscendEndpoint），RC模式下PCIe工作在主模式，RC模式下仅支持离线推理方式。具体请查看《昇腾产品形态说明》。

# 5.6.1 信息收集

本文以Atlas 200I A2 加速模块为例，分析使用Atlas 200I A2推理场景通常需要收集如下信息。

# Profiling 信息收集

Profiling数据采集操作详情可参见《性能调优工具用户指南》的“msprof模型调优工具 $>$ 性能数据采集 $>$ msprof采集通用命令”章节。

步骤1 登录运行环境，进入msprof工具所在目录“/var”。

步骤2 执行如下命令，采集性能数据。其中application为用户程序。msprof --output={path} {application}

命令示例：msprof --output=\$[HOME]/profiling_output \$[HOME]/HIAI_PROJECTS/MyAppname/out/main步骤3 命令执行完成后，在--output指定的目录下生成PROF_XXX目录，目录结构如下。

![](images/e3a0ee01c9c1e7a01bf7cd04a13e4e8c62f9cb476cb996591ed747ce4aea08a5.jpg)

步骤4 将PROF_XXX目录上传到安装toolkit包的开发环境，执行以下命令进行数据解析。msprof --export=on --output=<dir>

PROF_XXX目录下会新增数据文件，目录结构如下：

device_{id} ─data ─host ─data ─... ─mindstudio_profiler_output ─xx_\*.csv ─xx_\*.json ... ...

----结束

# ONNX 通过 ATC 转换 OM 的日志收集

设置转换ONNX文件时的日志级别，并将日志的输出定向到文件，分析该问题，步骤如下：

步骤1 配置ATC运行环境变量。export ASCEND_SLOG_PRINT_TO_STDOUT $\dot { = } 1$ export ASCEND_GLOBAL_LOG_LEVEL $_ { = 0 }$

步骤2 ATC命令加上--log $| = |$ debug，收集转换日志。

命令示例：

atc --model=\$modelPath/\$onnxfile \  
--log=debug \  
--framework=5 \  
--input_shape="x:\$batchsize,3,\$height,\$width" \  
--input_fp16_nodes="x" \  
--output_type $\varprojlim$ FP16 \  
--op_select_implmode=high_precision \  
--output=\$outputPath/\$outname \  
--soc_version=Ascendxxxyy \ # xxxyy为用户实际使用的具体芯片类型

步骤3 重新执行ATC，将输出的信息重定向到一个日志文本文件连同ATC执行产生的fusion_result.jsonl一并获取，用于后续的性能分析。

----结束

# 收集推理日志

执行OM文件进行推理，收集推理输出的日志，采集步骤如下：

步骤1 配置OM模型运行环境变量。export ASCEND_SLOG_PRINT_TO_STDOUT=1export ASCEND_GLOBAL_LOG_LEVEL $^ { - 1 }$

步骤2 然后执行OM文件，将输出的信息重定向到一个日志文本文件。----结束

# 返回 ATC 转换前后的 ONNX 文件和 OM 文件

ONNX是业内目前比较主流的模型格式，广泛用于模型交流及部署，离线推理需要将ONNX文件转换为OM文件来进行推理。

需要将训练环境模型运行导出的ONNX文件和转换产生的OM文件一并采集，用于后续分析。

# 5.6.2 问题分析

本文以Atlas 200I A2 加速模块为例，分析Atlas 200I A2推理场景的采集到的相关信息：

# 5.6.2.1 分析性能数据文件

步骤1 将收集到的性能数据文件导入至MindStudio Insight工具中进行分析。

步骤2 分析Free占比，通常情况Atlas 200I A2 加速模块的Free占比应该比较小(<10%)。如图5-62所示，Free占比超过30%，明显存在异常。需要进一步分析该芯片的OS是否运行了其它业务导致资源占用，进而导致出现等待。

![](images/ba7577f2793a3a60bc3fc1795b1e4b5ae4c709c74fceb2b841a1679213ada9bd.jpg)  
图 5-62 分析 Free 占比

步骤3 分析运行在AI CPU的算子。如图5-63所示，GridSampler2D运行在AI CPU上。找到问题算子并联系相关责任人，判断该算子是否可以优化为AI Core上运行或进行下一步分析。

![](images/41eb0daf85c5769050b350e6aa56bbbd3d8723e8af875a50c73395014bf6e233.jpg)  
图 5-63 分析运行在 AI CPU 的算子

步骤4 分析运行在AI Core，但是耗时长的算子。如图5-64所示，Conv2D算子，占据大部分耗时。找到问题算子并联系相关责任人，查看该算子是否可以进行下一步优化。

![](images/01fdd6cbc6a848539f1a61e05b2531c4c84eb3c72f10cd406b60514b7ed8e066.jpg)  
图 5-64 分析运行在 AI Core 且耗时长的算子

步骤5 op_summary文件分析。

建议以Task Duration降序排序，优先关注耗时较长的算子情况，此类算子为性能瓶颈算子。如果vec_ratio和mac_ratio均没有超过0.8，则说明算子仍有优化空间；如果

mtex_ratio较高，则说明数据搬运耗时较高，可考虑与前后算子融合以减少搬运，参数说明如表5-8所示。

表5-8 参数说明  

<table><tr><td rowspan=1 colspan=1>参数</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>aic_mte1_time(us)</td><td rowspan=1 colspan=1>代表MTE1类型指令（L1-&gt;LOA/LOB搬运类指令）耗时，不包括搬运等待时间。</td></tr><tr><td rowspan=1 colspan=1>aic_mte1_ratio</td><td rowspan=1 colspan=1>代表MTE1类型指令（L1-&gt;L0A/L0B搬运类指令）的cycle数在total cycle数中的占用比。</td></tr><tr><td rowspan=1 colspan=1>aic_mte2_time(us)</td><td rowspan=1 colspan=1>代表MTE2类型指令（GM-&gt;AICORE搬运类指令）耗时。</td></tr><tr><td rowspan=1 colspan=1>aic_mte2_ratio</td><td rowspan=1 colspan=1>代表MTE2类型指令（GM-&gt;AICORE搬运类指令）的cycle数在total cycle数中的占用比。</td></tr><tr><td rowspan=1 colspan=1>aic_mte3_time(us)</td><td rowspan=1 colspan=1>代表MTE3类型指令（AICORE-&gt;GM搬运类指令）耗时。</td></tr><tr><td rowspan=1 colspan=1>aic_mte3_ratio</td><td rowspan=1 colspan=1>代表MTE3类型指令（AICORE-&gt;GM搬运类指令）的cycle数在total cycle数中的占用比。</td></tr></table>

----结束

# 5.6.2.2 ATC 转换日志分析

检查耗时长的算子是否在日志中存在有未命中高性能知识库的记录：

搜索关键字1：does not hit the high-priority operator information library。

# 日志示例：

INFO:root: 2025-02-10-12:52:23.694.284 Op[/backbone/stages.2/blocks.14/attn/Div_4] does not hit the high-priority operator information library, which might result in compromised performance. INFO:root: 2025-02-10-12:52:23.709.410 Op[/backbone/stages.2/blocks.15/attn/Div_1] does not hit the high-priority operator information library, which might result in compromised performance.

搜索关键字2：from cost_model

日志示例：

[DEBUG] TBE(41403,python3):2025-03-20-15:08:44.009.430 [get_tiling_cube.py:147] [get_auto_tiling_v2] [auto tiling] tiling is from cost model tiling, kernel name is :"te_fused_op_conv2d_fix_pipe_d637645277a21bcd6e83a554eeadce4e230fd724a51d46ed7c1ff600f7c ddfc8_0"

# 5.6.2.3 OM 和 ONNX 文件

可以使用netron工具加载OM和ONNX查看模型结构，可以清晰的看到ONNX模型中算子的拓扑结构，输入和输出，方便后续改图。加载ONNX或者OM文件的示例如图5-65所示。

![](images/74485729efecc2a84a5870e5f37afc5f4174b23c03e3f444a3409abac30a3c9b.jpg)  
图 5-65 算子拓扑结构

# 5.6.3 优化方法

由于在NPU上实际运行的是OM文件，torch_npu之类的优化无法使用，性能优化可分为通用优化和改图优化。

通用优化对大部分模型都适用，改图优化比较有针对性，不同模型改图策略不同。通用优化方法包括对模型进行压缩量化、通过ONNX Simplifier等工具对模型进行自动优化、手动修改ONNX文件等。

# 5.6.3.1 模型压缩量化

量化可以使模型压缩，减少计算量。

# 说明

● 昇腾仅支持对Cube算子（MatMul、Conv）的量化。● 由于量化会插入一些数据转换算子，可能会导致性能劣化，如果需要量化，建议量化后使用AOE等手段进行优化，对比量化前后的性能。AOE方法参考5.6.3.2-ONNX模型调优章节。

量化方法包括以下几种：

通过ATC进行量化：进行ATC转换时使用--compression_optimize_conf参数，直接得到量化后的OM文件，使用方法详见《ATC离线模型编译工具用户指南》的“参数说明”章节。  
AMCT_ONNX：针对ONNX进行量化，需下载并安装“AMCT（ONNX）”，相当于ATC参数量化的ONNX版本。AMCT工具在CANN软件下载链接中获取，AMCT支持联合量化，在resnet结构上可能会有额外的性能提升。  
msModelSlim工具：针对ONNX进行量化，CANN包自带工具，无需安装，支持超2G的ONNX模型量化， 使用指导请参考msModelSlim工具。

# 5.6.3.2 ONNX 模型调优

# 5.6.3.2.1 简化 ONNX 文件

ONNX Simplifier是一款开源工具，可以简化ONNX模型。通过推断整个计算图，用常量输出替换冗余运算符（也称为常量折叠）。

执行以下命令，使用ONNX Simplifier工具。

pip install onnx-simplifier   
onnxsim -h #查看参数说明   
onnxsim --overwrite-input-shape $= ^ { 1 }$ "1,3,224,24" efficient.onnx efficient_sim.onnx

如图5-66所示，对导出的ONNX文件进行onnxsim后可以看到，减少了ONNX部分的操作。

图 5-66 使用 ONNX Simplifier 工具  

<table><tr><td colspan="3">Simplifying Finish! Here is the difference:</td></tr><tr><td></td><td>Original Model</td><td>Simplified Model</td></tr><tr><td>Add Constant Conv DepthToSpace Di Gather Mul Pow ReduceMean Shape Slice Sqrt Sub Model Size</td><td>128 517 85 5 64 32 160 32 64 32 64 32 64 27.8MiB</td><td>96 271 85 5 32 0 96 32 64 0 64 32 32 26.3MiB</td></tr></table>

# 5.6.3.2.2 ATC 调优

# ATC工具运行命令参考如下：例：

atc --framework ${ : = } 5$ \   
--model=models/vit-b-16.img.fp32.bs\${bs}.opt.onnx \   
--output=models/vit-b-16.img.bs\${bs} \   
--input_format=NCHW \   
--input_shape $: = ^ { 1 }$ "image:\${bs},3,224,224" \   
--soc_version=Ascend\${chip_name} \   
--log=error \   
--optypelist_for_implmode $\mathrel { \mathop : } \mathbf { l }$ "Sigmoid" \   
--op_select_implmode=high_performance \   
--enable_small_channel \   
--insert_op_conf=./insert_op.cfg

命令中的参数说明如表5-9所示，更多参数说明可参考《ATC离线模型编译工具用户指南》。

表 5-9 参数说明  

<table><tr><td rowspan=1 colspan=1>参数</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>--model</td><td rowspan=1 colspan=1>ONNX模型文件。</td></tr><tr><td rowspan=1 colspan=1>--framework</td><td rowspan=1 colspan=1>5代表ONNX模型。</td></tr><tr><td rowspan=1 colspan=1>--output</td><td rowspan=1 colspan=1>输出的OM模型。</td></tr><tr><td rowspan=1 colspan=1>--input_format</td><td rowspan=1 colspan=1>输入数据的格式。</td></tr><tr><td rowspan=1 colspan=1>--input_shape</td><td rowspan=1 colspan=1>输入数据的Shape。</td></tr><tr><td rowspan=1 colspan=1>--log</td><td rowspan=1 colspan=1>日志级别。</td></tr><tr><td rowspan=1 colspan=1>--soc_version</td><td rowspan=1 colspan=1>处理器型号。</td></tr><tr><td rowspan=1 colspan=1>--optypelist_for_implmode</td><td rowspan=1 colspan=1>指定算子。</td></tr><tr><td rowspan=1 colspan=1>--op_select_implmode</td><td rowspan=1 colspan=1>选择高性能/高精度模式，与--optypelist_for_implmode配合使用。</td></tr><tr><td rowspan=1 colspan=1>enable_small_channel</td><td rowspan=1 colspan=1>与--insert_op_conf配合使用。</td></tr><tr><td rowspan=1 colspan=1>--fusion_switch_file</td><td rowspan=1 colspan=1>关闭/打开部分融合规则。</td></tr></table>

# 5.6.3.2.3 AOE 调优

对于EP模式的产品，可以直接在OM模型运行的环境进行AOE调优。参考命令如下：aoe --framework 5 --model ./model.onnx --output model --job_type 2 --ip xx.xx.xx.xx --aicore_num=1参数的详细解释以及使用方法可参见《AOE调优工具用户指南》。

# 5.6.3.2.4 NCS 调优

对于部分运行在RC模式的产品，产品普遍存在内存较小的特点。这就导致了模型无法通过本地AOE进行调优，这样就需要用到另一种调优方式，NCS调优。

# 安装 CANN 包

# 环境准备

一台常规的Linux服务器（后文描述为Host机）和一台装有昇腾NPU的服务器（后文描述为Device机）。服务器上需要安装toolkit包，以CANN 8.0.0版本为例，toolkit包的安装方法参考对应版本的CANN安装指南。

![](images/8da25ca39ce6c14501f2aa1063b9d3b2199ee3e881bc81edf9907aa2f7bd279b.jpg)  
图 5-67 环境准备

配置环境变量

Host机  
加载CANN安装路径的环境变量脚本set_env.sh, CANN安装路径以\$  
{install_path为例。  
source \${install path}/ascend-toolkit/set_env.sh  
Device机  
export LD_LIBRARY_PATH $\scriptstyle 1 = \mathfrak { S }$ \${install_path}/latest/tools/ncs/lib64/:\${install_path}/latest/runtime/lib64/:\$LD_LIBRARY_PATH  
exportPATH=\${install_path}/latest/tools/ncs/bin/:\$PATH  
其中install_path请配置为CANN软件的实际安装路径。

# 配置密钥证书

请先确认Host机和Device机处于同一主网段，可以互相ping通。

步骤1 在Host机中上传shell脚本，脚本如下：

DEVICE_IP=10.x.x.196 # your device ip   
HOST_IP=10.x.x.66 # your host IP   
PASS_PHRASE $=$ Ncx12345 # yourpassphrase   
KEY_LEN $\mathtt { 1 = 3 0 7 2 }$ # [3072,4096]   
VALID_DAYS $\mathtt  \mathtt  \mathtt  \mathtt { \mathtt { \mathtt { \mathtt { \mathtt \mathtt { \mathtt { \mathtt \mathtt { \mathtt \mathtt { \mathtt \mathtt { \mathtt \mathtt { \mathtt \mathtt { \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt \mathtt \mathtt { \mathtt \mathtt \mathtt \mathtt \mathtt \mathtt \Lambda } } } } } } } } } } } } } } } } } } } } } } } } } = 3 6 5$ # thecertwillexpireafterthevaliddays   
COUNTRY ${ \tt = } { \sf C N }$ # yourcountrynameabbr.(2lettercode)   
STATE $\fallingdotseq$ Zhejiang # yourprovincename   
LOCATION=Hangzhou # yourcityname   
ORGANIZATION=ABC # yourcompanyname   
ORGANIZATION_UNIT $\mathop { \bf { \tilde { \theta } } } =$ DEF # yoursectionname   
COMMON_NAME_ROOT=www.aoe.com # yourdomainname   
ENCRYPT_MODE $=$ aes256 # [aes256,aes128]

##########该分割线以上内容请根据实际情况修改，以下内容不建议修改。##########

# generateconf   
rm-rfhost-ext.cnfdevice-ext.cnf   
echo"[ext]">>host-ext.cnf   
echo"subjectAltName=IP:\${HOST_IP}">>host-ext.cnf   
echo"[ext]">>device-ext.cnf   
echo"subjectAltName $\lvert = \rvert$ P:\${DEVICE_IP}">>device-ext.cnf

# generaterootcert opensslreq-x509-newkeyrsa:\${KEY_LEN}-days\${VALID_DAYS}-nodes-keyoutca-key.pem-outca-cert.pemsubj"/C=\${COUNTRY}/ST $\mathbf { \Phi } = \mathbf { \Phi }$ \${STATE}/ $L = \$ 1$ {LOCATION} $\scriptstyle 1 0 = \oint$ {ORGANIZATION} $\textstyle 1 0 0 = S$ {ORGANIZATION_UNIT}/CN=\${COMMON_NAME_ROOT}"-addextkeyUsage=keyCertSign

# generatedevicecertrequest   
opensslreq-newkeyrsa:\${KEY_LEN}-nodes-keyoutdevice-key.pem-outdevice-cert.csr-subj" $1 C = S$   
{COUNTRY}/ST=\${STATE}/L $\cdot = \$ 5$ {LOCATION}/ $O = \$ 4$ {ORGANIZATION}/OU $\mathbf { \Phi } = \mathbf { \Phi }$ \${ORGANIZATION_UNIT}/CN=NCS"   
# generatedevicecert   
opensslx509-req-indevice-cert.csr-days\${VALID_DAYS}-CAca-cert.pem-CAkeyca-key.pem-CAcreateserial  
outdevice-cert.pem-extensionsext-extfiledevice-ext.cnf   
# generatehostcertrequest   
opensslreq-newkeyrsa:\${KEY_LEN}-nodes-keyouthost-key.pem-outhost-cert.csr-subj"/ $C = \$ 5$ {COUNTRY}/ST=\$   
{STATE}/ $\therefore = \$ 1$ {LOCATION}/ $\scriptstyle 1 0 = \$ 4$ {ORGANIZATION}/OU $\scriptstyle = \mathfrak { s }$ \${ORGANIZATION_UNIT}/CN=NCA"   
# generatehostcert   
opensslx509-req-inhost-cert.csr-days\${VALID_DAYS}-CAca-cert.pem-CAkeyca-key.pem-CAcreateserial  
outhost-cert.pem-extensionsext-extfilehost-ext.cnf

# encrytprivatekey opensslrsa-inhost-key.pem-passoutpass:\${PASS_PHRASE}-\${ENCRYPT_MODE}-outhost-key.pem opensslrsa-indevice-key.pem-passoutpass:\${PASS_PHRASE}-\${ENCRYPT_MODE}-outdevice-key.pem 步骤2 将Device Ip和Host Ip改成对应的ip地址，在Host机上运行该shell脚本，生成密钥证书 文件。

表 5-10 相关文件说明  

<table><tr><td rowspan=1 colspan=1>文件名</td><td rowspan=1 colspan=1>作用</td></tr><tr><td rowspan=1 colspan=1>ca-cert.pem</td><td rowspan=1 colspan=1>根CA，需要拷贝到开发环境和运行环境上。</td></tr><tr><td rowspan=1 colspan=1>host-key.pem</td><td rowspan=1 colspan=1>开发环境端私钥，需要拷贝到开发环境上。</td></tr><tr><td rowspan=1 colspan=1>host-cert.pem</td><td rowspan=1 colspan=1>开发环境端证书，需要拷贝到开发环境上。</td></tr><tr><td rowspan=1 colspan=1>device-key.pem</td><td rowspan=1 colspan=1>运行环境端私钥，需要拷贝到运行环境上。</td></tr><tr><td rowspan=1 colspan=1>device-cert.pem</td><td rowspan=1 colspan=1>运行环境端证书，需要拷贝到运行环境上。</td></tr><tr><td rowspan=1 colspan=1>ca-key.pem</td><td rowspan=1 colspan=1>中间过程文件，可忽略。</td></tr><tr><td rowspan=1 colspan=1>ca-cert.srl</td><td rowspan=1 colspan=1>中间过程文件，可忽略。</td></tr><tr><td rowspan=1 colspan=1>host-cert.csr</td><td rowspan=1 colspan=1>中间过程文件，可忽略。</td></tr><tr><td rowspan=1 colspan=1>device-cert.csr</td><td rowspan=1 colspan=1>中间过程文件，可忽略。</td></tr><tr><td rowspan=1 colspan=1>host-ext.cnf</td><td rowspan=1 colspan=1>中间过程文件，可忽略。</td></tr><tr><td rowspan=1 colspan=1>device-ext.cnf</td><td rowspan=1 colspan=1>中间过程文件，可忽略。</td></tr></table>

步骤3 将device-key.pem 、device-cert.pem和ca-cert.pem三个文件拷贝到Device机上，在密钥证书所在路径执行如下命令。

Host机

akt --private_key host-key.pem --public_cert host-cert.pem --ca_cert ca-cert.pem

Device机 akt --private_key device-key.pem --public_cert device-cert.pem --ca_cert ca-cert.pem

# 说明

示例命令中的私钥文件名为：host-key.pem或者device-key.pem，设备证书文件名为：host-cert.pem或者device-cert.pem，根证书文件名为：ca-cert.pem。

步骤4 执行命令后，提示“Enter Password:”，请输入加密私钥的口令PASS_PHRASE（口令必须与生成私钥时的口令一致）。

回显如下，表示证书导入成功。  
Load cert, password, and key successfully.

# ----结束

# 执行调优

步骤1 AOE调优前，在Host机上，使用如下命令可选配置部分环境。export TUNE_BANK_PATH={bank_path} # 指定知识库保存地址export TE_PARALLEL_COMPILER $^ { \circ 3 2 }$ # 加速aoe调优

步骤2 执行以下命令，使用AOE调优。

Device机侧：

ncs &

执行后查询NCS是否启动成功。 ps -ef|grep ncs|grep -v "grep"

回显信息如下所示，代表NCS服务启动成功。ID为257435的进程是ncs守护进程，ID为257440的进程是ncs运行进程。

root257440257435007:54pts/300:00:01ncs--ipXX.XX.XX.XX--portXXXX--daemonfalse

Host机侧：

aoe --framework 5 --model ./model.onnx --output model --job_type 2 --ip xx.xx.xx.xx --aicore_num $^ { 1 = 1 }$

命令中的参数说明如表5-11所示。更多调优参数可以参考《AOE调优工具用户指南》。

表 5-11 参数说明  

<table><tr><td rowspan=1 colspan=1>参数</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>--model</td><td rowspan=1 colspan=1>需要调优的模型。</td></tr><tr><td rowspan=1 colspan=1>--output</td><td rowspan=1 colspan=1>调优完成的模型的保存名字。</td></tr><tr><td rowspan=1 colspan=1>--job_type</td><td rowspan=1 colspan=1>子图调优或者算子调优，分别是2和1，需要先进行子图调优，再进行算子调优，先后执行一遍。</td></tr><tr><td rowspan=1 colspan=1>--ip</td><td rowspan=1 colspan=1>Device机的Ip地址。</td></tr><tr><td rowspan=1 colspan=1>--aicore_num</td><td rowspan=1 colspan=1>Al Core数量。</td></tr></table>

步骤3 命令执行后，会有打印提示性能优化的比例，如图5-68所示，性能优化提升 $5 3 \%$ 。

![](images/71fd3a33ba5a97a85f94c866f5ae0bfb405539acc6d5f57939e090e0beee389e.jpg)  
图 5-68 调优回显

# ----结束

# 5.6.3.3 msit debug surgeon 自动优化 ONNX

surgeon（自动调优）工具使能ONNX模型在昇腾芯片的优化，并提供基于ONNX的改图功能。

# 使用方法

使用如下命令，进行调优，参数说明如表5-12所示。

msit debug surgeon COMMAND

表 5-12 参数说明  

<table><tr><td>参数</td><td>说明</td></tr><tr><td rowspan="5">COMMAND</td><td>COMMAND为surgeon工具提供的五个选项：</td></tr><tr><td>·list：列举当前支持自动调优的所有知识库。</td></tr><tr><td>evaluate：搜索可以被指定知识库优化的ONNX模型。 .</td></tr><tr><td>optimize：使用指定的知识库来优化指定的ONNX模型。 .</td></tr><tr><td>： extract：对模型进行子图切分。 . concatenate：对模型进行拼接。</td></tr></table>

# 说明

每个子任务下面的可选项和必选项不同。具体使用方法参考msit debug surgeon功能使用指南。

# 优化实例

COMMAND参数取值的实例图5-69和图5-70所示。

![](images/58ee06820ee8c295df510373b95a0c5b1e99a8fc8fd984a88c5ad1c7bd534523.jpg)  
图 5-69 取值为 list

# 图 5-70 取值为 evaluate 和 optimize

![](images/513acc8f1c146629632f62d49f3b1483397b2bfdbd9f007b6cd48890a6c3fc4d.jpg)

执行前后对比图5-71所示，可以看出执行后cast算子被消除了。

![](images/a76d0c453121b386516a5352da2b52c438605e809cad40d8e1e57c418434ab46.jpg)  
图 5-71 执行前后对比

![](images/05a67bb6216b72dca9254f40292ddf225ce51117576a37751b97dceed19b5d40.jpg)

# 5.6.3.4 升级 CANN 版本

CANN版本每一次升级都会优化部分算子。如果发现部分算子的执行性能差，或者执行在AI CPU上，考虑升级CANN版本。需要注意toolkit和kernel都要升级。通常情况下，升级CANN版本不会带来负面影响。

下面通过GridSampler2D算子的两个案例来直观感受解决方案，如图5-72所示，该算子在AI CPU上执行，性能较差；如图5-73所示，该算子虽然在vector_core上，但是执行效率也极低，这个案例的CANN版本比较旧。

<table><tr><td colspan="8">算子详情</td></tr><tr><td>类型</td><td>加速器核 数量</td><td></td><td>总耗（us)</td><td>← 平均耗时(μs)</td><td>最大耗时(μs)</td><td>最小耗时（us）</td><td>详情</td></tr><tr><td>BatchMatMulNd</td><td>AI_CORE</td><td>240</td><td>1372211.54</td><td>5717.55</td><td>23744.24</td><td>38.14</td><td>查看更多∨</td></tr><tr><td>SliceAiCore</td><td>AI_VECTOR_CORE</td><td>162540</td><td>136599.76</td><td>8.4</td><td>156.16</td><td>1.76</td><td>查看更多∨</td></tr><tr><td>GridSampler2D</td><td>AI_CPU</td><td>80</td><td>868262</td><td>10853.28</td><td>26866.4</td><td>3506.06</td><td>查看更多∨</td></tr><tr><td>MulAiCore</td><td>AI_VECTOR_CORE</td><td>19294</td><td>762680.34</td><td>39.53</td><td>1584.7</td><td>1.1</td><td>查看更多∨</td></tr><tr><td>GridSampler2DGrad</td><td>AI_CPU</td><td>18</td><td>531736.58</td><td>29540.92</td><td>30617.06</td><td>28956</td><td>查看更多∨</td></tr><tr><td>TransData</td><td>AI_VECTOR_CORE</td><td>32772</td><td>492363</td><td>15.02</td><td>481.42</td><td>1.64</td><td>查看更多</td></tr><tr><td>RealDivAiCore</td><td>AI_VECTOR_CORE</td><td>3690</td><td>380702.3</td><td>103.17</td><td>858.9</td><td>1.1</td><td>查看更多∨</td></tr><tr><td>Conv2DBackproplnp...</td><td>AI_CORE</td><td>432</td><td>334220.14</td><td>773.66</td><td>7206.24</td><td>11.32</td><td>查看更多∨</td></tr><tr><td>Max2AiCore</td><td>AI_VECTOR_CORE</td><td>540</td><td>325056.44</td><td>601.96</td><td>6149.84</td><td>2.94</td><td>查看更多∨</td></tr><tr><td>AddAiCore</td><td>AI_VECTOR_CORE</td><td>15278</td><td>321979.82</td><td>21.07</td><td>1399.84</td><td>1.16</td><td>查看更多 V</td></tr><tr><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td></tr></table>

![](images/7cacdf7d692845a23c5315b58a076e48201aa78a574b473764b8a53f457c0727.jpg)  
图 5-72 算子在 AI CPU 上执行  
图 5-73 算子在 vector_core 上执行

![](images/5737ac7d3ad5c5a48a92ed594df0d5c13cf287bc701afdbfe9cf5a4a09c64991.jpg)

针对以上两个问题，可以判断出在当前的CANN版本中，该算子在AI CPU上，存在性能问题，解决方案为将版本升级至CANN 8.0.RC3及以上版本，升级后改善明显，GridSampler2D耗时几乎可忽略。

# 5.6.3.5 改图优化

如果自动优化无法满足目标，也已经识别到模型中存在一些冗余计算，或者需要通过调整数据类型使其从AI CPU运行切换到AI Core运行等场景时，可以通过改图的方式来进行性能优化。

以在GlobalAveragePool操作前加入Cast操作为例，改图脚本如下：

import onnx   
from onnx import helper, TensorProto   
from onnx import shape_inference   
model_path $=$ "D:\\035-Code\\om_test\\resnet50.onnx"   
model $=$ onnx.load(model_path)   
def create_cast_node(input_name, output_name, to_type): return helper.make_node( 'Cast', inputs=[input_name], outputs [output_name], to=to_type )   
for i, node in enumerate(model.graph.node): if node.op_type $= =$ 'GlobalAveragePool': # 获取GlobalAveragePool节点的输入 input_name $=$ node.input[0]

# 生成新的Cast节点的输出名称cast_output_name $=$ f"{input_name}_cast"# 创建Cast节点cast_node $=$ create_cast_node(input_name, cast_output_name, TensorProto.FLOAT)# 更新GlobalAveragePool节点的输入node.input[0] $=$ cast_output_name# 将Cast节点插入到计算图中model.graph.node.insert(i, cast_node)break # 只处理第一个找到的GlobalAveragePool节点output_model_path $=$ "D:\\035-Code\\om_test\\resnet50_new.onnx" # 替换为你想要保存的模型路径onnx.save(model, output_model_path)

脚本执行后，效果如图5-74所示。

![](images/aec95f23f8224cd04573cda6ec2a4457887cba08eb2ff717a4af112cf0ae279e.jpg)  
图 5-74 执行后效果

# 5.7 MindIE 推理性能解决方案

# 5.7.1 整体思路

MindIE推理性能可以从纯模型和服务化两个角度进行优化。

首先确认当前纯模型的性能是否有调优空间：

1. 版本基线覆盖的场景，和版本基线的性能比较，检查配置。  
2. 版本基线未覆盖，或检查配置后仍存在问题的场景，先进行相同输入输出的纯模型测试。  
3. 如果纯模型测试结果未达到预期，进行纯模型性能调优；如果达到预期，则进行服务化性能调优。  
4. 服务化调优时性能瓶颈定位思路如图5-75所示。

![](images/03825af21f0bf2c62f3875d711787fa1dc22632910485c89c858065573f88f04.jpg)  
图 5-75 服务化性能瓶颈定位流程图

# 5.7.2 纯模型性能调优

常见的模型和使用场景下，如果采用了相同的权重、镜像、部署策略，理论上可以复现相似的性能。

因此出现性能问题后，可以优先对环境变量、涉及性能的开关进行检查，大概率问题在此处（例如没有绑核、内核版本低、日志等级等问题）。

纯模型性能测试可使用AISBench工具的benchmark-mindie评测插件评估。

如果纯模型测试结果已经出现性能劣化现象，可以进行详细的Profiling性能分析 ，分析方法见4.2.4 模型调优深入分析（MindStudio Insight）

# 纯模型初步测试

纯模型测试根据用户输入的--batch_size参数确定测试请求的大小，根据--case_pair确定请求输入长度和输出长度的组合，在推理时先处理所有请求的Prefill，再将所有请求组成同一个Decode Batch完成Decode推理。对于基线未覆盖的测试场景，特别是有TTFT或TPOT时延要求的场景，可以参考如下步骤进行纯模型测试。

步骤1 设置”--case_pair [[输入长度,1]]”, 即可模拟单个Prefill Batch的纯模型测试，此时指定的--batch_size参数记录为prefill_batchsize，测试结果中的”Total Time(s)”记录为prefill_time。

步骤2 按照实际输入输出长度设置--case_pair参数正常进行测试，此时指定的--batch_size参数记录为decode_batchsize，测试结果中的”Non-first token time(ms)”记录为decode_token_time。

步骤3 调整decode_batchsize和prefill_batchsize，找到满足要求的配置，如表5-13所示。

表 5-13 配置说明  

<table><tr><td rowspan=1 colspan=2>场景</td><td rowspan=1 colspan=1>配置方法</td></tr><tr><td rowspan=1 colspan=2>不限时延基线吞吐</td><td rowspan=1 colspan=1>调大decode_batchsize至OOM。（或选取增速放缓的decode_batchsize）</td></tr><tr><td rowspan=1 colspan=2>限制TPOT，不限制TTFT</td><td rowspan=1 colspan=1>调整decode_batchsize至decode_token_time刚好满足TPOT限制。</td></tr><tr><td rowspan=1 colspan=1>限制TPOT和TTFT</td><td rowspan=1 colspan=1>爬坡测试（限制Request Rate）</td><td rowspan=1 colspan=1>调整decode_batchsize至decode_token_time刚好满足TPOT限制，调整prefill_batchsize令prefill_time接近TTFT限制。</td></tr><tr><td rowspan=2 colspan=1>限制TPOT和TTFT</td><td rowspan=2 colspan=1>固定并发（不限制Request Rate）</td><td rowspan=1 colspan=1>调整decode_batchsize至decode_token_time刚好满足TPOT限制，设置prefill_batchsize=dp,调小decode_batchsize直至decode_batchsize prefill_time</td></tr><tr><td rowspan=1 colspan=1>prefill_batchsize            2接近平均TTFT限制。</td></tr></table>

步骤4 如需纯模型性能调优（并行策略，环境变量等），对不同的配置重复步骤1直到选取最优配置。

步骤5 将纯模型的Decode Batchsize和Prefill Batchsize转化为服务化参数maxBatchsize和maxPrefillBatchSize。

步骤6 设置其他服务化和测试参数，根据实际测试结果调整最大并发Requestrate/maxBatchSize/maxPrefillBatchsize。----结束

# 5.7.3 服务化性能调优方法论

# 5.7.3.1 明确优化目标

用户通常关注首Token时延、吞吐等性能指标。测试时的并发、输入输出长度对性能影响较大，必要时可以进行引导。

低时延场景：实时交互（如对话系统），关注首Token生成速度（Prefill阶段优化）。  
高吞吐场景：离线批量处理（如文档生成），关注Tokens/秒（Decode阶段优化）。

# 说明

有时性能目标和实际差异较大，可以根据纯模型性能简单评估性能上限，判断时延或吞吐的极限，从而判断目标是否有可能达成。

# 服务化性能上限评估

按照估算场景和目标划分，基于纯模型测试结果，对于服务化性能上限的几种（乐观）估计方法如下：

单并发：与纯模型单并发性能接近。

大并发下平均首Token时延。

Prefill阶段推理是算力密集型场景，Prefill Batchsize增大后会触发算力瓶  
颈；触发瓶颈后，Prefill时延会随Batchsize增加接近线性增长。  
使用纯模型测试找到达到算力瓶颈的Prefill Batchsize，以这个Prefill  
Batchsize划分服务化的并发请求数，即基于纯模型结果计算首Token时延上  
限；假设纯模型测试找到瓶颈处Prefill Batchsize为 ，此时单个T  
Prefill Batch的纯模型计算时间为 prefill ，要估计的服务化实际并发为  
Con，则服务化此轮并发请求的Prefill总时间估计为Con.TBS prefillprefill ，在先来先服务（FCFS）调度策略下，平均首Con1+BS prefill T2 prefill

Token时延约为

大并发下吞吐、非首Token时延。

Decode是带宽瓶颈而非算力瓶颈，理论单个Decode的Batchsize越大，总吞吐越大，直至达到显存限制。  
服务化的输出吞吐和非首Token时延，可参考相同并发（即Decode  
Batchsize）下的纯模型结果作为理论估计的上限，考虑到服务化推理会有框架侧调度等额外的时间开销，服务化吞吐的乐观估计可折算为纯模型吞吐0.8\~0.9倍。

显存瓶颈下的最大并发/maxBatchsize的理论上限：Decode阶段maxBatchSize主要受显存限制，可根据KV Cache Block数量和序列长度计算。

KV Cache缓存池以Block为单位分配，Block的大小通过MindIE服务化参数配置中的cacheBlockSize设置，一般使用默认值128，即每个Block可存储128个Token，根据使用场景的上下文长度计算每个请求需要占用的KV Cache Block数量：

最大Block数量=Ceil(输入Token数/cacheBlockSize)+Ceil(最大输出Token数/cacheBlockSize)  
平均Block数量=Ceil(平均输入Token数/cacheBlockSize)+Ceil(平均输出Token数/cacheBlockSize)。

MindIE服务端NPU上KV Cache Block总可用数量 Total Block Num可通过实测获取。

清空/root/mindie/log下的所有旧日志文件，通过使能环境变量  
MINDIE_LLM_PYTHON_LOG_LEVEL和  
MINDIE_LLM_PYTHON_LOG_TO_FILE，使MindIE LLM中Python程序运行时产生INFO级别日志并写入文件；

在确定除maxBatchSize之外的其他服务化配置后，使用默认值拉起服务，在服务端拉起成功后，使用grep命令搜索MindIE LLM Python程序日志中的"npuBlockNum"关键字，根据使用的卡数不同会返回多个结果，其中的最小值即为MindIE服务端NPU上KV Cache Block总可用数量，以图5-76为例，总可用数量为817。

![](images/7daa170b62c45c8af7baf2c5ed285c5bb440daf11b39d42d0e85626b7f284917.jpg)  
图 5-76 获取 MindIE 服务端 NPU 上 KV Cache Block 总可用数量示例

为保持最佳性能，maxBatchSize一般不宜超过Floor[Total Block Num/最大Block数量]；若请求之间的上下文长度差距较大，则上限可适当提高至Floor[Total Block Num/平均Block数量]。

# 5.7.3.2 针对目标进行调优

明确优化目标后，再调整服务化参数，从而提升服务化性能。

# 核心参数介绍

核心参数请参见表5-14，详细介绍请参见《MindIE LLM开发指南》。

表 5-14 核心参数说明  

<table><tr><td colspan="1" rowspan="1">优化方向</td><td colspan="1" rowspan="1">关键参数</td><td colspan="1" rowspan="1">推荐值/策略</td></tr><tr><td colspan="1" rowspan="3">低时延</td><td colspan="1" rowspan="1">maxPrefillBatchSize</td><td colspan="1" rowspan="1">小批量（4-16），降低首Token计算量。</td></tr><tr><td colspan="1" rowspan="1">supportSelectBatch</td><td colspan="1" rowspan="1">false,强制Prefill优先调度。</td></tr><tr><td colspan="1" rowspan="1">maxQueueDelayMicroseconds</td><td colspan="1" rowspan="1">≤50ms（减少等待延迟）。</td></tr><tr><td colspan="1" rowspan="5">高吞吐</td><td colspan="1" rowspan="1">maxBatchSize（Decode阶段）</td><td colspan="1" rowspan="1">最大化（受显存限制）。</td></tr><tr><td colspan="1" rowspan="1">maxPrefillBatchSize/maxPrefillTokens</td><td colspan="1" rowspan="1">相对调高，按照实际平均输入折算，使maxPrefilltokens在10000上下。</td></tr><tr><td colspan="1" rowspan="1">supportSelectBatch</td><td colspan="1" rowspan="1">开启吞吐优先调度。</td></tr><tr><td colspan="1" rowspan="1">maxQueueDelayMicroseconds</td><td colspan="1" rowspan="1">等待时间加大，让测试开始时尽量组成大的Batch。</td></tr><tr><td colspan="1" rowspan="1">RequestRate（通过测试工具设定）</td><td colspan="1" rowspan="1">增加下发频率，提高至硬件上限。</td></tr><tr><td colspan="1" rowspan="2"></td><td colspan="1" rowspan="1">Concurrency（通过测试工具设定）</td><td colspan="1" rowspan="1">同步逐步提升并发至吞吐饱和点。</td></tr><tr><td colspan="1" rowspan="1">序列长度三件套</td><td colspan="1" rowspan="1">根据实际场景和要求合理配置maxlnputTokenLen， maxlterTimes,maxSeqLen。</td></tr></table>

# 手动调优

针对未达成目标的参数进行调优，例如时延不满足目标或吞吐不满足目标。

调优前可以先确认各个参数的理论上限和理论最优配置，比如计算理论maxBatchSize最优值等，具体计算方法可参考服务化性能上限评估。

# 工具调优

手动调优需要一定的服务化基础。想要更便捷的调优，可以使用服务化调优工具（msServiceProfiler）的专家建议功能，请参见msserviceprofiler 工具介绍。 使用专家建议工具需要先使用MindIE Benchmark进行服务化性能测试，专家建议工具会基于MindIE Benchmark测试结果落盘文件给出调优建议。

# 疑难问题

上边所述的服务化调优，均为黑盒状态下的调优。我们并不清楚请求具体的调度过程，比如哪些请求组成了一个Batch、一个Batch有多大、什么时间执行的Prefill、什么时间执行的Decode等。

某些问题无法仅靠黑盒调优完成，需要展开进一步分析。与纯模型的性能优化类似，需要借助msServiceProfiler进行分析。

# 5.7.4 服务化性能调优定位案例

表 5-15 调优案例  

<table><tr><td rowspan=1 colspan=1>问题类型</td><td rowspan=1 colspan=1>调优思路</td></tr><tr><td rowspan=1 colspan=1>框架调度耗时导致性能劣化</td><td rowspan=1 colspan=1>采集服务化Profiling数据，查看两次调度之间的Gap、框架在做什么，来减少服务化框架的性能损耗。</td></tr><tr><td rowspan=1 colspan=1>模型执行耗时优化案例</td><td rowspan=1 colspan=1>分析服务化或纯模型Profiling数据的算子执行时间，确定性能瓶颈</td></tr><tr><td rowspan=1 colspan=1>显存瓶颈分析案例</td><td rowspan=1 colspan=1>根据产品规格，计算理论上最大Batchsize，使用服务化Profiling动态采集功能，分析KVCache实际使用情况，合理调整最大Batchsize、最大序列长度、显存分配相关的环境变量等配置。</td></tr><tr><td rowspan=1 colspan=1>优化服务化参数，降低时延提升吞吐</td><td rowspan=1 colspan=1>采集服务化核心指标，分析调度行为，对服务化参数进行优化，在满足SLO时延要求的前提下尽可能提升吞吐。</td></tr></table>

# 5.7.4.1 框架调度耗时导致性能劣化

# 问题现象

同一个环境，其他配置都一样，MindIE 2.0.RC1相比MindIE 2.0.T3的服务化性能劣化了很多，需确认是否为版本问题。

如图5-77所示，上半部分截图为纯模型测试结果，300并发下纯模型Decode阶段平均时延为36ms，请查看图中红框标注的“non_first_token_time”；下半部分为服务化测试结果，300并发下Decode阶段平均时延约为66ms，请查看图中红框标注的“DecodeTime” 。

![](images/66dbfc6e90e3dda403ea5838ce3499565e6d92e6ac690e9400c9e389d296df98.jpg)  
25-05-2009:43:42.880+08:00][8017][281472941050432][benchmark][INF0][output.py:121] BenchMark test common metricresultis:

图 5-77 MindIE 2.0.T3 性能测试结果  

<table><tr><td>Common Metric</td><td>value</td></tr><tr><td>CurrentTime TimeElapsed DataSource Failed Returned Total Concurrency ModelName</td><td>2025-05-20 09:43:42 38.1024 s None 0(0.0% 300( 100.0% 300[ 100.0%</td></tr><tr><td>lpct Throughput GenerateSpeed</td><td>300 DeepSeek-R1-Distill-Qwen-14B 6.0002ms 7.8735 req/s 4031.2467token/s GenerateSpeedPerClient 13.4375 token/s</td></tr></table>

如图5-78所示，上半部分截图为纯模型测试结果，300并发下纯模型Decode阶段平均时延为35.44ms，请查看图中红框标注的“non_first_token_time”，与2.0.T3性能非常接近；下半部分为服务化测试结果，300并发下Decode阶段平均时延约为95ms，请查看图中红框标注的“DecodeTime”，较2.0.T3版本性能劣化50%。

![](images/d7e6f7efdbc6190bd48de6b79584c7f502242dba4e58da9f3f278f242427a655.jpg)

步骤1 使用预检工具dump对比一下配置，如图5-79所示，其中“ms_performance_prechecker_dump_20250520_152124.json”为MindIE 2.0.T3版本环境的落盘文件，“ms_performance_prechecker_dump_20250520_152138.json”为MindIE 2.0.RC1版本环境的落盘文件，除日志设置等不影响性能的环境变量外，没有看到明显的配置差异。

![](images/9ed2f92ad8721c82be1120c7470064cea3714c8a807527caf0e1e5d61683a18d.jpg)  
图 5-79 对比配置

步骤2 采集MindIE 2.0.RC1的服务化性能数据进行对比，发现MindIE 2.0.RC1的Decode阶段forward之间的间隙过大，说明CPU侧的前后处理耗时长，如图5-80所示。

![](images/8ff368f53ef92a43aa7db133431d9f4565d38ce5c857cc3dd7c3f58606100662.jpg)  
图 5-80 查看 forward

步骤3 开启异步调度，缩短forward间隙后，MindIE 2.0.RC1版本E2E输出吞吐2900->4500，较T3提升500token/s。异步调度的开启方式请参见《MindIE LLM开发指南》的“关键特性 $>$ 异步调度”章节。

![](images/ec614edc354fa42e7c5f06fca2f6e78707fe31340c6b68322900c1c61a603fd2.jpg)  
图 5-81 开启异步调度

[2025-05-2110:15:10.794+08:00][39703][281473716972096][benchmark][WARN][visual_utils.py:130][MIE01W0004]Nod ata to draw histogram: Frequency_Histogram_of_Decoder_Batch_Size_None.png，return.Please verify which mode has the da [2025-05-2110:15:12.975+08:00][39703][281473716972096][benchmark][WARN][visual_utils.py:130][MIE01W0004]Nod atato draw histogram: FrequencyHistogram_of_Queue_Wait_Time_Nne.png，return.Please verify which mode hasthe data [2025-05-2110:15:14.817+08:00][39703][281473716972096][benchmark][WARN][visual_utils.py:178][MIE01W0004] Nod ata to plot: Scatter_Plot_of_Decoder_Batch_Size_vs_Decode_Token_atencies_None.png,return.Please verify whichmode ha s the data to plot

# ----结束

# 5.7.4.2 模型执行耗时优化案例

问题现象

DeepSeek PD分离大规模专家并行方案场景下，模型性能显著劣化。

# 解决方案

步骤1 使用msServiceProfiler采集D节点服务化性能数据。

步骤2 分析采集的性能数据，发现同一个D节点内的两台不同服务器，Decode阶段存在快慢卡现象。具体表现为来自两个服务器上的两张卡单个Decode执行时间分别为260ms和170ms，差距较大；一张卡的算子泳道出现很长的"MIX_AIC"字段，说明此时该卡在执行通算融合算子并且同步等待时间很长，可以认为此卡为快卡，另一张卡"MIX_AIC"较短，为慢卡。

![](images/bfbe59a31248fa6b456fbae094112d852a00d0f88522a0d554104a33141f3691.jpg)  
图5-82 快卡性能数据截图

![](images/47fad8d1ceae90468b5af08f8f8ac637d9e954893e97a7eed5df1e1bf5c21342.jpg)  
图5-83 慢卡性能数据截图

步骤3 从CANN CPU侧泳道（图5-82、图5-83中折叠为灰色）可以看出，每张卡启动任务的时间有差异，导致forward的第一个通算融合算子（moedispatch）时间不同，可以观察到90ms+的同步等待；去除第一个moedispatch后，剩余140ms左右的计算时间，两张卡算子性能接近——结合以上两点可以断定快慢卡是模型下发而非算子下发的时间差异导致的。

步骤4 通过在D节点开启分布式调度特性（使能环境变量MINDIE_ENABLE_DP_DISTRIBUTED，推理解决方案25.0.RC1之后的大规模专家并行方案已默认开启）后，性能恢复正常。

----结束

# 5.7.4.3 显存瓶颈分析案例

问题现象

DeepSeek双机服务化推理时，实际Batchsize无法达到192。

# 分析

步骤1 采集服务化性能数据。

步骤2 组到192 Batchsize时，随着Decode增加，Decode推理到185次时，可以看到KV Block即将用尽，剩余36（初始1747）；随后剩余的KV Block降低到0，Batchsize降为72，降低前最后的192 Batchsize推理到253长度。

![](images/9dfae11de560e666f37336476eadbb6f5f129b448f0d26e1ba851abb86cb8fdb.jpg)  
图 5-84 查看数据  
图 5-85 实际执行

步骤3 从实际的执行来看，Batchsize=150的情况下，可以执行到Decode接近1000。

<table><tr><td rowspan=1 colspan=1>rid                                                                     一 iter</td></tr><tr><td rowspan=1 colspan=1>122                                                                      881</td></tr><tr><td rowspan=1 colspan=1>72                                                                       881</td></tr><tr><td rowspan=1 colspan=1>116                                                                      881</td></tr><tr><td rowspan=1 colspan=1>70                                                                       1009</td></tr><tr><td rowspan=1 colspan=1>68                                                                       1009</td></tr><tr><td rowspan=1 colspan=1>118                                                                      881</td></tr><tr><td rowspan=1 colspan=1>66                                                                       1009</td></tr><tr><td rowspan=1 colspan=1>64                                                                       1009</td></tr><tr><td rowspan=1 colspan=1>120                                                                      881</td></tr><tr><td rowspan=1 colspan=1>62                                                                       1009</td></tr><tr><td rowspan=1 colspan=1>60                                                                       1009</td></tr></table>

步骤4 估算显存瓶颈下的并发上限，Profiling显示可用的KV Cache block数量是1747，默认配置下每个Block大小是128个Token，根据1024输入2048输出的上下文要求，平均上下文为1536，KV Cache平均可容纳1747/[(1024+2048)/2/128]≈145。

![](images/8650b9200e07444d83655f5274b1c8227f0a6c446d6900e22bb22e88e653a978.jpg)  
图 5-86 Block 数量

步骤5 调整服务化参数，export NPU_MEMORY_FRACTION=0.96，maxSeqLen降低到3K，达到最优效果。

----结束

# 5.7.4.4 服务化参数优化案例

# 问题现象

Qwen3-32B 单机4卡服务化部署，用户使用场景中并发较小且请求缓慢达到（最大并发40，实际使用情况接近请求速率9.4），PD混部的默认服务化配置下测试输入长度128输出长度100的性能，平均非首Token时延 48ms，但非首Token时延请求级别P99分位数（TPOT SLO P99）为157ms，导致个别请求流式输出卡顿，无法满足用户非首Token时延50ms的时延要求。

# 解决方案

步骤1 观察默认服务化配置下，性能测试的初步结果如图5-87所示，可以发现（1）默认的Prefill优先调度下，首Token时延远最大值也只有158ms，说明Prefill性能正常且未达到算力瓶颈（2）P75首Token时延小于50ms，说明大多数非首Token时延正常，说明实际Decode性能已符合预期（3）注意到非首Token的P90/P99/最大时延偏高，推测是因为请求缓慢达到，新请求的Prefill打断推理中请求的Decode，导致部分非首Token需等待新请求Prefill完成，非首Token时延激增。

![](images/8619f0f46c22659a7b1d0b8c509175554d2fc83f0e596a0e11a668b72743dbd9.jpg)  
[2025-06-2811:12:24.983+08:00][35841][281472993185824][benchmark][INF0][output.py:121] TheBenchMarktestcommonmetricresultis:

![](images/1f8f91f0b768da07d4203f0a236be5c2031b87f9e9915b03d6c38d7652333cdc.jpg)

步骤2 在此场景下，可以开启SupportSelectBatch，通过设置prefillTimeMsPerReq和decodeTimeMsPerReq调整Prefill和Decode的优先级，使调度器允许在一定情况下Decode优先。此策略可以减少“新请求Prefill打断推理中请求的Decode“的情况，进而提高连续Decode的占比，降低非首Token时延，如图5-88所示；虽然此策略会导致部分请求Prefill等待，但此案例场景中模型Prefill性能已经足够好，即使首Token时延略有增加，仍可通过调整优先级参数，将首Token时延控制在用户可以接受的范围内。

![](images/dedc986b2fca85f02922d48983c1cde578f1f2d8681a50bf58789c49a58c9d12.jpg)  
图 5-88 非首 Token 时延劣化现象原理示意图

![](images/dbba99a683f28a04f9be850c8c517a60953e238e0e4b5eaa13d4a527682ca6e8.jpg)  
开启SupportSelectBatch：通过调度算法在一些情况下不优先做Prefill，继续连续做Decode，部分减少长时延Decode

步骤3 调整优先级参数，prefillTimeMsPerReq越高，decodeTimeMsPerReq越低，则Decode优先级越高，反之则Prefill优先级越高；设置supportSelectBatch为true，prefillTimeMsPerReq为1000，decodeTimeMsPerReq为1，性能测试结果如图5-89，此时首Token时延增加至1562ms，平均非首Token降低至31.7ms，P99非首Token时延降低至36ms，整体流式输出流畅，满足用户需求。

![](images/5c9f752f10a73fcf4ea0304fae135f1569335c7a19faf4d950c06d34a250ea81.jpg)

# ----结束

# 5.7.5 DeepSeek 进阶调优

进阶调优对于服务化和纯模型都有收益，暂不区分；部分进阶调优手段可能需要相关组件支撑。

# 显存分析

优化达到瓶颈后，一个直观的方法是优化显存来使用更优的配置（例如并发数，并行策略等），可考虑采用量化来降低计算量和显存，目前适配最好方案为W8A8。

# 并行策略调优

当前16卡推理场景，一般最优配置是TP=8，DP=2，MOE_TP=4，MOE_EP=4；但用户有不同的Host端（arm/x86），输入输出要求等，这会造成最优的并行策略发生变化，因此需要调整并行策略。

# 通信策略优化

不同通信策略会产生的通信量不同，需要根据并行策略进行评估。

调优建议：

● Attention的TP尽量减小，增加DP，避免KV Cache复制，避免重复访问和存储。  
● 纯DP通信可节省KV Cache，但模型权重需要占更多空间。  
● 专家通信（EP）通过ATB-Models安装目录中配置文件config.json里的ep_level关键字配置，AlltoAll理论通信量会少一些。  
● 通信可选择走LCCL或者HCCL，通常LCCL性能更优，但部分并行策略下适配可能有问题。

# 其他优化方法

权重格式转换：权重转换为NZ格式，减少格式随路转换耗时。

总请求设置：由于Decode阶段性能存在爬坡（受限Request Rate，前期Batch较小，未达到瓶颈，Decode阶段的Batch在逐步提高），因此建议设置总请求数=并发数\*10。

# 5.8 版本升级性能劣化定位方法论

当出现以下场景时，均可使用msprof-analyze compare工具对比分析前后的性能差异，快速定位性能劣化原因，工具具体情况请参见4.2.3 模型调优快速分析（msprof-analyze命令行工具）。

● PyTorch训练工程从GPU迁移至NPU后出现性能劣化，通过工具分析出劣化点。  
● PyTorch或MindSpore训练工程在NPU上，不同版本之间存在性能差距，通过工具定位具体差异。  
● PyTorch训练工程从GPU迁移至MindSpore NPU后出现性能劣化，通过工具分析出劣化点。

# 5.8.1 确认主要性能瓶颈

使用msprof-analyze compare工具，查看打点信息，如图5-90所示。或查看performance_comparison_result_{timestamp}.xlsx的OverallMetrics页。

![](images/7bef39ba1e216f51ca844a0e6a8f7ff1c0eb1b7dd4f10abf6a03a868aa5f1eb2.jpg)  
图 5-90 打印信息

需重点关注四个核心维度差异，找到差异最明显的维度进一步分析。

● Computing Time（计算时间）  
● Uncovered Communication Time（未被计算掩盖的通信时间）  
● Mem Usage（内存使用）  
● Free Time（空闲时间）

# 5.8.2 算子性能劣化对比分析

# 算子级性能比对

msprof-analyze compare工具提供了算子级性能比对，结果在  
performance_comparison_result_{timestamp}.xlsx中“OperatorCompare”和“OperatorCompareStatistic”的sheet页，呈现了全面的Shape信息、下发的kernel、device耗时，以及内存占用情况。

步骤1 查看“OperatorCompareStatistic”页，给出了算子调用次数与Device上运行总耗时，按照"Diff Duration(ms)"或"Diff Ratio"字段逆向排序，找出耗时差距TOP的算子。

步骤2 在“OperatorCompare”页，搜索耗时差距TOP的算子，查看具体执行的kernel耗时，如图5-91所示，寻找可优化点。

图 5-91 查看执行的 kernel 耗时  

<table><tr><td rowspan=1 colspan=1>Base Profling; /hotte/data_space/ubuntu_3748236_20250718065928720_ascend_ptl</td><td rowspan=1 colspan=1>Input Shape</td><td rowspan=1 colspan=3>Kernel Details    Device Duration(us)</td><td rowspan=1 colspan=3>ComparisonProfiling:/home/dataspace/ubuntu_3748234_20250718065928720_ascend_p</td><td rowspan=1 colspan=1></td><td rowspan=1 colspan=1></td><td rowspan=1 colspan=1></td><td rowspan=1 colspan=1></td></tr><tr><td rowspan=1 colspan=1>aten:argmin</td><td rowspan=1 colspan=1>[23520, 8192], 0</td><td rowspan=1 colspan=1>[foaf,‘Scalar&#x27;, &#x27;Scalar]</td><td rowspan=1 colspan=1>voidat:native:reduce_kemek&lt;512,1.at:nate:ReduceOpcfloat.at:ative.ArgMnOps-foat&gt;,unsigned int, Iong, 4&gt;&gt;(at:native:ReduceOp&lt;float,at:native:ArgMinOps&lt;float&gt;,unsiged int, og, 4ua585.0]</td><td rowspan=1 colspan=1>Device Duration(us)585.00</td><td rowspan=1 colspan=1>Operator Nameatenc:argmin</td><td rowspan=1 colspan=1>Input Shape23520,8192;</td><td rowspan=1 colspan=1>Input Typefloat,Int,Bool</td><td rowspan=1 colspan=1>Kemel DetailsArgMin, 35707, AI_CPU [duration 277683.38]trans_Cast_9,35708, A_CORE (duration826)277,698.46Cast, 35709, A_CORE (duratonc 6.82)</td><td rowspan=1 colspan=1>[Device Duration(us)</td><td rowspan=1 colspan=1>Diff Duration(us)277,113.46</td><td rowspan=1 colspan=1>DifRatio</td></tr></table>

----结束

# module 级性能比对

compare工具也支持module级比对，帮助快速识别劣化的模块和算子，并定位至代码块。

步骤1 查看“ModuleCompareStatistic”页， 筛选“Operator Name”字段为[ TOTAL ]，将模块总体情况按照“Device Self Time(ms)”字段逆向排序，找出耗时差距TOP的模块。

步骤2 查看“ModuleCompare”页，查找耗时差距TOP模块下的劣化算子。

步骤3 通过调用栈找到对应的代码行，如图5-92所示。

图 5-92 查找代码行  

<table><tr><td colspan="11">se Profii eparisonProfiit home/data_space/ubuntu_1378758_20250207023158118_asoend_pt</td></tr><tr><td></td><td>opieLer</td><td></td><td></td><td></td><td></td><td>arNave</td><td>Kenel Detals 97.84</td><td>Device Self Timnejus)Device Total Tinelus) 07.84</td><td></td><td>uDinTetilRatle</td><td></td></tr><tr><td>nn.Module: ParalelAttent nnModue: ParalelTrand</td><td></td><td>BACKWARD V[ n.cat DistbutedDuts arallel_1/ FloattGModule fen add</td><td>aclnCa_CorcedD_Cane aclnnAdd AdcAiCore_Ad4.76</td><td></td><td></td><td>aen.cat aten add</td><td>aclhnCa_CercadD_Con,97.84 aclnnAdd_AdAiCore_Ad14.30</td><td></td><td>9.54</td><td></td><td>HA MA</td></tr><tr><td></td><td></td><td>n.cat</td><td></td><td></td><td>TOT men.cat</td><td></td><td>BchnCa CereD Can043</td><td>94.32 02.48</td><td></td><td>retan_gem23t): &lt;moduepret</td><td>train_gptpy23t) &lt;module&gt;preg</td></tr><tr><td>nnModule: ColumnParal</td><td>BACKWARD y</td><td>DistrbutedDatsParalel_1 aten mratrul</td><td>tindVuCe0 xhrl4sn VatuCen09.40</td><td></td><td>sterc motrul</td><td></td><td>rtndWuCr15T6</td><td></td><td>78.00</td><td></td><td>H</td></tr><tr><td>nnModule: ColumnParal2 nn Module: ColumnParal;</td><td></td><td>BCWRDnbusDs5Pad1 gterorstnul</td><td>achrtatnul_WaduCon68.02</td><td></td><td></td><td>n</td><td>S1 22 achrtatnudMadauCor131.54</td><td></td><td></td><td></td><td></td></tr><tr><td>nnModule: ColumnParall nnModule: R RowPeraleLi</td><td></td><td>BACKWARD V DsrusdDesPes1 gsonetrd DistributedDataPeralel_0 BACKWARD</td><td>acrttotmuMadudCon79.36 acihnttatnul_Madurv3C80.06</td><td></td><td></td><td>aecnstd</td><td>achrtatmdMaddCer150.76 achnttatmul_Madaufv3C;151.28</td><td></td><td>71.40</td><td></td><td>MA</td></tr><tr><td></td><td></td><td>BACKWARD y DistributedDataParalel_ IIAL 104CKW0RD yD DistributedDataPualel_0 en matmul</td><td>shrtd VuW03Cg0.cI</td><td></td><td>134.94</td><td>sten metru</td><td>hrtind VeMOC1068</td><td>345.42</td><td>163.48</td><td>En_gpt py(231) </td><td></td></tr><tr><td>nnModue: RowParalkL nnModuie: RowParalelLi</td><td></td><td>BACKWARD y DistributedDataParallel_o aten matrul</td><td>hrn VadWn3C10488 acrdatnulVsduulCon9.25</td><td></td><td></td><td>sterc matru</td><td>an VaMOC19414 achnlatnudMsduulCor128.15</td><td></td><td></td><td></td><td>MOA MA</td></tr><tr><td>Moduie: ColmnPaa</td><td></td><td>/ BACKWaARD /DistributedDataParalel_0|TOTAL ten matmul</td><td>518.17</td><td>518.17</td><td></td><td></td><td>958.58</td><td>958.58 440.41</td><td>440.41</td><td></td><td>retrain_gtgy(231): &lt;module&gt; pretrcprebrain_g[ py(231); &lt;module&gt; prets) NOA</td></tr></table>

# ----结束

# 5.8.3 通信时间（未被掩盖的）劣化对比分析

步骤1 确认是否存在显著的通信算子性能劣化现象。

打开performance_comparison_result_.xlsx文件中的“CommunicationCompare”工作表，重点对比以下通信大算子的性能指标，如图5-93所示。

● 算子类型（Broadcast、AllReduce等）  
● 耗时指标（平均耗时、最大/最小耗时）与调用频次统计关联的子任务信息（Reduce_Inline、Notify_Record、Notify_Wait、Memcpy等）

图 5-93 对比通信大算子的性能指标  

<table><tr><td>Base Profiling</td><td colspan="11">Comparison Profiling:/home/data_space/tmp/compare_testcase_3/ubuntu_137875B_20250207023156116_ascend_pt</td></tr><tr><td>OrderId</td><td>ion OPName</td><td>Task Nar</td><td>Calh</td><td>Total Duration(us)Avg Dura</td><td>2,791.57</td><td>Max Dura 13,624.81</td><td>n(us)Min Duration(us)</td><td>Communication OP NameTask Name</td><td></td><td>Calis</td><td>2,882.78</td><td>Total Duration[us]Avg Duration(us) 22.37</td><td>Max Durationus)Min Duration[us)</td><td></td><td>Diff Duratio on(us)</td><td>0.63% Dift Ratio</td></tr><tr><td></td><td>broadcast</td><td>Reduce Ihine</td><td></td><td>457,817.78 4,595.57</td><td>35.62 248 22</td><td>22.74 0.58</td><td>broadcast</td><td></td><td>Reduce_Inline</td><td>128 16 10.22</td><td>0.64</td><td>25 20 0.78</td><td>21.10 0.58</td><td></td><td>-454,954.99</td><td></td></tr><tr><td>3</td><td></td><td>Notty Record</td><td>156</td><td>11066 0.71 117,656.15 754.21</td><td>0.78 12,625.49</td><td>0.68</td><td></td><td>Memcpy</td><td></td><td>17.18 20.58</td><td>1.07 0.86</td><td>1.38 2.38</td><td>0.68 0.00</td><td></td><td></td><td></td></tr><tr><td>4 5</td><td></td><td>Noty Wat Memcpy</td><td>155</td><td>8,546.07 54.43</td><td>281.37</td><td>0.00 0.68</td><td></td><td></td><td>Motly_Wat Notify_Record</td><td>24 17.06</td><td></td><td>0.74</td><td>0.70</td><td></td><td></td><td></td></tr><tr><td></td><td>allreduce</td><td></td><td>P</td><td>65,185.20</td><td>814.82</td><td>5,573.85</td><td>allreduce</td><td></td><td></td><td></td><td>59,171.64</td><td></td><td>5,605.66 </td><td>-6,013.56</td><td></td><td>90.77%</td></tr><tr><td></td><td></td><td>Reduce_hine es</td><td></td><td>198609</td><td></td><td></td><td></td><td></td><td>Redixe_Jrine </td><td>2400</td><td>8.142.94 8.025.08</td><td>防</td><td>6</td><td></td><td></td><td></td></tr><tr><td>8</td><td></td><td></td><td>40.82</td><td></td><td>2.34</td><td></td><td></td><td></td><td></td><td>84.48</td><td></td><td>0.72 1.32</td><td>0.68</td><td></td><td></td><td></td></tr><tr><td>10 1</td><td>allgather</td><td>Notty_Wat</td><td></td><td>16,037.90 311.01 543,322.09 316.01</td><td>3,199.34 7,249.65</td><td>10.00 0.60</td><td>allgather</td><td></td><td>Notfy_Wat</td><td>110 1.703</td><td>19,918.30 409,124.12</td><td>160.80 240.24</td><td>2.111.32 0.60 0.00</td><td></td><td></td><td></td></tr><tr><td>12</td><td></td><td>Reduce_hine</td><td></td><td>1,904.68 2.84</td><td>315.40</td><td>0.58</td><td></td><td></td><td>Reduce_Infine</td><td>567</td><td>1,857.56</td><td>218 3.28</td><td>0.58</td><td></td><td>-134,197.97</td><td>75.30%</td></tr><tr><td>13 14</td><td></td><td></td><td></td><td>139.049.10 142.18 477.23 0.72</td><td>875.45 2.60</td><td>0.74</td><td></td><td></td><td>Mekgy</td><td></td><td>121,892.14</td><td>141.57</td><td>647.78 0.72</td><td></td><td></td><td></td></tr><tr><td>E</td><td></td><td>uPR uety_Wat</td><td></td><td>53,767.29 牌</td><td>7,013.40</td><td></td><td></td><td></td><td>Notly Record Notify_Wait</td><td></td><td>423.12 4836.58</td><td>0.72 3.84 新</td><td>0.68 0.00</td><td></td><td></td><td></td></tr><tr><td>1</td><td>reducescatter</td><td></td><td></td><td>348,845.82</td><td>8</td><td></td><td></td><td>reduceseatter</td><td></td><td></td><td></td><td></td><td></td><td></td><td>-46,175.78</td><td>86.76%</td></tr><tr><td>18</td><td></td><td>Redxenire</td><td>的多好出</td><td>8</td><td>275.83</td><td></td><td></td><td></td><td>Redace_hire Memcpy</td><td></td><td>21,944.80</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td>19</td><td></td><td></td><td></td><td>0.71 330.01</td><td>0.78</td><td></td><td></td><td></td><td>Notty_Record</td><td></td><td>326.26 0.74</td><td>物 2.12</td><td>0.70</td><td></td><td></td><td></td></tr><tr><td>20</td><td>send</td><td></td><td>253</td><td>28,196.82 59.49</td><td>1,342.41</td><td></td><td></td><td></td><td>Notify_Wait</td><td></td><td>94,089.22</td><td>209.56</td><td>9,341.94 0.00</td><td></td><td></td><td></td></tr><tr><td>21</td><td></td><td>Reduce_Infine</td><td>2,006</td><td>78,597.21 1,573,439.70 28.02</td><td>6,219.13 33,341.99 318.53</td><td></td><td>send</td><td></td><td>Reduce_Infine</td><td>248 3,004</td><td>1,678,295.82 07,198.86 28.74</td><td>6,767.32 304.56</td><td>33,344.14 0.60 0.58</td><td></td><td>104,858.12</td><td></td></tr><tr><td></td><td></td><td>Memcpy</td><td>3,136</td><td>343,522.30 109.54</td><td>254.55</td><td>4.42</td><td></td><td></td><td>Memcpy</td><td>3,403</td><td>369.000.40</td><td>100.67 217.14</td><td>4.44</td><td></td><td></td><td></td></tr><tr><td></td><td></td><td>Notly Record</td><td></td><td>0.71</td><td>0.80</td><td></td><td></td><td></td><td>Notfy Record</td><td></td><td>1,884.0 1,982,449.08</td><td></td><td>0.70</td><td></td><td></td><td></td></tr><tr><td></td><td></td><td>Notty_Wat</td><td></td><td></td><td></td><td>80</td><td></td><td></td><td>Notily_Wait</td><td></td><td>2,905,972.40</td><td></td><td>0.00 208 96</td><td>42,51884</td><td></td><td></td></tr><tr><td></td><td></td><td>Roducerire</td><td>18</td><td>125,642.17 250,128.78</td><td>772.32 398.39</td><td></td><td></td><td></td><td></td><td>7</td><td>125.893.70 39.65</td><td></td><td>0.58</td><td></td><td></td><td></td></tr><tr><td></td><td></td><td></td><td>2,963</td><td>2,186.16 0.74</td><td>W 16.52</td><td></td><td></td><td></td><td></td><td>2,990</td><td>2.144.84 0.72 2 524,735.58</td><td>6.30 844.39</td><td>30.063.58 0.00 0.68</td><td></td><td></td><td></td></tr><tr><td>29 30</td><td></td><td>Notty_Walt</td><td>2,966</td><td>2,624,671.24</td><td>84.92</td><td>30,061.28 0.00</td><td></td><td></td><td>Memcpy</td><td>3,083</td><td>|249,680.18</td><td>80.99 383.12</td><td>0.68</td><td></td><td></td><td></td></tr></table>

步骤2 在“OverallMetrics”工作表中，按通信域维度进行深入对比分析。

需重点关注同一通信域下transit_time，wait time指标的差异分析，如图5-94所示。

图5-94 关注重点指标差异  

<table><tr><td>Uncovered Communication Time</td><td>1,798.60</td><td>53.10%</td><td></td><td>1,706.90</td><td>50.42%</td><td>-91.70</td><td>94.90%</td></tr><tr><td>tp: Communication</td><td>976.61</td><td>28.83%</td><td></td><td>606.84</td><td>17.92%</td><td>369.76</td><td>62.14%</td></tr><tr><td>Wait</td><td>19.10</td><td>0.56%</td><td></td><td>0.52</td><td>0.02%</td><td>-18.58</td><td>2.70%</td></tr><tr><td>Transmit</td><td></td><td>28.27%</td><td></td><td>606.33</td><td>17.91%</td><td>-351.18</td><td>63.32%</td></tr><tr><td></td><td>957.51</td><td>0.30%</td><td></td><td>0.00</td><td>0.00%</td><td>-10.10</td><td>0.00%</td></tr><tr><td>dp_cp:Group10.174.216.241%enp189s0f1_510.10</td><td></td><td>0.30%</td><td></td><td>0.00</td><td>0.00%</td><td>10.10</td><td>0.00%</td></tr><tr><td>Transmit</td><td>10.10</td><td></td><td></td><td>0.00</td><td></td><td>-881.93</td><td></td></tr><tr><td>pp:Group10.174.216.241%enp189s0f1_5500881.93 Wait</td><td></td><td>26.04%</td><td></td><td>0.00</td><td>0.00%</td><td>-1379.84</td><td>0.00%</td></tr><tr><td></td><td>1.379.84</td><td>40.74%</td><td></td><td></td><td>0.00%</td><td></td><td>0.00%</td></tr><tr><td>Transmit</td><td>497.91</td><td>-14.70%</td><td></td><td>0.00</td><td>0.00%</td><td>497.91</td><td>0.00%</td></tr><tr><td></td><td>default_group:Group10.174.216.241%enp1891,456.76</td><td>43.01%</td><td></td><td>1,641.48</td><td>48.48%</td><td>184.72</td><td>112.68%</td></tr><tr><td>Wait</td><td>0.00</td><td>0.00%</td><td></td><td>0.10</td><td>0.00%</td><td>0.10</td><td>119050.00%</td></tr><tr><td>Transmit</td><td>1,456.76</td><td>43.01%</td><td></td><td>1,641.38</td><td>48.48%</td><td>184.62</td><td>112.67%</td></tr><tr><td>embd:Group 10.174.216.241%enp189s0f1_555.62</td><td></td><td>0.17%</td><td></td><td>0.00</td><td>0.00%</td><td>-5.62</td><td>0.00%</td></tr><tr><td>Transmit</td><td>5.62</td><td>0.17%</td><td></td><td>0.00</td><td>0.00%</td><td>-5.62</td><td>0.00%</td></tr><tr><td>mp:Group 10.174.216.241%enp189s0f1_55005.86</td><td></td><td>0.17%</td><td></td><td>0.36</td><td>0.01%</td><td>-5.50</td><td>6.17%</td></tr><tr><td>Wait</td><td>5.81</td><td>0.17%</td><td></td><td>0.02</td><td>0.00%</td><td>-5.79</td><td>0.42%</td></tr><tr><td>Transmit</td><td>0.05</td><td>0.00%</td><td></td><td>0.34</td><td>0.01%</td><td>0.29</td><td>681.27% INF</td></tr><tr><td>Group10.174.216.241%enp189s0f1_55000_00.00</td><td></td><td>0.00%</td><td></td><td>940.65</td><td>27.78%</td><td>940.65</td><td rowspan="3">INF</td></tr><tr><td>Wait</td><td>10.00</td><td>0.00%</td><td></td><td>1390.74</td><td>41.08%</td><td>1,390.74</td></tr><tr><td>Transmit</td><td>0.00</td><td>0.00%</td><td></td><td>-450.09</td><td>-13.29%</td><td>-450.09</td></tr><tr><td>Group10.174.216.241%enp189s0f1_55000_00.00</td><td></td><td>0.00%</td><td></td><td>10.17</td><td>0.30%</td><td>10.17</td><td>INF INF</td></tr><tr><td>Transmit</td><td>0.00</td><td>0.00%</td><td></td><td>10.17</td><td>0.30%</td><td>10.17</td><td></td></tr><tr><td>Group10.174.216.241%enp189s0f1_55000_00.00</td><td></td><td>0.00%</td><td></td><td>5.63</td><td>0.17%</td><td>5.63</td><td>INF INF</td></tr><tr><td>Transmit</td><td>10.00</td><td>0.00%</td><td></td><td>5.63</td><td>0.17%</td><td>5.63</td><td>INF</td></tr></table>

步骤3 如果通信性能分析没有劣化的通信算子，代表通信与计算的并行度较差，继续进行NPU的集群性能分析。

----结束

# 5.8.4 内存劣化对比分析

算子内存比对结果在performance_comparison_result_\*.xlsx中“MemoryCompare”和“MemoryCompareStatistic”的sheet页呈现。

步骤1 查看“MemoryCompareStatistic”页，找出内存占用差距TOP的算子。

步骤2 查看“MemoryCompare”页，搜索内存占用差距TOP的算子，查看具体占用的算子。

例如：现场某次升级CANN软件包和PTA软件包后，发生OOM情况，于是采集升级前后两次的Profiling，根据算子级比对发现是aten::group_norm多申请了10GB+内存，如图5-95所示。

![](images/faea91f3b928020ddf24c8f9364df1f5c33a9ec015e12fd9a251f025cdd57243.jpg)  
图 5-95 算子比对

# ----结束

# 5.8.5 空闲时间劣化对比分析

当计算时间和通信时间无明显变化，但Free时间明显增大时，可以借助performance_comparison_result_\*.xlsx中“ApiCompare”页的比对结果，找到耗时差距TOP的API，结合MindStudio Insight界面的下发连线，确认是否有Host bound。

图 5-96 查看 ApiCompare 页  

<table><tr><td colspan="10">BaseProf</td><td colspan="3"></td><td></td><td></td></tr><tr><td>Order Id</td><td>lapi nam</td><td>Total Duration(ms)</td><td>Self Time(ms)</td><td></td><td></td><td>Total Duration(ms)</td><td>Self Time(ms)</td><td></td><td>Avg Duration( Calls</td><td></td><td>Diff Total Ratio</td><td>Diff Self Ratlo</td><td>Diff Avg Ratio</td><td></td><td>Diff Calls Ratlo</td></tr><tr><td></td><td>aten:zero_ acInnInplaceZero</td><td>13.55 6.74</td><td>6.81 6.74</td><td>0.01 0.01</td><td></td><td>15.59 8.61</td><td>6.98 8.61</td><td></td><td>0.02 0.01</td><td>1,030 1,030</td><td></td><td></td><td></td><td>1.00 1.00</td><td></td></tr><tr><td>2</td><td>destroy_event</td><td>9.55</td><td>9.55</td><td>0.00</td><td>3.730</td><td>10.71</td><td>10.71</td><td></td><td>10.00</td><td>3,734</td><td></td><td></td><td>1.00 1.00</td><td></td><td></td></tr><tr><td>4</td><td>aten:empty</td><td>41.53</td><td>14.03</td><td></td><td></td><td>46.03</td><td></td><td></td><td></td><td>3,084</td><td>10.84</td><td>0.87</td><td>1.00</td><td>0.86</td><td></td></tr><tr><td></td><td>aten:to</td><td>21.76</td><td></td><td></td><td></td><td>47.81</td><td></td><td></td><td></td><td>1,840</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>ten:lt fresh</td><td>0.14</td><td>0.45</td><td></td><td></td><td>0.16 0.72</td><td></td><td></td><td></td><td></td><td></td><td></td><td>1</td><td>1.00</td><td></td></tr><tr><td></td><td>aten:detach_</td><td>1820</td><td>0.25</td><td>1</td><td>134</td><td>0.25</td><td></td><td></td><td></td><td></td><td>1.00</td><td>1.00</td><td></td><td></td><td>1.00</td></tr><tr><td></td><td>detach. aten:_to_copy</td><td>1.81</td><td></td><td></td><td>0</td><td>23.78</td><td></td><td></td><td></td><td></td><td></td><td></td><td>1.00</td><td></td><td>1.00</td></tr><tr><td>10</td><td>atencempty strided</td><td>0.40</td><td>1841</td><td></td><td>70</td><td>11.12</td><td></td><td></td><td></td><td>582</td><td></td><td></td><td></td><td></td><td></td></tr><tr><td></td><td>sPy_tensor</td><td>119.76</td><td>119.76 17.04</td><td></td><td>17.482</td><td>190.93</td><td>190.93</td><td></td><td>0.01</td><td>17.582</td><td></td><td></td><td>1.00</td><td></td><td></td></tr><tr><td>12</td><td>sten:copy.. e10d:broadcast_</td><td>27.65 7.67</td><td>4.91</td><td></td><td>1,290 128</td><td>37.18 7.46</td><td>21.57 4.66</td><td></td><td></td><td>1,792 128</td><td>0.97</td><td>0.95</td><td>1.00</td><td></td><td></td></tr><tr><td>13</td><td>record_event</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td>1.00</td><td></td><td>1.00</td></tr><tr><td>1R</td><td>wait_event</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td>1.00</td><td></td><td>1.00</td></tr><tr><td></td><td>HccIBroadcast aten:lt</td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td></td><td>1.75</td><td></td><td>100</td></tr></table>