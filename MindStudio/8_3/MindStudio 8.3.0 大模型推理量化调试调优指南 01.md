MindStudio 8.3.0

# 大模型推理量化调试调优指南

文档版本 01  
发布日期 2026-01-19

![](images/86c52eaed20fdbc5c986ed54462d05f014dbfbcac666d5886d53a06363eae034.jpg)

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

1 推理量化精度问题的挑战..

2 推理量化精度调优指南..

3 推理量化精度调优案例. 8

#

# 推理量化精度问题的挑战

# 前提条件

需熟悉msModelSlim量化压缩工具的各项功能和基本操作。

# 问题与挑战

在昇腾平台上部署大模型时，需特别关注模型的推理性能和精度之间的平衡。而量化技术作为提高推理效率的有效手段，能够显著提升模型的推理速度。但在量化过程中也面临激活值难量化、离群值难量化及离群值误差累积等挑战，具体请参见表1-1。

表 1-1 问题量化场景列表  

<table><tr><td rowspan=1 colspan=1>场景</td><td rowspan=1 colspan=1>量化难点</td></tr><tr><td rowspan=1 colspan=1>激活值难量化</td><td rowspan=1 colspan=1>激活值动态生成，分布范围广，离群值多，直接量化易导致关键特征丢失。</td></tr><tr><td rowspan=1 colspan=1>离群值难量化</td><td rowspan=1 colspan=1>离群值难界定，且在量化过程中存在以下问题：·将离群值纳入量化区间，导致非离群值被压缩到狭窄的量化区间内，将会造成精度损失。不将离群值纳入量化区间，导致离群值量化后被裁剪为整数范围的最小值或最大值，丢失部分离群值信息，造成精度损失。</td></tr><tr><td rowspan=1 colspan=1>离群值误差累积</td><td rowspan=1 colspan=1>离群值聚集在特定通道（如Transformer注意力层输出通道），导致局部量化误差逐层传递，最终影响全局精度。</td></tr></table>

为了有效解决这些问题，我们可以使用msModelSlim量化压缩工具的离群值抑制算法，并结合精度调优，快速减少量化带来的精度损失，确保模型在实际应用中具备准确性和稳定性。

# 2 推理量化精度调优指南

# 概述

本章节提供了一套系统化的量化精度调优方法论，遵循“确认精度问题可信→调整离群值抑制算法→调整量化策略→调整校准集→量化回退”的递进路径。详细介绍了各步骤的操作方法、算法对比和配置示例，帮助用户在可接受的精度损失内实现模型的高效量化部署。

核心要点

优先使用Iterative Smooth、Flex Smooth Quant等离群值抑制算法抑制激活离群值。  
根据场景选择minmax/ssz/autoround等量化方法。  
通过优化校准集和敏感层回退进一步提升精度。

核心目标在可接受的精度损失内，实现模型的高效量化部署。

# 使用前准备

安装 msModelSlim 工具，详情请参见《msModelSlim工具安装指南》。

# 调优步骤详解

步骤1 确认精度问题可信

在开始调优前，需要排除环境干扰，确保问题真实存在。具体可参见表2-1。

表 2-1 调优前检查项  

<table><tr><td rowspan=1 colspan=1>验证项</td><td rowspan=1 colspan=1>具体操作</td></tr><tr><td rowspan=1 colspan=1>推理引擎验证</td><td rowspan=1 colspan=1>先用浮点模型在目标推理引擎上测评，确认是否能复现原始精度</td></tr><tr><td rowspan=1 colspan=1>测评结果检查</td><td rowspan=1 colspan=1>检查量化模型的测评输出，确认无上下文截断、超时等非量化问题</td></tr><tr><td rowspan=1 colspan=1>确定波动范围</td><td rowspan=1 colspan=1>了解测评数据集本身的精度波动范围，判断当前精度损失是否异常</td></tr></table>

# 步骤2 调整离群值抑制算法（关键步骤）

激活值中的离群值会大幅扩展量化范围，占用有效量化比特，导致精度损失。可以使用离群值抑制算法，将激活的量化难度“转移”到权重上。具体可参见表2-2。

表 2-2 离群值抑制算法对比  

