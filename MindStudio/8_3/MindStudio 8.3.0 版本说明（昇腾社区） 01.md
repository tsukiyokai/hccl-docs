# MindStudio8.3.0版本说明（昇腾社区）

文档版本 01  
发布日期 2026-01-19

![](images/0a311cd425be16bc5bfed7dbed301ce6c00cd82c7c4fc5edd1a06463ed7e92ca.jpg)

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

1 版本配套关系...

2 文件校验标准.... 3

3 特性变更. 4

3.1 新增特性... 4

3.2 删除特性... 4

4 漏洞修补列表..

# 版本配套关系

# 版本配套关系

表 1-1 版本配套关系  

<table><tr><td colspan="1" rowspan="1">MindStudio版本</td><td colspan="1" rowspan="1">CANN版本</td><td colspan="1" rowspan="1">发布时间</td></tr><tr><td colspan="1" rowspan="1">商用版：8.3.0社区版：8.3.0</td><td colspan="1" rowspan="1">商用版：8.5.0社区版：8.5.0</td><td colspan="1" rowspan="1">2026年1月</td></tr><tr><td colspan="1" rowspan="1">商用版：8.2.RC1社区版：8.2.RC1</td><td colspan="1" rowspan="1">商用版：8.3.RC1社区版：8.3.RC1</td><td colspan="1" rowspan="1">2025年10月</td></tr><tr><td colspan="1" rowspan="1">商用版：8.1.RC1社区版：8.1.RC1</td><td colspan="1" rowspan="1">商用版：8.2.RC1社区版：8.2.RC1</td><td colspan="1" rowspan="1">2025年7月</td></tr><tr><td colspan="1" rowspan="1">商用版：8.0.RC1社区版：8.0.RC1</td><td colspan="1" rowspan="1">商用版：8.1.RC1社区版：8.1.RC1</td><td colspan="1" rowspan="1">2025年4月</td></tr><tr><td colspan="1" rowspan="1">商用版：7.0.0社区版：7.0.0</td><td colspan="1" rowspan="1">商用版：8.0.0社区版：8.0.0.beta1</td><td colspan="1" rowspan="1">2024年12月</td></tr><tr><td colspan="1" rowspan="1">商用版：7.0.RC3.1</td><td colspan="1" rowspan="1">商用版：8.0.RC3.20</td><td colspan="1" rowspan="1">2024年12月</td></tr><tr><td colspan="1" rowspan="1">商用版：7.0.RC3社区版：7.0.RC3</td><td colspan="1" rowspan="1">商用版：8.0.RC3社区版：8.0.RC3.beta1</td><td colspan="1" rowspan="1">2024年9月</td></tr><tr><td colspan="1" rowspan="1">7.0.RC2</td><td colspan="1" rowspan="1">商用版：8.0.RC2社区版：8.0.RC2.beta1</td><td colspan="1" rowspan="1">2024年7月</td></tr><tr><td colspan="1" rowspan="1">7.0.RC1</td><td colspan="1" rowspan="1">商用版：8.0.RC1社区版：8.0.RC1.beta1</td><td colspan="1" rowspan="1">2024年4月</td></tr><tr><td colspan="1" rowspan="1">6.0.0</td><td colspan="1" rowspan="1">商用版：7.0.0社区版：7.0.1.beta1（推荐）或7.0.0.beta1</td><td colspan="1" rowspan="1">2024年1月</td></tr><tr><td colspan="1" rowspan="1">5.0.0</td><td colspan="1" rowspan="1">商用版：6.0.1社区版：6.0.1.alpha001</td><td colspan="1" rowspan="1">2023年1月</td></tr></table>

兼容性说明

无

# 2 文件校验标准

# 说明

MindStudio默认使用普通用户安装，且推荐使用普通用户进行操作，不建议使用root用户进行操作。

<table><tr><td rowspan=1 colspan=1>文件类型</td><td rowspan=1 colspan=1>校验标准</td></tr><tr><td rowspan=1 colspan=1>输入数据文件</td><td rowspan=1 colspan=1>确保输入的数据文件存在且可读，路径长度适中，文件大小适中，且非软链接。该文件的属主为root用户或当前用户，group和other用户组均不可写。</td></tr><tr><td rowspan=1 colspan=1>输入文件夹</td><td rowspan=1 colspan=1>·确保输入的文件夹存在、可读、路径长度适中且非软链接。文件夹的属主为root用户或当前用户，group·和other用户组均不可写。输入文件夹内的文件也需满足存在、可读、文件大小适中且非软链接的要求。文件的属主为root用户或当前用户，group和other用户组均不可写。</td></tr><tr><td rowspan=1 colspan=1>输出文件路径</td><td rowspan=1 colspan=1>输出文件的路径必须存在，且不能为软链接。该目录的属主为当前用户，other用户组不可写。</td></tr><tr><td rowspan=1 colspan=1>输出文件</td><td rowspan=1 colspan=1>输出文件不能为软链接，其属主为当前用户。此外，输出文件的权限不大于640，这意味着除了属主以外的任何人（包括同组用户和其他用户）都不能读取或写入该文件。</td></tr><tr><td rowspan=1 colspan=1>输出路径</td><td rowspan=1 colspan=1>输出路径需要确保可写，且不能为软链接。该路径的属主为root或当前用户，group和other用户组均不可写。</td></tr></table>

# 3 特性变更

新增特性删除特性

# 3.1 新增特性

<table><tr><td rowspan=1 colspan=1>工具名称</td><td rowspan=1 colspan=1>新增特性</td></tr><tr><td rowspan=1 colspan=1>性能调优工具</td><td rowspan=1 colspan=1>·服务化调优工具支持非侵入式的调用链信息采集。msprof支持Stream容量扩容到32K的性能数据关联。msprof支持aclgraph场景下呈现算子的shape信息。</td></tr><tr><td rowspan=1 colspan=1>算子开发工具</td><td rowspan=1 colspan=1>·msProf op支持L2Cache Evict情况分析。·msDebug支持基于coreDump文件回溯异常代码行。</td></tr></table>

# 3.2 删除特性

<table><tr><td>工具名称</td><td>删除特性</td></tr><tr><td>算子及模型速查工 具</td><td>从MindStudio 8.3.0版本及后续版本删除。</td></tr></table>

# 4

# 漏洞修补列表

MindStudio 8.3.0 漏洞修补列表.xlsx