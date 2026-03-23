MindStudio 8.3.0

# mstx API 参考

文档版本 01  
发布日期 2026-01-19

![](images/63fad584668ad54fe65be7513de93684f38690cc96d82517aaf1965329f03993.jpg)

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

1 接口简介..............   
2 mstxGetToolId..   
3 mstxMarkA.   
4 mstxRangeStartA.   
5 mstxRangeEnd.. 10   
6 mstxDomainCreateA.. 12   
7 mstxDomainDestroy....... 14   
8 mstxDomainMarkA. 15   
9 mstxDomainRangeStartA. 17   
10 mstxDomainRangeEnd.. 19   
11 mstxMemHeapRegister.. 21   
12 mstxMemRegionsRegister......... 23   
13 mstxMemRegionsUnregister.. 25   
14 mstxMemHeapUnregister.. 27

# 接口简介

本节介绍mstx打点接口。可以自定义采集时间段或者关键函数的开始和结束时间点，识别关键函数或迭代等信息，对性能和算子问题快速定界。

默认情况下mstx API无任何功能，需要在用户应用程序中调用mstx API后，根据不同场景使能mstx打点功能，例如使用msprof命令行采集时配置--msproftx=on、使用AscendCL API采集时配置ACL_PROF_MSPROFTX以及Ascend PyTorch Profiler接口采集时配置mstx=True等。

● 库文件libms_tools_ext.so路径：\${INSTALL_DIR}/lib64/。使用头文件编译时，用户程序编译时需链接dl库。头文件ms_tools_ext.h路径：\${INSTALL_DIR}/include/mstx。

\${INSTALL_DIR}请替换为CANN软件安装后文件存储路径。以root用户安装为例，则安装后文件存储路径为：/usr/local/Ascend/cann。

# 接口列表

表 1-1 MindStudio mstx 接口列表  

<table><tr><td colspan="1" rowspan="1">接口名称</td><td colspan="1" rowspan="1">功能简介</td></tr><tr><td colspan="1" rowspan="1">2 mstxGetToolld</td><td colspan="1" rowspan="1">用于获取当前劫持mstx接口的工具ID。</td></tr><tr><td colspan="1" rowspan="1"> 3 mstxMarkA</td><td colspan="1" rowspan="1">标识瞬时事件。</td></tr><tr><td colspan="1" rowspan="1"> 4 mstxRangeStartA</td><td colspan="1" rowspan="1">标识时间段事件的开始。</td></tr><tr><td colspan="1" rowspan="1">5 mstxRangeEnd</td><td colspan="1" rowspan="1">标识时间段事件的结束。</td></tr><tr><td colspan="1" rowspan="1">6 mstxDomainCreateA</td><td colspan="1" rowspan="1">创建自定义domain。</td></tr><tr><td colspan="1" rowspan="1"> 7 mstxDomainDestroy</td><td colspan="1" rowspan="1">销毁指定的domain，销毁后的domain不能再次使用，需要重新创建。</td></tr><tr><td colspan="1" rowspan="1">8 mstxDomainMarkA</td><td colspan="1" rowspan="1">在指定的domain内，标记瞬时事件。</td></tr><tr><td colspan="1" rowspan="1">9 mstxDomainRangeStartA</td><td colspan="1" rowspan="1">在指定的domain内，标识时间段事件的开始。</td></tr><tr><td colspan="1" rowspan="1">10 mstxDomainRangeEnd</td><td colspan="1" rowspan="1">在指定的domain内，标识时间段事件的结束。</td></tr><tr><td colspan="1" rowspan="1"> 11 mstxMemHeapRegister</td><td colspan="1" rowspan="1">注册内存池。</td></tr><tr><td colspan="1" rowspan="1">12 mstxMemRegionsRegister</td><td colspan="1" rowspan="1">注册内存池二次分配。</td></tr><tr><td colspan="1" rowspan="1">13 mstxMemRegionsUnregister</td><td colspan="1" rowspan="1">注销内存池二次分配。</td></tr><tr><td colspan="1" rowspan="1">14 mstxMemHeapUnregister</td><td colspan="1" rowspan="1">注销内存池时，与之关联的Regions将一并被注销。</td></tr></table>

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2 推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