<table><tr><td rowspan=1 colspan=1>算法</td><td rowspan=1 colspan=1>算法特点</td><td rowspan=1 colspan=1>适用场景与建议</td><td rowspan=1 colspan=1>配置示例链接</td></tr><tr><td rowspan=1 colspan=1>Smooth Quant</td><td rowspan=1 colspan=1>仅对norm-linear子图做平滑处理，支持对称/非对称</td><td rowspan=1 colspan=1>在Qwen、DeepSeek等热门系列模型上精度较差，不建议使用</td><td rowspan=1 colspan=1>Smooth_Quant.md</td></tr><tr><td rowspan=1 colspan=1>Iterative Smooth</td><td rowspan=1 colspan=1>解决o_proj、down_proj等层因无相邻LayerNorm而无法转移scale的问题。支持对称/非对称</td><td rowspan=1 colspan=1>优先使用。运行快，精度较高。超长序列校准集时优先使用。可调整alpha参数优化</td><td rowspan=1 colspan=1>Iterative_Smooth.md</td></tr><tr><td rowspan=1 colspan=1>Flex SmoothQuant</td><td rowspan=1 colspan=1>通过二阶段网格搜索自动寻找最优alpha和beta参数，实现更精细的平衡</td><td rowspan=1 colspan=1>当lterativeSmooth不达标、对量化时间不敏感且显存充足时尝试。运行速度较慢</td><td rowspan=1 colspan=1>Flex_Smooth_Quant.md</td></tr><tr><td rowspan=1 colspan=1>QuaRot</td><td rowspan=1 colspan=1>通过对权重和激活进行旋转变换，将离群值&quot;分散&quot;到多个通道，平滑分布</td><td rowspan=1 colspan=1>可与其他算法叠加使用，作为进一步提升精度的备选方案。</td><td rowspan=1 colspan=1>QuaRot.md</td></tr></table>

# 总结建议

1. 优先选用Iterative Smooth算法：运行速度快，精度较高，是大多数场景下的首选方案。  
2. 对称性选择：非对称离群值抑制算法在大多数情况下优于对称方案，但需提前确认推理引擎是否已完成对非对称离群值抑制算法的适配支持。  
3. 参数调优：若Iterative Smooth算法的量化精度未达预期，可通过调整alpha参数进行针对性优化。  
4. 进阶方案：若调整后精度仍无法满足需求，可进一步尝试启用Flex SmoothQuant算法，或通过叠加QuaRot算法实现协同优化。

# 步骤3 量化算法选择

根据量化对象（权重/激活）和比特数选择合适算法。量化算法选择包括表3 权重量化方法对比选择和表4 激活量化方法对比选择两部分。

表 2-3 权重量化方法对比  

<table><tr><td rowspan=1 colspan=1>量化方法</td><td rowspan=1 colspan=1>特点</td><td rowspan=1 colspan=1>量化精度</td><td rowspan=1 colspan=1>量化速度</td><td rowspan=1 colspan=1>适用场景与建议</td></tr><tr><td rowspan=1 colspan=1>minmax</td><td rowspan=1 colspan=1>统计权重张量的最小值和最大值来确定量化范围，方法简单，计算速度快</td><td rowspan=1 colspan=1>低</td><td rowspan=1 colspan=1>快</td><td rowspan=1 colspan=1>INT8量化场景优先推荐。方法简单，速度快，通常能获得不错的精度</td></tr><tr><td rowspan=1 colspan=1>SSZ</td><td rowspan=1 colspan=1>通过迭代搜索最优的量化参数来最小化量化误差</td><td rowspan=1 colspan=1>中</td><td rowspan=1 colspan=1>中</td><td rowspan=1 colspan=1>INT4等低比特量化场景优先推荐。相比minmax，通过更精细的搜索可以获得更高的量化精度，但速度稍慢</td></tr><tr><td rowspan=1 colspan=1>autoround</td><td rowspan=1 colspan=1>通过引入可学习的舍入偏移参数，结合SignSGD优化器自适应调整各权重的舍入方向，训练得到最优的取整补偿</td><td rowspan=1 colspan=1>高</td><td rowspan=1 colspan=1>慢</td><td rowspan=1 colspan=1>当 ssz方法精度不达标时，可以尝试使用autoround 来进一步提升精度，尤其在超低比特条件下寻求最优平衡</td></tr></table>

# 配置示例 (YAML)

在量化配置文件中，权重量化通常在 linear_quant 处理器的 qconfig.weight 部分进行配置：

- type: "linear_quant"  
qconfig:weight:scope: "per_channel" # 量化粒度：per_channeldtype: "int8" # 量化数据类型：int8 或 int4symmetric: true # 是否对称量化：权重量化通常使用对称量化method: "minmax" # 量化方法：minmax、ssz 或 autoround

# 总结建议

1. INT8权重量化：优先使用 minmax 方法，在保证精度的同时速度最快。  
2. INT4等低比特权重量化：优先使用 ssz 方法，若精度不足再尝试 autoround方法。  
3. 量化粒度 (scope)：权重量化建议使用逐通道（per_channel），这会比逐张量（per_tensor）粒度更细，能获得更高的量化精度。  
4. 对称性 (symmetric)：权重量化通常设置为 true（对称量化），计算更简单高效。

