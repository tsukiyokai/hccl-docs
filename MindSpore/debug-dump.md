# Dump功能调试

## 功能演进

MindSpore Dump功能已陆续迁移到[msprobe工具](https://atomgit.com/Ascend/mstt/tree/master/debug/accuracy_tools/msprobe)。

其中动态图、静态图Ascend GE后端Dump已完全迁移到msprobe工具，通过msprobe工具入口使能。静态图Ascend ms_backend和CPU/GPU后端仍然通过框架入口使能，后续会陆续迁移到msprobe工具。

## 配置指南

MindSpore在不同后端下支持的Dump功能不完全相同，需要的配置文件和生成的数据格式也不同。

### 支持功能对比表

| 功能             | Ascend ms_backend          | CPU/GPU                       |
| ---------------- | -------------------------- | ----------------------------- |
| 整网数据dump     | 支持                       | 支持                          |
| 统计信息dump     | 支持host和device模式       | CPU不支持，GPU仅支持host模式  |
| 数据采样dump     | 支持                       | 不支持                        |
| 溢出dump         | 支持                       | 不支持                        |
| 指定算子名称     | 支持                       | 支持                          |
| 指定迭代         | 支持                       | 支持                          |
| 指定device       | 支持                       | 支持                          |
| set_dump         | 支持                       | 支持                          |
| 图ir dump        | 支持                       | 支持                          |
| 执行序dump       | 支持                       | 支持                          |

## Ascend下ms_backend后端Dump

### 操作步骤

#### 1. 创建json格式配置文件

```json
{
    "common_dump_settings": {
        "op_debug_mode": 0,
        "dump_mode": 0,
        "path": "/absolute_path",
        "net_name": "ResNet50",
        "iteration": "0|5-8|100-120",
        "saved_data": "tensor",
        "input_output": 0,
        "kernels": ["Default/Conv-op12"],
        "support_device": [0,1,2,3,4,5,6,7],
        "statistic_category": ["max", "min", "l2norm"]
    },
    "e2e_dump_settings": {
        "enable": true,
        "trans_flag": true,
        "stat_calc_mode": "host"
    }
}
```

#### common_dump_settings详细说明

- op_debug_mode:
  - 0：保存所有算子或指定算子
  - 3：只保存溢出算子
  - 4：只保存异常算子的输入
  - 默认值：0

- dump_mode:
  - 0：Dump所有算子数据
  - 1：Dump指定算子数据
  - 2：使用mindspore.set_dump指定对象

- path: Dump保存数据的绝对路径

- net_name: 自定义网络名称

- iteration: 指定需要Dump的迭代，用"|"分离。例如"0|5-8|100-120"表示第1个、第6-9个、第101-121个step。"all"表示所有迭代

- saved_data:
  - "tensor"：完整张量数据
  - "statistic"：张量统计信息
  - "full"：两种都要

- input_output:
  - 0：输入和输出
  - 1：仅输入
  - 2：仅输出

- kernels: 支持三种配置格式：
  1. 算子名称列表
  2. 算子类型（不区分大小写匹配）
  3. 正则表达式：格式为"name-regex(xxx)"

- support_device: 支持的设备ID列表

- statistic_category: 统计信息类别列表
  - "max"、"min"、"avg"、"count"
  - "negative zero count"、"positive zero count"
  - "nan count"、"negative inf count"、"positive inf count"
  - "zero count"
  - "hash"（默认SHA1，也支持"hash:md5"）
  - "l2norm"
  - 默认值：["max", "min", "l2norm"]

- overflow_number: 溢出dump数据个数（仅在op_debug_mode为3时需配置）

- initial_iteration: Dump初始迭代数，非负整数，默认值：0

#### e2e_dump_settings详细说明

- enable:
  - true：同步Dump
  - false：异步Dump（默认）

- trans_flag:
  - true：转换为NCHW格式
  - false：保留Device侧格式
  - 默认值：true

- stat_calc_mode:
  - "host"：host侧计算统计信息
  - "device"：device侧计算（仅Ascend生效）
  - 默认值："host"

- device_stat_precision_mode: device统计精度模式
  - "high"：float32计算，精度高，内存占用大
  - "low"：原数据类型计算，内存占用小
  - 默认值："high"

- sample_mode:
  - 0：不开启切片dump
  - 1：开启切片dump（仅在ms_backend）

- sample_num: 切片大小，默认值100

- save_kernel_args: true时保存算子初始化信息

#### 2. 设置环境变量

```bash
export MINDSPORE_DUMP_CONFIG=/path/to/data_dump.json
```

如果配置文件未设置`path`字段或为空：

```bash
export MS_DIAGNOSTIC_DATA_PATH=/path/to/data
```

则实际path为`$MS_DIAGNOSTIC_DATA_PATH/debug_dump`

注意：
- 在网络脚本执行前设置环境变量
- 分布式场景下需在调用`mindspore.communication.init`之前配置

#### 3. 启动网络训练脚本

正确配置后，训练启动时会自动读取配置文件并保存数据。使用Dump功能将自动生成最终执行图的IR文件。

#### 4. 读取Dump数据

```python
import numpy
numpy.load("filename.npy")
```

### 数据对象目录结构

```
{path}/
    - rank_{rank_id}/
        - .dump_metadata/
        - {net_name}/
            - {graph_id}/
                - {iteration_id}/
                    {op_type}.{op_name}.json
                    statistic.csv
                    {op_type}.{op_name}.{task_id}.{stream_id}.{timestamp}.{input_output_index}.{slot}.{format}.{dtype}.npy
                - constants/
                    Parameter.data-{data_id}.0.0.{timestamp}.output.0.DefaultFormat.{dtype}.npy
            ...
        - graphs/
            ms_output_trace_code_graph_{graph_id}.pb
            ms_output_trace_code_graph_{graph_id}.ir
        - execution_order/
            ms_execution_order_graph_{graph_id}.csv
            ms_global_execution_order_graph_{graph_id}.csv
```

#### 目录字段说明

- path: 配置文件中设置的绝对路径
- rank_id: 逻辑卡号
- net_name: 配置文件中的网络名称
- graph_id: 训练的图标号
- iteration_id: 训练轮次
- op_type: 算子类型
- op_name: 算子名称
- task_id: 任务标号
- stream_id: 流标号
- timestamp: 时间戳
- input_output_index: 输入/输出标号（如output.0表示第1个输出）
- slot: slot标号
- format: 数据格式
- dtype: 原始数据类型（bfloat16、int4、uint1会被转换）
- data_id: 常量数据标号

#### 数据文件说明

- statistic.csv: 仅当`saved_data`为"statistic"或"full"时生成，包含所有落盘张量的统计信息
- npy文件: 仅当`saved_data`为"tensor"或"full"时生成
- {op_type}.{op_name}.json: 仅当`save_kernel_args`为true时生成，保存算子初始化参数

#### 算子初始化信息示例

```json
{
    "transpose_a": "False",
    "transpose_b": "False"
}
```

### 数据分析样例

完整样例脚本可通过执行`bash run_sync_dump.sh`获得。

执行完训练后，会产生最终执行图文件`ms_output_trace_code_graph_{graph_id}.ir`，其中包含每个算子的堆栈信息和生成代码位置。

#### 执行图中的信息示例

对于AlexNet中的Conv2D-op12算子，执行图包含：

1. Host侧和Device侧的输入输出信息：
```
: (<Tensor[Float32], (32, 256, 13, 13)>, <Tensor[Float32], (384, 256, 3, 3)>)
-> (<Tensor[Float32], (32, 384, 13, 13)>)
```

2. 完整算子名称：
```
Default/network-WithLossCell/_backbone-AlexNet/conv3-Conv2d/Conv2D-op12
```

3. 源代码位置：
```
# In file ./train_alexnet.py(175)/        x = self.conv3(x)/
```

#### 数据文件查找流程

1. 从执行图中获取算子完整名称
2. 确定input_output_index（如output.0表示第1个输出）
3. 确定slot（通常为0）
4. 在Dump保存目录中搜索匹配的npy文件

例如，查找Conv2D-op12的第1个输出：

```
Conv2D.Conv2D-op12.0.0.1623124369613540.output.0.DefaultFormat.float16.npy
```

## Ascend下GE后端Dump

Ascend下GE后端Dump已迁移到msprobe工具，详见[msprobe文档](https://atomgit.com/Ascend/mstt/blob/master/debug/accuracy_tools/msprobe/docs/zh/dump/mindspore_data_dump_instruct.md)。

### 迁移到msprobe后暂不支持的功能

1. 数据切片保存（sample_num、sample_mode字段）
2. set_dump能力（dump_mode为2的场景）
3. tensor和statistic同时保存（saved_data为full）
4. MD5和其他统计量无法同时开启

## CPU/GPU后端Dump

### 操作步骤

#### 1. 创建json配置文件

```json
{
    "common_dump_settings": {
        "op_debug_mode": 0,
        "dump_mode": 0,
        "path": "/absolute_path",
        "net_name": "ResNet50",
        "iteration": "0|5-8|100-120",
        "saved_data": "tensor",
        "input_output": 0,
        "kernels": ["Default/Conv-op12"],
        "support_device": [0,1,2,3,4,5,6,7],
        "statistic_category": ["max", "min", "l2norm"]
    },
    "e2e_dump_settings": {
        "enable": true,
        "trans_flag": true
    }
}
```

#### common_dump_settings说明

- op_debug_mode: CPU/GPU仅支持设置为0
- dump_mode: 0表示全量dump，1表示指定算子，2表示使用set_dump
- path: Dump保存路径
- net_name: 网络名称
- iteration: 指定迭代范围
- saved_data:
  - "tensor"：完整张量数据
  - "statistic"：统计信息（GPU支持，CPU不支持）
  - "full"：两种都要
  - 默认："tensor"
- input_output: 0表示输入和输出，1表示仅输入，2表示仅输出
- kernels: 支持三种配置（名称列表、类型、正则表达式）
- support_device: 设备ID列表
- statistic_category: 统计类别列表
  - 支持："max"、"min"、"avg"、"count"
  - "negative/positive zero count"、"nan/inf count"、"zero count"
  - "md5"、"l2norm"
  - 默认：["max", "min", "l2norm"]

#### e2e_dump_settings说明

- enable: CPU/GPU必须设置为true
- trans_flag:
  - true：转换为NCHW格式
  - false：保留原格式
  - 默认：true

#### 2. 设置环境变量

```bash
export MINDSPORE_DUMP_CONFIG=/path/to/data_dump.json
```

如果配置文件未设置`path`字段：

```bash
export MS_DIAGNOSTIC_DATA_PATH=/path/to/data
```

#### 3. 启动训练脚本

GPU环境必须采用非数据下沉模式，设置`dataset_sink_mode=False`。

#### 4. 读取数据

```python
import numpy
numpy.load("filename.npy")
```

### 数据对象目录结构

```
{path}/
    - rank_{rank_id}/
        - .dump_metadata/
        - {net_name}/
            - {graph_id}/
                - {iteration_id}/
                    {op_type}.{op_name}.json
                    statistic.csv
                    {op_type}.{op_name}.{task_id}.{stream_id}.{timestamp}.{input_output_index}.{slot}.{format}.npy
                - constants/
                    Parameter.data-{data_id}.0.0.{timestamp}.output.0.DefaultFormat.npy
            ...
        - graphs/
            ms_output_trace_code_graph_{graph_id}.pb
            ms_output_trace_code_graph_{graph_id}.ir
        - execution_order/
            ms_execution_order_graph_{graph_id}.csv
            ms_global_execution_order_graph_{graph_id}.csv
```

#### 文件说明

- statistic.csv: `saved_data`为"statistic"或"full"时生成
- npy文件: `saved_data`为"tensor"或"full"时生成
- ms_output_trace_code_graph_{graph_id}.ir: 最终执行图（可用vi查看）
- ms_execution_order_graph_{graph_id}.csv: 节点执行序
- ms_global_execution_order_graph_{graph_id}.csv: 图执行历史

### 数据分析样例

CPU/GPU后端执行`bash run_sync_dump.sh`获得完整样例。

执行图包含算子堆栈信息和源代码位置，可用于追溯算子来源。

## 注意事项

- bfloat16类型: 保存到npy时会转换为float32
- 支持的数据类型: bool、int、int8、int16、int32、int64、uint、uint8、uint16、uint32、uint64、float、float16、float32、float64、bfloat16、double、complex64、complex128
- complex类型: 仅支持保存npy，不支持统计值
- Print算子: 含string类型不支持，会有错误日志但不影响其他数据保存
- Ascend GE后端: sink size只能设置为1
- 无效输出: 默认被忽略（如Send/Print输出），可设置环境变量`MINDSPORE_DUMP_IGNORE_USELESS_OUTPUT=0`保留