# 功能说明

用于获取当前劫持mstx接口的工具ID，工具ID宏定义如下：

#define MSTX_TOOL_INVALID_ID 0x0 // 无效值0，表示无工具拉起程序  
#define MSTX_TOOL_MSPROF_ID $0 \times 1 0 0 0$ // $0 \times 1 0 0 0$ ，表示程序由《msprof模型调优工具》或《MSPTI调  
优工具》工具拉起  
#define MSTX_TOOL_MSOPPROF_ID 0x1001 // 0x1001，表示程序由算子调优（msProf）工具拉起  
#define MSTX_TOOL_MSSANITIZER_ID $0 \times 1 0 0 2$ // $0 \times 1 0 0 2$ ，表示程序由异常检测（msSanitizer）工具拉起  
#define MSTX_TOOL_MSLEAKS_ID $0 \times 1 0 0 3$ // $0 \times 1 0 0 3$ ，表示程序由《msLeaks内存泄漏检测工具》拉起

# 函数原型

# 参数说明

表 2-1 参数说明  

<table><tr><td rowspan=1 colspan=1>参数</td><td rowspan=1 colspan=1>输入/输出</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>id</td><td rowspan=1 colspan=1>输出</td><td rowspan=1 colspan=1>作为出参，返回当前劫持mstx接口的工具ID。数据类型：uint64*。</td></tr></table>

# 返回值

# 调用示例

# 无

# 3 mstxMarkA

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

功能说明

标识瞬时事件。

函数原型

void mstxMarkA(const char \*message, aclrtStream stream)

# 参数说明

表 3-1 参数说明  

<table><tr><td rowspan=1 colspan=1>参数名</td><td rowspan=1 colspan=1>输入/输出</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>message</td><td rowspan=1 colspan=1>输入</td><td rowspan=1 colspan=1>打点携带信息字符串指针。传入的message字符串长度要求：·MSPTI场景：不能超过255字节。非MSPTI场景（例如msprof命令行、Ascend PyTorchProfiler）：不能超过156字节。说明message不能传入空指针。</td></tr><tr><td rowspan=1 colspan=1>stream</td><td rowspan=1 colspan=1>输入</td><td rowspan=1 colspan=1>用于执行打点任务的stream。·配置为nullptr时，只标记Host侧的瞬时事件。配置为有效的stream时，标识Host侧和对应Device侧的瞬时事件。</td></tr></table>

无

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

功能说明

mstx range指定范围能力的起始位置标记。

# 函数原型

$\mathsf { C } / \mathsf { C } + +$

mstxRangeId mstxRangeStartA(const char \*message, aclrtStream stream)

Python：

mstx.range_start(message, stream)

# 参数说明

表 4-1 参数说明  

<table><tr><td rowspan=1 colspan=1>参数</td><td rowspan=1 colspan=1>输入/输出</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>message</td><td rowspan=1 colspan=1>输入</td><td rowspan=1 colspan=1>message为标记的文字，携带打点信息。C/C++中数据类型：const char *。Python中，message为字符串。默认None。传入的message字符串长度要求：·MSPTI场景：不能超过255字节。·非MSPTI场景（例如msprof命令行、Ascend PyTorchProfiler）：不能超过156字节。说明message不能传入空指针。</td></tr><tr><td rowspan=1 colspan=1>stream</td><td rowspan=1 colspan=1>输入</td><td rowspan=1 colspan=1>stream表示使用mark的线程。C/C++中数据类型：aclrtStream。Python中stream是aclrtStream对象。默认None。·配置为nullptr时，只标记Host侧的瞬时事件。配置为有效的stream时，标识Host侧和对应Device侧的瞬时事件。</td></tr></table>

# 返回值

如果返回0，则表示失败。

调用示例