表 2-4 激活量化方法对比  

<table><tr><td rowspan=1 colspan=1>量化方法</td><td rowspan=1 colspan=1>特点</td><td rowspan=1 colspan=1>量化精度</td><td rowspan=1 colspan=1>量化速度</td><td rowspan=1 colspan=1>适用场景与建议</td></tr><tr><td rowspan=1 colspan=1>minmax</td><td rowspan=1 colspan=1>统计激活张量的最小值和最大值来确定量化范围，方法简单，计算速度快</td><td rowspan=1 colspan=1>低</td><td rowspan=1 colspan=1>快</td><td rowspan=1 colspan=1>优先推荐。方法简单，速度快，适用于大多数场景</td></tr><tr><td rowspan=1 colspan=1>histogram</td><td rowspan=1 colspan=1>通过分析激活值的直方图分布，自动搜索最优的截断区间，过滤离群值，提高量化精度和模型性能</td><td rowspan=1 colspan=1>高</td><td rowspan=1 colspan=1>慢</td><td rowspan=1 colspan=1>当minmax方法精度不足时，可以尝试使用histogram 来进一步提升精度，但速度稍慢</td></tr></table>

# 激活值量化粒度选择

激活值量化支持多种粒度，对精度和性能有直接影响，具体可参见表2-5。

表 2-5 激活值量化粒度对比  

<table><tr><td rowspan=1 colspan=1>粒度类型</td><td rowspan=1 colspan=1>特点</td><td rowspan=1 colspan=1>适用场景</td></tr><tr><td rowspan=1 colspan=1>per_tensor</td><td rowspan=1 colspan=1>整个张量使用一组量化参数。静态量化。计算简单，性能最好，几乎所有硬件都支持。但当张量内数据分布差异大时，量化误差显著</td><td rowspan=1 colspan=1>追求最佳性能时使用</td></tr><tr><td rowspan=1 colspan=1>per_token</td><td rowspan=1 colspan=1>每个token独立使用一组量化参数。动态量化。量化粒度更细，能获得更高的量化精度，但计算也更复杂，性能较差</td><td rowspan=1 colspan=1>追求更高精度时使用</td></tr><tr><td rowspan=1 colspan=1>pd_mix</td><td rowspan=1 colspan=1>profiling阶段使用per_token，decoding阶段使用per_tensor。混合策略。旨在平衡精度和性能</td><td rowspan=1 colspan=1>需要平衡精度和性能时使用</td></tr></table>

# 配置示例 (YAML)

在量化配置文件中，激活值量化通常在 linear_quant 处理器的 qconfig.act 部分进行配置。

- type: "linear_quant"   
qconfig: act: scope: "per_tensor" # 量化粒度：per_tensor、per_token 或 pd_mix dtype: "int8" # 量化数据类型：int8 或 int4 symmetric: false # 是否对称量化：激活值量化通常使用非对称量化 method: "minmax" # 量化方法：minmax weight: scope: "per_channel" dtype: "int8" symmetric: true method: "minmax"

# 总结建议

1. 量化方法：优先使用minmax方法，当精度不足时可尝试histogram方法。

2. 量化粒度 (scope)：

追求性能时，使用per_tensor（静态量化）。追求精度时，使用per_token（动态量化）。需要平衡精度和性能时，使用pd_mix（混合策略）

3. 对称性 (symmetric)：激活值量化通常设置为false（非对称量化），以更好地适应非零中心的数据分布。

# 步骤4 校准集调整

当算法调整效果有限时，通过优化校准数据来提升量化模型精度。校准集的质量直接影响量化参数的准确性。具体可参考表2-6。

表 2-6 校准集优化策略  

<table><tr><td rowspan=1 colspan=1>调整策略</td><td rowspan=1 colspan=1>具体操作</td><td rowspan=1 colspan=1>优化目的</td></tr><tr><td rowspan=1 colspan=1>增加数据量</td><td rowspan=1 colspan=1>适当增加数据量（建议10-50条样本）</td><td rowspan=1 colspan=1>提升量化参数估计的准确性</td></tr><tr><td rowspan=1 colspan=1>匹配应用场景</td><td rowspan=1 colspan=1>使用与模型应用场景匹配的数据（如中文模型用中文数据，代码模型用代码数据）</td><td rowspan=1 colspan=1>使校准数据更贴近实际应用场景</td></tr><tr><td rowspan=1 colspan=1>平衡数据分布</td><td rowspan=1 colspan=1>从多个数据集中抽取样本混合，平衡数据分布</td><td rowspan=1 colspan=1>提升数据分布的多样性和均衡性</td></tr><tr><td rowspan=1 colspan=1>删除异常数据</td><td rowspan=1 colspan=1>删除导致精度显著下降的校准数据</td><td rowspan=1 colspan=1>减少异常样本对量化参数的干扰</td></tr><tr><td rowspan=1 colspan=1>加入badcase</td><td rowspan=1 colspan=1>加入模型在该数据集上的badcase数据，以更好地反映模型真实输入分布</td><td rowspan=1 colspan=1>帮助量化模型学习困难样本，提升精度</td></tr></table>

# 总结建议

1. 数据量：建议使用10-50条样本，过少可能无法充分估计量化参数，过多则可能增加量化时间。  
2. 场景匹配：优先使用与模型应用场景匹配的数据，确保校准集能代表实际使用场景。

3. 数据质量：及时删除异常数据，避免对量化参数产生负面影响。

4. 困难样本：适当加入badcase数据，有助于提升模型在困难样本上的量化精度。

步骤5 量化回退（最终手段）

当通过算法和校准集调整仍无法达到预期精度时，可针对模型中最敏感层进行回退处理，使其保持高精度（FP16/BF16），从而有效缓解量化带来的精度下降问题。

使用场景

通过步骤1-4调整后，精度仍无法满足精度要求一 需要在精度和性能之间寻求更精细的平衡某些特定层对量化极度敏感，需要保持高精度

操作流程a. 敏感层分析

使用msModelSlim提供的敏感层分析工具识别量化敏感层。详细使用方法请参考《量化敏感层分析使用指南》。

功能说明

自动评估：工具会自动评估模型中线性层对量化操作的敏感程度，并为每个可分析对象生成量化敏感度评分。▪ 决策依据：用户可根据生成的敏感度评分，判断哪些高敏感层需要进行回退处理。

# b. 配置回退

根据第一步敏感层分析的分析结果，在量化策略的YAML配置文件中，通过exclude字段排除需要回退的高敏感层。

# 配置示例

- type: "linear_quant"   
qconfig: act: scope: "per_tensor" dtype: "int8" symmetric: false method: "minmax" weight: scope: "per_channel" dtype: "int8" symmetric: true method: "minmax"   
include: ["\*"]   
exclude: ["\*model.layers.\*.mlp.down_proj\*"] # 回退所有mlp.down_proj层

总结建议

a. 优先回退建议：通过经验得知，mlp.down_proj层通常是量化敏感度最高的层之一，建议将其列为回退的优先对象。  
b. 权衡代价：回退会部分减少量化带来的性能提升和内存节省收益，需要根据具体的业务目标来决定回退的层数和范围。  
c. 回退策略：建议采用“从上到下”的策略，从敏感度最高的层开始，进行逐步回退，以在模型精度与计算性能之间找到最优平衡点。

# ----结束

# 3 推理量化精度调优案例

本章节以Qwen3-32B模型为例，通过W8A8量化场景演示量化精度调优方法。

初始状态：使用全静态量化（per-channel/per-tensor）搭配Smooth Quant离群值抑制算法，量化后模型对话出现乱码，无法正常使用。  
目标状态：对Qwen3-32B模型进行W8A8量化，使量化模型相比浮点模型的精度损失控制在可控范围以内。

# 环境准备

安装 msModelSlim 工具，详情请参见《msModelSlim工具安装指南》。

# 调优步骤

步骤1 确认精度问题可信

在开始调优前，首先排除环境干扰，确保问题真实存在：

● 推理引擎验证：浮点模型在目标推理引擎上能正常复现原始精度。  
● 测评结果检查：量化模型测评输出存在明显异常（对话乱码），确认为量化精度问题。  
● 确定波动范围：AIME25评测数据集当前精度损失异常。

步骤2 调整离群值抑制算法（首要步骤）

初始配置使用Smooth Quant算法导致对话中出现乱码，按照调优策略，依次尝试不同的离群值抑制算法：

表 3-1 离群值抑制算法  

<table><tr><td colspan="1" rowspan="1">离群值抑制算法</td><td colspan="1" rowspan="1">AIME25 精度(%)</td><td colspan="1" rowspan="1">量化时间（s）</td><td colspan="1" rowspan="1">备注</td></tr><tr><td colspan="1" rowspan="1">Smooth Quant</td><td colspan="1" rowspan="1">对话乱码</td><td colspan="1" rowspan="1">326</td><td colspan="1" rowspan="1">初始配置，精度下降明显</td></tr><tr><td colspan="1" rowspan="1">Iterative Smooth（对称/alpha:0.5)</td><td colspan="1" rowspan="1">53.33</td><td colspan="1" rowspan="1">324</td><td colspan="1" rowspan="1">相比SmoothQuant有改善，但精度仍不足</td></tr><tr><td colspan="1" rowspan="1">Iterative Smooth（非对称/alpha:0.5 )</td><td colspan="1" rowspan="1">63.33</td><td colspan="1" rowspan="1">305</td><td colspan="1" rowspan="1">非对称方案精度提升10%，符合预期</td></tr><tr><td colspan="1" rowspan="1">Iterative Smooth（对称/alpha:0.9 )</td><td colspan="1" rowspan="1">66.67</td><td colspan="1" rowspan="1">319</td><td colspan="1" rowspan="1">调整alpha参数后精度进一步提升</td></tr><tr><td colspan="1" rowspan="1">Flex SmoothQuant</td><td colspan="1" rowspan="1">63.33</td><td colspan="1" rowspan="1">1380</td><td colspan="1" rowspan="1">精度与IterativeSmooth（非对称/alpha:0.5）相当，但所需时间更长</td></tr></table>