$\mathsf { C } / \mathsf { C } + +$ 调用方法：   
bool RunOp()   
{   
// create op desc   
...   
const char \*message $=$ "h1";   
mstxRangeId id $=$ mstxRangeStartA(message, NULL); ...   
// Run op   
if   
(!opRunner.RunOp()) {   
ERROR_LOG("Run   
op failed");   
return false; }mstxRangeEnd(id);   
...

Python调用方法一：

通过Python API接口，以C/C++语言实现相关接口内容并编译生成so，相关so在PYTHONPATH中可以被Python直接引用。

import mstx  
mstx.range_start("aaa")  
print(1)  
mstx.range_end(1)  
import torch  
import torch_npu  
${ \sf a } =$ torch.Tensor([1,2,3,4]).npu()  
$\flat =$ torch.Tensor([1,2,3,4]).npu()  
hi_str $=$ "hi"  
hello_str $=$ "hello"  
hi_id $=$ mstx.range_start(hi_str, None)  
${ \mathsf { C } } = { \mathsf { a } } + { \mathsf { b } }$   
hello_id $=$ mstx.range_start(hello_str, stream $| =$ None)  
${ \mathsf { d } } = { \mathsf { a } } - { \mathsf { b } }$   
mstx.range_end(hi_id)  
$\mathtt { e } = \mathtt { a } ^ { \star }$ b  
mstx.range_end(hello_id)

Python调用方法二：

直接使用Python开发，通过ctypes.CDLL("libms_tools_ext.so")直接引用原mstx的so文件，并使用其中提供的API。

import mstx  
import torch  
import torch_npu  
import acl  
import sys  
import ctypes  
lib $=$ ctypes.CDLL("libms_tools_ext.so")  
# 定义函数的参数类型和返回类型  
lib.mstxRangeStartA.argtypes $=$ [ctypes.c_char_p, ctypes.c_void_p]  
lib.mstxRangeStartA.restype $=$ ctypes.c_uint64  
lib.mstxRangeEnd.argtypes $=$ [ctypes.c_uint64]  
lib.mstxRangeEnd.restype $=$ None  
a $=$ torch.Tensor([1,2,3,4]).npu()  
$\flat =$ torch.Tensor([1,2,3,4]).npu()  
# 创建一个ctypes.c_char_p指针  
hi_str $=$ b"hi"  
hi_ptr $=$ ctypes.c_char_p(hi_str)  
hi_id $=$ ctypes.c_uint64()  
# 创建一个ctypes.c_char_p指针  
hello_str $=$ b"hello"  
hello_ptr $=$ ctypes.c_char_p(hello_str)  
hello_id $=$ ctypes.c_uint64()  
# 调用函数  
hi_id.value $=$ lib.mstxRangeStartA(hi_ptr, None)  
${ \mathsf { C } } = { \mathsf { a } } + { \mathsf { b } }$   
hello_id.value $=$ lib.mstxRangeStartA(hello_ptr, None)  
${ \mathsf { d } } = { \mathsf { a } } - { \mathsf { b } }$   
lib.mstxRangeEnd(hi_id)  
$\mathtt { e } = \mathtt { a } ^ { \star } \mathtt { b }$   
lib.mstxRangeEnd(hello_id)

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

功能说明

mstx range指定范围能力的结束位置标记。

# 函数原型

$\mathsf { C } / \mathsf { C } + +$ 函数原型：

void mstxRangeEnd(mstxRangeId id)

# Python函数：

mstx.range_end(range_id)

# 参数说明

表 5-1 参数说明  

<table><tr><td colspan="1" rowspan="1">参数</td><td colspan="1" rowspan="1">输入/输出</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">id （ C/C++ )</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">通过mstxRangeStartA返回的ID(C/C++）。</td></tr><tr><td colspan="1" rowspan="1">range_id（Python）</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">通过mstx.range_start返回的range_id（Python）。</td></tr></table>

返回值

如果返回0，则表示失败。

调用示例

C/C++调用：mstxRangeEnd接口需要与mstxRangeStartA配合使用，具体示例请参考C/C++调用方法。

Python调用：mstx.range_end接口需要与mstx.range_start配合使用，具体示例请参考Python调用方法。

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

# 功能说明

创建自定义的mstx域。

domain（域）：用于对打点数据进行划分，便于用户自定义管理打点数据，不指定domain的打点数据均属于默认域（域名：default）。默认情况下，所有打点数据均属于默认域。

# 函数原型

mstxDomainHandle_t mstxDomainCreateA(const char\* name)

# 参数说明

表 6-1 参数说明  

<table><tr><td rowspan=1 colspan=1>参数名</td><td rowspan=1 colspan=1>输入/输出</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>name</td><td rowspan=1 colspan=1>输入</td><td rowspan=1 colspan=1>要创建的域的名称。·数据类型：const char *。·默认域名为globalDomain。最大长度为1023字节，仅支持数字、大小写字母和_符号。. MSPTI场景：不能超过255字节。非MSPTI场景（例如msprof命令行、AscendPyTorch Profiler）：不能超过1024字节。</td></tr></table>

返回值

返回有效的domain句柄，表示接口执行成功；返回nullptr，表示接口执行失败。

# 调用示例

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

功能说明

销毁指定的domain，销毁后的domain不能再次使用，需要重新创建。

函数原型

void mstxDomainDestroy (mstxDomainHandle_t domain)

参数说明

表 7-1 参数说明  

<table><tr><td rowspan=1 colspan=1>参数名</td><td rowspan=1 colspan=1>输入/输出</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>domain</td><td rowspan=1 colspan=1>输入</td><td rowspan=1 colspan=1>指定要销毁的domain句柄。</td></tr></table>

返回值

无

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

功能说明

函数原型

参数说明

在指定的domain内，标记瞬时事件。

如果传入的domain已被销毁，日志打印告警提示，接口不再执行打点流程。

void mstxDomainMarkA(mstxDomainHandle_t domain, const char \*message, aclrtStream stream)

表 8-1 参数说明  

<table><tr><td colspan="1" rowspan="1">参数名</td><td colspan="1" rowspan="1">输入/输出</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">domain</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">指定的domain句柄。</td></tr><tr><td colspan="1" rowspan="1">message</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">打点携带信息字符串指针。传入的message字符串长度要求：·MSPTI场景：不能超过255字节。非MSPTI场景（例如msprof命令行、Ascend PyTorchProfiler）：不能超过156字节。</td></tr><tr><td colspan="1" rowspan="1">stream</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">用于执行打点任务的stream。·配置为nullptr时，只标记Host侧的瞬时事件。配置为有效的stream时，标记Host侧和对应Device侧的瞬时事件。</td></tr></table>

返回值

无

# 9 mstxDomainRangeStartA

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

# 功能说明

在指定的domain内，标识时间段事件的开始。

如果传入的domain已被销毁，日志打印告警提示，接口不再执行打点流程。

# 函数原型

mstxRangeId mstxDomainRangeStartA(mstxDomainHandle_t domain, const char \*message, aclrtStream stream)

参数说明

表 9-1 参数说明  

<table><tr><td colspan="1" rowspan="1">参数名</td><td colspan="1" rowspan="1">输入/输出</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">domain</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">指定的domain句柄。</td></tr><tr><td colspan="1" rowspan="1">message</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">打点携带信息字符串指针。传入的message字符串长度要求：·MSPTI场景：不能超过255字节。非MSPTI场景（例如msprof命令行、Ascend PyTorchProfiler）：不能超过156字节。</td></tr><tr><td colspan="1" rowspan="1">stream</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">用于执行打点任务的stream。·配置为nullptr时，只标记Host侧的瞬时事件。配置为有效的stream时，标识Host侧和对应Device侧的瞬时事件。</td></tr></table>

range_id：标识该range，如果打点失败或domain已销毁，则返回0。

# 10 mstxDomainRangeEnd

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

# 功能说明

在指定的domain内，标识时间段事件的结束。

如果传入的domain已被销毁，日志打印告警提示，接口不再执行打点流程。

# 函数原型

void mstxDomainRangeEnd(mstxDomainHandle_t domain, mstxRangeId id)

# 参数说明

表 10-1 参数说明  

<table><tr><td rowspan=1 colspan=1>参数名</td><td rowspan=1 colspan=1>输入/输出</td><td rowspan=1 colspan=1>说明</td></tr><tr><td rowspan=1 colspan=1>domain</td><td rowspan=1 colspan=1>输入</td><td rowspan=1 colspan=1>指定的domain句柄。</td></tr><tr><td rowspan=1 colspan=1>id</td><td rowspan=1 colspan=1>输入</td><td rowspan=1 colspan=1>通过mstxDomainRangeStartA接口返回的id。</td></tr></table>

# 返回值

无

# mstxMemHeapRegister

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

功能说明

注册内存池。用户在调用该接口注册内存池时，需确保该内存已提前申请。

# 函数原型

mstxMemHeapHandle_t mstxMemHeapRegister(mstxDomainHandle_t domain, mstxMemHeapDesc_t const \*desc)

# 参数说明

表 11-1 参数说明  

<table><tr><td colspan="1" rowspan="1">参数</td><td colspan="1" rowspan="1">输入/输出</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">domain</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">为globalDomain或6mstxDomainCreateA返回的句柄。数据类型：const char *。</td></tr><tr><td>desc</td><td>输入</td><td>typedef enum mstxMemHeapUsageType { /* @brief此堆内存作为内存池使用 *使用此使用方式注册的堆内存，需要使用二次分配注册后才可以访问 */ MSTX_MEM_HEAP_USAGE_TYPE_SUB_ALLOCATOR = 0, } mstxMemHeapUsageType; /** @brief堆内存的类型 *此处的“类型”是指通过何种方式来描述堆内存指针。当前仅支持线性排布 的 *内存，但此处保留日后支持更多内存类型的扩展能力。比如某些API返回 *多个句柄来描述内存范围，或者一些高维内存需要使用stride、tiling或 * interlace来描述 */ typedef enum_mstxMemType { /** @brief 标准线性排布的虚拟内存 *此时mstxMemHeapRegister接收mstxMemVirtualRangeDesc_t类型的 描述 */ MSTX_MEM_TYPE_VIRTUAL_ADDRESS = 0, } mstxMemType; typedef struct mstxMemVirtualRangeDesc_t { uint32_tdeviceld;//内存区域对应的设备 ID void const *ptr； //内存区域的起始地址 uint64_t size；//内存区域的长度 } mstxMemVirtualRangeDesc_t; typedef struct mstxMemHeapDesc_t { mstxMemHeapUsageTypeusage；_//堆内存的使用方式 mstxMemType type; //堆内存的类型</td></tr></table>

返回值

内存池对应的句柄。

调用示例

mstxMemVirtualRangeDesc_t rangeDesc $=$ {}; rangeDesc.deviceId $=$ deviceId; // 设备编号 rangeDesc.ptr $=$ gm; // 注册的内存池gm首地址 rangeDesc.size $= 1 0 2 4$ ; // 内存池大小 heapDesc.typeSpecificDesc $=$ &rangeDesc; mstxMemHeapDesc_t heapDesc{}; mstxMemHeapHandle_t memPool $=$ mstxMemHeapRegister(globalDomain, &heapDesc); // 注册内存池

# 12 mstxMemRegionsRegister

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

# 功能说明

注册内存池二次分配。用户需保证RegionsRegister的内存位于11mstxMemHeapRegister注册的范围内，否则工具会提示越界读写。

# 函数原型

# 参数说明

表 12-1 参数说明  

<table><tr><td colspan="1" rowspan="1">参数</td><td colspan="1" rowspan="1">输入/输出</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">domain</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">为globalDomain或6mstxDomainCreateA返回的句柄。数据类型：const char *。</td></tr><tr><td colspan="1" rowspan="1">desc</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">内存池待二次分配的内存区域描述信息，不能为空。struct mstxMemRegion_st;typedef struct mstxMemRegion_stmstxMemRegion_t;typedef mstxMemRegion_t*mstxMemRegionHandle_t;typedef structmstxMemRegionsRegisterBatch_t {mstxMemHeapHandle_t heap; //要进行二次分配的内存池句柄mstxMemType regionType；//内存区域的内存类型size_t regionCount;//内存区域的个数void const *regionDescArray； //内存区域描述数据mstxMemRegionHandle_t*regionHandleArrayOut；//返回的注册二次分配得到的句柄数组} mstxMemRegionsRegisterBatch_t;</td></tr></table>

返回值

无

调用示例

mstxMemRegionsRegisterBatch_t regionsDesc{};   
regionsDesc.heap $=$ memPool;   
regionsDesc.regionType $=$ MSTX_MEM_TYPE_VIRTUAL_ADDRESS;   
regionsDesc.regionCount $= 1$ ;   
regionsDesc.regionDescArray $=$ rangesDesc;   
regionsDesc.regionHandleArrayOut $=$ regionHandles;   
mstxMemRegionsRegister(globalDomain, ®ionsDesc); // 二次分配注册

# 13 mstxMemRegionsUnregister

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

功能说明

注销内存池二次分配。

# 函数原型

void mstxMemRegionsUnregister(mstxDomainHandle_t domain, mstxMemRegionsUnregisterBatch_t const \*desc)

# 参数说明

表 13-1 参数说明  

<table><tr><td colspan="1" rowspan="1">参数</td><td colspan="1" rowspan="1">输入/输出</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">domain</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">为globalDomain或6mstxDomainCreateA返回的句柄。数据类型：const char *。</td></tr><tr><td colspan="2" rowspan="1">desc                        输入</td><td colspan="1" rowspan="1">输入的描述信息必须是某一次11mstxMemHeapRegister的输入描述信息，否则工具将打印提示错误。typedef enummstxMemRegionRefType{//通过指针描述内存引用MSTX_MEM_REGION_REF_TYPE_POINTER=0,//通过句柄描述内存引用MSTX_MEM_REGION_REF_TYPE_HANDLE} mstxMemRegionRefType;typedef struct mstxMemRegionRef_t {mstxMemRegionRefType refType;//描述内存引用的方式union{voidconst*pointer；//当前内存引用通过指针描述时，此处保存内存区域指针mstxMemRegionHandle_t handle; //当内存引用通过句柄描述时，此处保存内存区域的句柄}} mstxMemRegionRef_t;typedef struct mstxMemRegionsUnregisterBatch_t {size_trefCount；//内存引用的个数mstxMemRegionRef_t const *refArray; //要注销的内存区域引用} mstxMemRegionsUnregisterBatch_t;</td></tr></table>

返回值

调用示例

无

# 14 mstxMemHeapUnregister

产品支持情况  

<table><tr><td rowspan=1 colspan=1>产品</td><td rowspan=1 colspan=1>是否支持</td></tr><tr><td rowspan=1 colspan=1>Atlas A3 训练系列产品/Atlas A3推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas A2训练系列产品/Atlas A2推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 200I/500 A2推理产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 推理系列产品</td><td rowspan=1 colspan=1>√</td></tr><tr><td rowspan=1 colspan=1>Atlas 训练系列产品</td><td rowspan=1 colspan=1>√</td></tr></table>

功能说明

注销内存池时，与之关联的Regions将一并被注销。

函数原型

void mstxMemHeapUnregister(mstxDomainHandle_t domain, mstxMemHeapHandle_t heap)

参数说明

表 14-1 参数说明  

<table><tr><td colspan="1" rowspan="1">参数</td><td colspan="1" rowspan="1">输入/输出</td><td colspan="1" rowspan="1">说明</td></tr><tr><td colspan="1" rowspan="1">domain</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">domain为内存池所属的域，为globalDomain或6mstxDomainCreateA返回的句柄。数据类型：const char *。</td></tr><tr><td colspan="1" rowspan="1">heap</td><td colspan="1" rowspan="1">输入</td><td colspan="1" rowspan="1">heap为需要注销内存池的句柄，为11 mstxMemHeapRegister的返回值。struct mstxMemHeap_st;typedef struct mstxMemHeap_stmstxMemHeap_t;typedef mstxMemHeap_t*mstxMemHeapHandle_t;</td></tr></table>

返回值

无

调用示例

mstxMemHeapDesc_t heapDesc{}; mstxMemHeapHandle_t memPool $=$ mstxMemHeapRegister(globalDomain, &heapDesc); // 注册内存池 mstxMemHeapUnregister(globalDomain, memPool); // 注销内存池