调优结果：综合考虑精度和量化时间，最终选择 Iterative Smooth（对称/alpha:0.9） 算法，具体分析如下。

1. 精度对比分析

Iterative Smooth（对称/alpha:0.9）精度为 $6 6 . 6 7 \%$ ，在对称方案中最高。相比Iterative Smooth（对称/alpha:0.5）的 $5 3 . 3 3 \%$ ，精度提升13.34个百分点。  
虽然Iterative Smooth（非对称/alpha:0.5）精度为 $6 3 . 3 3 \%$ ，但对称方案（alpha:0.9）精度为 $6 6 . 6 7 \%$ ，已超过非对称方案。  
相比Flex Smooth Quant的 $6 3 . 3 3 \%$ ，精度提升3.34个百分点。

2. 量化时间对比分析

Iterative Smooth（对称/alpha:0.9）量化时间为319秒，与Iterative Smooth（对称/alpha:0.5）的324秒相当。相比Flex Smooth Quant的1380秒，量化时间节省 $7 6 . 9 \%$ ，效率明显提升。

最终决策： 综合以上分析，Iterative Smooth（对称/alpha:0.9） 是当前场景下的最佳选择。

# 步骤3 量化算法选择

在确定离群值抑制算法后，优化量化算法配置。

表 3-2 量化方法对比  

<table><tr><td colspan="1" rowspan="1">权重量化方法</td><td colspan="1" rowspan="1">激活量化粒度</td><td colspan="1" rowspan="1">AIME25精度(%)</td><td colspan="1" rowspan="1">量化时间（s）</td><td colspan="1" rowspan="1">备注</td></tr><tr><td colspan="1" rowspan="1">minmax</td><td colspan="1" rowspan="1">per-tensor（静态量化）</td><td colspan="1" rowspan="1">66.67</td><td colspan="1" rowspan="1">319</td><td colspan="1" rowspan="1">基础配置（基于步骤2结果）</td></tr><tr><td colspan="1" rowspan="1">minmax</td><td colspan="1" rowspan="1">per-token（动态量化）</td><td colspan="1" rowspan="1">80.00</td><td colspan="1" rowspan="1">289</td><td colspan="1" rowspan="1">激活使用per-token后精度提升13.33个百分点</td></tr><tr><td colspan="1" rowspan="1">SSZ</td><td colspan="1" rowspan="1">per-tensor（静态量化）</td><td colspan="1" rowspan="1">63.33</td><td colspan="1" rowspan="1">408</td><td colspan="1" rowspan="1">权重量化使用SSz方法，但静态量化精度下降</td></tr><tr><td colspan="1" rowspan="1">SSZ</td><td colspan="1" rowspan="1">per-token（动态量化）</td><td colspan="1" rowspan="1">70.00</td><td colspan="1" rowspan="1">348</td><td colspan="1" rowspan="1">SSZ + per-token精度低于minmax+per-token</td></tr></table>

调优结果：综合考虑精度和量化时间，最终选择 minmax $^ +$ per-token（动态量化）配置，具体分析如下。

1. 精度对比分析

minmax + per-token（动态量化）精度为 $8 0 . 0 0 \%$ ，相比minmax $^ +$ pertensor（静态量化）的 $6 6 . 6 7 \%$ ，精度提升13.33个百分点。   
ssz + per-tensor（静态量化）精度为 $6 3 . 3 3 \%$ ，相比minmax $^ +$ per-tensor （静态量化）配置下降3.34个百分点，说明ssz方法在该INT8量化场景下效果 不如minmax方法。   
ssz + per-token（动态量化）精度为 $7 0 . 0 0 \%$ ，低于minmax $^ +$ per-token （ $8 0 . 0 0 \%$ ）10个百分点，说明ssz方法在该INT8动态量化场景下也不如 minmax方法。

2. 量化时间对比分析

minmax $^ +$ per-token量化时间为289秒，相比minmax $^ +$ per-tensor（319秒）节省30秒（ $9 . 4 \%$ ），量化效率提升。  
${ \tt S S Z + }$ per-token量化时间为348秒，比minmax $^ +$ per-token多59秒  
（ $2 0 . 4 \%$ ），量化效率较低。

3. 综合对比分析

精度方面：minmax $^ +$ per-token精度最高（80.00%），优于ssz + per-token（ $7 0 . 0 0 \%$ ）和所有静态量化方案。  
量化时间方面：minmax $^ +$ per-token量化时间最短（289秒），相比ssz $^ +$ per-token（348秒）节省59秒（17.0%），相比minmax + per-tensor（319秒）也节省30秒。  
方法复杂度：minmax方法实现简单，计算速度快；ssz方法通过迭代搜索，计算更复杂，INT8量化优先选择minmax算法。

最终决策： 综合以上分析，minmax + per-token（动态量化） 在精度和量化时间两个方面达到最佳平衡。minmax + per-token不仅精度最高（80.00%），比ssz $^ +$ per-token高10个百分点，而且量化时间最短（289秒），比ssz + per-token节省59秒，实现更简单，是当前场景下的最佳选择。相比步骤2的配置（66.67%），精度提升了13.33个百分点（相对提升20%），为后续调优奠定了良好基础。

# 步骤4 校准集调整

步骤3达到80.00%精度后，已满足预设精度要求。为展示完整的调优流程并验证校准集调整的效果，本节在GPQA数据集上进行测试验证。GPQA数据集题目数量更多，能

够更清晰地展现不同配置间的精度差异。本节以Iterative Smooth配合静态量化策略作为基准配置。校准集的质量直接影响量化参数的准确性。

表 3-3 校准集优化策略  

<table><tr><td rowspan=1 colspan=1>调整策略</td><td rowspan=1 colspan=1>具体操作</td><td rowspan=1 colspan=1>校准集变化</td><td rowspan=1 colspan=1>优化目的</td></tr><tr><td rowspan=1 colspan=1>初始校准集</td><td rowspan=1 colspan=1>10条随机样本</td><td rowspan=1 colspan=1>10条</td><td rowspan=1 colspan=1>建立基准配置</td></tr><tr><td rowspan=1 colspan=1>增加数据量</td><td rowspan=1 colspan=1>从10条增加到30条样本</td><td rowspan=1 colspan=1>10→30条</td><td rowspan=1 colspan=1>提升量化参数估计的准确性</td></tr><tr><td rowspan=1 colspan=1>匹配应用场景</td><td rowspan=1 colspan=1>使用中文对话数据替换随机数据</td><td rowspan=1 colspan=1>30条（中文对话）</td><td rowspan=1 colspan=1>使校准数据更贴近实际应用场景</td></tr><tr><td rowspan=1 colspan=1>平衡数据分布</td><td rowspan=1 colspan=1>从GPQA、C-Eval、MMLU等多个数据集抽取样本混合</td><td rowspan=1 colspan=1>30条（多数据集混合）</td><td rowspan=1 colspan=1>提升数据分布的多样性和均衡性</td></tr><tr><td rowspan=1 colspan=1>剔除异常数据</td><td rowspan=1 colspan=1>移除导致量化精度下降的3条异常样本</td><td rowspan=1 colspan=1>30→27条</td><td rowspan=1 colspan=1>减少异常样本对量化参数的干扰</td></tr><tr><td rowspan=1 colspan=1>加入badcase</td><td rowspan=1 colspan=1>加入浮点模型在GPQA上的5个badcase样本</td><td rowspan=1 colspan=1>27→32条</td><td rowspan=1 colspan=1>帮助量化模型学习困难样本，提升精度</td></tr></table>

调优过程：将AISBench测评结果中的badcase样本加入量化校准集，重新生成量化权重。具体操作如下。

# 1. 获取badcase样本

从AISBench测评结果中提取少量badcase样本。例如，一个badcase样本为：

What is the correct answer to this question: Two quantum states with energies E1 and E2 have a lifetime of 10^-9 sec and 10^-8 sec, respectively. We want to clearly distinguish these two energy levels. Which one of the following options could be their energy difference so that they can be clearly resolved?

Choices:   
(A)10^-11 eV   
(B)10^-8 eV   
(C)10^-9 eV   
(D)10^-4 eV   
Format your response as follows: "The correct answer is (insert answer here)"

# 2. 格式转换

JSONL格式：参考 msmodelslim/lab_calib/mix_calib.jsonl，详情请见Link。将文本放在"inputs_pretokenized"字段后，格式如下。  
"inputs_pretokenized":

"What is the correct answer to this question: Two quantum states with energies E1 and E2 have a lifetime of ${ 1 0 \land \_ 9 }$ sec and ${ 1 0 \land \_ 8 }$ sec, respectively. We want to clearly distinguish these two energy levels. Which one of the following options could be their energy difference so that they can be clearly resolved $\mathrm { ? \backslash n \backslash n C h o i c e s . : \backslash n ( A ) } 1 0 \land \mathrm { - } 1 1 \mathrm { e V \backslash n ( B ) } 1 0 \land \mathrm { - } 8 \mathrm { e V \backslash n \backslash n ( C ) } 1 0 \land \mathrm { - } 9 \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 \mathrm { e V \backslash n ( B ) } 1 0 \land \mathrm { - } 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { - } 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land 0 \land \mathrm { - } 1 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land 0 \land \mathrm { - } 1 0 \land 0 \land \mathrm { e V \backslash n ( C ) } 1 0 \land 0 \land \mathrm { - } 1 0 \land 0 \land \mathrm { e V \land n ( C ) } 1 0 0 \land \land 0 \land \mathrm { - } 1 0 0 \land \land 0 \land \mathrm { - }$ $\mathsf { \backslash n ( D ) } 1 0 \land 4 \mathsf { e V }$ nFormat your response as follows: \"The correct answer is (insert answer here)\"" }

JSON格式：参考 msmodelslim/lab_calib/qwen3_cot_w4a4.json，详情请见Link。直接将文本加入字符串列表中即可。

# 3. 重新量化

将调整后的校准集用于量化，重新生成量化权重。

表 3-4 调优结果  

<table><tr><td rowspan=1 colspan=1>量化策略</td><td rowspan=1 colspan=1>GPQA 精度（%）</td><td rowspan=1 colspan=1>备注</td></tr><tr><td rowspan=1 colspan=1>Iterative Smooth + 静态量化</td><td rowspan=1 colspan=1>46.97</td><td rowspan=1 colspan=1>基准配置</td></tr><tr><td rowspan=1 colspan=1>Iterative Smooth +静态量化+badcase调整校准集</td><td rowspan=1 colspan=1>55.56</td><td rowspan=1 colspan=1>相比基准配置精度提升8.59个百分点，说明badcase样本有助于量化模型学习困难样本，提升量化精度</td></tr></table>

# 步骤5 量化回退（备选方案）

量化回退是指将量化敏感层保持为原始浮点精度，以提升量化模型精度。当通过步骤1-4调整后精度仍无法满足要求时，可通过量化回退策略进一步优化。本节在GPQA数据集上验证量化回退的效果，展示完整的调优流程。

# 使用场景

量化回退适用于以下情况：

● 通过步骤1-4调整后，精度仍无法满足精度要求。  
● 需要在精度和性能之间寻求更精细的平衡。  
$\bullet$ 某些特定层对量化极度敏感，需要保持高精度。

# 调优过程

# 1. 敏感层分析

使用msModelSlim提供的敏感层分析工具识别量化敏感层。详细使用方法请参考《量化敏感层分析使用指南》。

# 执行分析命令：

msmodelslim analyze \--model_type Qwen3-32B \--model_path \${model_path}

根据量化敏感度得分从高到低排序，Top敏感层结果如下：

layers.3.mlp.down_proj layers.63.mlp.down_proj layers.2.mlp.down_proj layers.1.mlp.down_proj layers.4.mlp.down_proj layers.6.mlp.down_proj layers.7.mlp.down_proj layers.5.mlp.down_proj layers.0.mlp.down_proj layers.31.mlp.down_proj layers.62.mlp.down_proj layers.5.mlp.gate_proj layers.5.mlp.up_proj layers.32.mlp.down_proj layers.8.mlp.gate_proj layers.8.mlp.up_proj layers.6.mlp.gate_proj layers.6.mlp.up_proj

分析结果：mlp.down_proj 层敏感度排名靠前，是量化难度较大的层类型，应优先考虑回退。

# 2. 修改量化配置

在量化配置YAML中，通过excl   
mlp.down_proj层）   
apiversion: modelslim_v1   
spec:   
process: - type: "iter_smooth" alpha: 0.9 scale_min: 1e-5 symmetric: True enable_subgraph_type: - 'norm-linear' - 'linear-linear' - 'ov' - 'up-down' include: - "\*" - type: "linear_quant" qconfig: act: scope: "per_tensor" dtype: "int8" symmetric: false method: "minmax" weight: scope: "per_channel" dtype: "int8" symmetric: true method: "minmax" include: - "\*" exclude: - 'model.layers.3.mlp.down_proj' - 'model.layers.63.mlp.down_proj' - 'model.layers.2.mlp.down_proj' - 'model.layers.1.mlp.down_proj' - 'model.layers.4.mlp.down_proj' - 'model.layers.6.mlp.down_proj' - 'model.layers.7.mlp.down_proj' - 'model.layers.5.mlp.down_proj' - 'model.layers.0.mlp.down_proj'   
save:

- type: "ascendv1_saver" part_file_size: 4

# 3. 重新生成量化权重

使用修改后的配置重新进行量化，生成包含回退层的量化模型。

表 3-5 调优结果  

<table><tr><td rowspan=1 colspan=1>量化策略</td><td rowspan=1 colspan=1>GPQA 精度（%）</td><td rowspan=1 colspan=1>备注</td></tr><tr><td rowspan=1 colspan=1>Iterative Smooth +静态量化</td><td rowspan=1 colspan=1>46.97</td><td rowspan=1 colspan=1>基准配置</td></tr><tr><td rowspan=1 colspan=1>Iterative Smooth +静态量化+回退前9层</td><td rowspan=1 colspan=1>51.51</td><td rowspan=1 colspan=1>相比基准配置精度提升4.54个百分点，说明回退量化敏感层能有效提升量化精度，但会带来-定的性能开销和模型大小增加</td></tr></table>

# ----结束

# 最终配置总结

调优路径回顾

表 3-6 调优步骤回顾  

<table><tr><td rowspan=1 colspan=1>步骤</td><td rowspan=1 colspan=1>关键操作</td><td rowspan=1 colspan=1>AIME25 精度(%)</td><td rowspan=1 colspan=1>精度提升</td><td rowspan=1 colspan=1>备注</td></tr><tr><td rowspan=1 colspan=1>初始状态</td><td rowspan=1 colspan=1>SmoothQuant +minmax + 静态量化</td><td rowspan=1 colspan=1>乱码</td><td rowspan=1 colspan=1></td><td rowspan=1 colspan=1>初始配置，无法正常使用</td></tr><tr><td rowspan=1 colspan=1>步骤2</td><td rowspan=1 colspan=1>IterativeSmooth（对称/alpha:0.9 )</td><td rowspan=1 colspan=1>66.67%</td><td rowspan=1 colspan=1>+66.67%</td><td rowspan=1 colspan=1>离群值抑制算法优化，解决乱码问题</td></tr><tr><td rowspan=1 colspan=1>步骤3</td><td rowspan=1 colspan=1>minmax +per-token（动态量化）</td><td rowspan=1 colspan=1>80.00%</td><td rowspan=1 colspan=1>+13.33%</td><td rowspan=1 colspan=1>激活量化粒度优化，达到精度要求</td></tr></table>

# 说明

步骤3达到 $8 0 . 0 0 \%$ 精度后，已满足预设精度要求。步骤4和步骤5在GPQA数据集上进行验证，展示校准集调整和量化回退的调优效果。

# 最终配置

算法配置：离群值抑制算法Iterative Smooth（对称/alpha:0.9）

量化配置：

权重量化：minmax 方法，per_channel 粒度，int8 数据类型，对称量化。激活量化：minmax 方法，per_token 粒度（动态量化），int8 数据类型，对称量化。

调优经验总结

# a. 离群值抑制算法的优化对精度提升具有关键作用

将算法由Smooth Quant切换到Iterative Smooth（对称/alpha:0.9）后，模型精度从“乱码”提升至 $6 6 . 6 7 \%$ ，使量化模型具备基本可用性。

# b. 激活量化策略的选择直接影响模型性能表现

从 per-tensor（静态量化）切换为 per-token（动态量化）后，模型精度由$6 6 . 6 7 \%$ 提升至 $8 0 . 0 0 \%$ ，提升 13.33 个百分点（相对提升约 20%）。但需注意，动态量化可能带来一定的推理性能损耗。

# c. 量化算法的选择对精度与效率有显著影响

在 INT8 量化场景下，minmax 方法相比 ssz 方法表现更优：不仅精度更高（ $8 0 . 0 0 \%$ vs $7 0 . 0 0 \%$ ），且量化耗时更短（289 秒 vs 348 秒），实现更简单，是当前场景下的推荐方案。

# d. 校准数据集的构建质量对量化结果有重要影响

在 GPQA 数据集上，通过加入 badcase 样本优化校准集，模型精度从$4 6 . 9 7 \%$ 提升至 $5 5 . 5 6 \%$ ，提升了 8.59 个百分点。这表明使用与目标场景匹配的数据，尤其是困难样本，可有效提升量化效果。

# e. 量化回退策略可作为精度保障的补充手段

在 GPQA 数据集测试中，通过回退 9 个量化敏感层，模型精度由 46.97% 提升至 $5 1 . 5 1 \%$ ，提升了 4.54 个百分点。尽管该策略能在一定程度上提高模型精度，但会增加模型体积和推理开销，建议作为最后的优化手段使用。