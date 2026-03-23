# Ascend性能调优

## 概述

本教程介绍如何在Ascend AI处理器上使用MindSpore Profiler进行性能调优。MindSpore Profiler可以提供算子执行时间分析、内存使用分析、AI Core指标分析、Timeline展示等功能，帮助用户分析性能瓶颈、优化训练效率。

## 操作流程

1. 准备训练脚本
2. 在训练脚本中调用性能调试接口，如`mindspore.profiler.profile`以及`mindspore.profiler.DynamicProfilerMonitor`接口
3. 运行训练脚本
4. 通过MindStudio Insight软件查看性能数据

## 使用方法

### 方式一：mindspore.profiler.profile接口使能

在训练脚本中添加MindSpore profile相关接口。该接口支持两种采集方式：自定义for循环方式和CallBack方式，且在Graph和PyNative两种模式下都支持。

#### 自定义for循环方式采集样例

```python
import mindspore
from mindspore.profiler import ProfilerLevel, ProfilerActivity, AicoreMetrics, HostSystem

steps = 15
net = Net()

experimental_config = mindspore.profiler._ExperimentalConfig(
    profiler_level=ProfilerLevel.Level0,
    aic_metrics=AicoreMetrics.AiCoreNone,
    l2_cache=False,
    mstx=False,
    data_simplification=False,
    host_sys=[HostSystem.CPU, HostSystem.MEM]
)

with mindspore.profiler.profile(activities=[ProfilerActivity.CPU, ProfilerActivity.NPU],
                                schedule=mindspore.profiler.schedule(wait=0, warmup=0, active=1,
                                    repeat=1, skip_first=0),
                                on_trace_ready=mindspore.profiler.tensorboard_trace_handler("./data"),
                                profile_memory=False,
                                experimental_config=experimental_config) as prof:
    for step in range(steps):
        train(net)
        prof.step()
```

schedule参数说明：

- `skip_first`：跳过前skip_first个step
- `wait`：等待阶段，跳过wait个step
- `warmup`：预热阶段，跳过warmup个step
- `active`：采集active个step
- `repeat`：重复执行次数

例如：配置`schedule(skip_first=10, wait=10, warmup=5, active=5, repeat=2)`时，Profiler先跳过前10个step（0-9），第1个repeat等待10个step（10-19），预热5个step（20-24），采集5个step（25-29）的性能数据，第2个repeat重复等待10个step（30-39），预热5个step（40-44），采集5个step（45-49）的性能数据。

#### CallBack方式采集样例

```python
import mindspore

class StopAtStep(mindspore.Callback):
    def __init__(self, start_step, stop_step):
        super(StopAtStep, self).__init__()
        self.start_step = start_step
        self.stop_step = stop_step
        experimental_config = mindspore.profiler._ExperimentalConfig()
        self.profiler = mindspore.profiler.profile(
            start_profile=False,
            experimental_config=experimental_config,
            schedule=mindspore.profiler.schedule(
                wait=0, warmup=0,
                active=self.stop_step - self.start_step + 1,
                repeat=1, skip_first=0),
            on_trace_ready=mindspore.profiler.tensorboard_trace_handler("./data"))

    def on_train_step_begin(self, run_context):
        cb_params = run_context.original_args()
        step_num = cb_params.cur_step_num
        if step_num == self.start_step:
            self.profiler.start()

    def on_train_step_end(self, run_context):
        cb_params = run_context.original_args()
        step_num = cb_params.cur_step_num
        if self.start_step <= step_num <= self.stop_step:
            self.profiler.step()
        if step_num == self.stop_step:
            self.profiler.stop()
```

### 方式二：动态profiler使能

使用`mindspore.profiler.DynamicProfilerMonitor`接口，需要配置JSON文件"profiler_config.json"。

JSON配置样例：

```json
{
   "start_step": 2,
   "stop_step": 5,
   "aic_metrics": "AiCoreNone",
   "profiler_level": "Level0",
   "analyse_mode": 0,
   "activities": ["CPU", "NPU"],
   "export_type": ["text"],
   "profile_memory": false,
   "mstx": false,
   "parallel_strategy": false,
   "with_stack": false,
   "data_simplification": true,
   "l2_cache": false,
   "analyse": true,
   "record_shape": false,
   "prof_path": "./data",
   "mstx_domain_include": [],
   "mstx_domain_exclude": [],
   "host_sys": [],
   "sys_io": false,
   "sys_interconnection": false
}
```

使用示例：

```python
from mindspore.profiler import DynamicProfilerMonitor

dp = DynamicProfilerMonitor(cfg_path="./cfg_path", output_path="./output_path")
STEP_NUM = 15
net = Net()
for _ in range(STEP_NUM):
    train(net)
    dp.step()
```

### 方式三：环境变量使能

使用环境变量使能方式，只需将参数配置到环境变量中，在模型训练中会自动采集性能数据。该方式只支持单卡场景。

环境变量配置示例：

```bash
export MS_PROFILER_OPTIONS='{
"start": true,
"output_path": "/XXX",
"activities": ["CPU", "NPU"],
"with_stack": true,
"aic_metrics": "AicoreNone",
"l2_cache": false,
"profiler_level": "Level0"}'
```

### 方式四：离线解析

使用`mindspore.profiler.profiler.analyse`接口进行离线解析：

```python
from mindspore.profiler.profiler import analyse

analyse("./profiler_data_path")
```

### 方式五：轻量化打点

使用mstx接口自定义打点：

```python
from mindspore.profiler import mstx

range_id = mstx.range_start("train")
mstx.mark("start")
# train_step
mstx.range_end(range_id)
```

## 性能数据

性能数据采集完成后，原始数据按照以下目录结构存储：

```
└── localhost.localdomain_*_ascend_ms
    ├── profiler_info_{Rank_ID}.json
    ├── profiler_metadata.json
    ├── ASCEND_PROFILER_OUTPUT
    │   ├── dataset.csv
    │   ├── operator_details.csv
    │   ├── minddata_pipeline_raw_{Rank_ID}.csv
    │   ├── minddata_pipeline_summary_{Rank_ID}.csv
    │   ├── minddata_pipeline_summary_{Rank_ID}.json
    │   ├── api_statistic.csv
    │   ├── ascend_mindspore_profiler_{Rank_ID}.db
    │   ├── communication_analyzer.db
    │   ├── communication.json
    │   ├── communication_matrix.json
    │   ├── data_preprocess.csv
    │   ├── hbm.csv
    │   ├── hccs.csv
    │   ├── kernel_details.csv
    │   ├── l2_cache.csv
    │   ├── memory_record.csv
    │   ├── nic.csv
    │   ├── npu_module_mem.csv
    │   ├── op_statistic.csv
    │   ├── pcie.csv
    │   ├── roce.csv
    │   ├── step_trace_time.csv
    │   ├── operator_memory.csv
    │   └── trace_view.json
    ├── FRAMEWORK
    ├── logs
    └── PROF_000001_20230628101435646_FKFLNPEPPRRCFCBA
```

### dataset.csv

记录dataset算子的信息。

| 字段名      | 字段解释                 |
| ----------- | ------------------------ |
| Operation   | 对应的数据集操作名称     |
| Stage       | 操作所处的阶段           |
| Occurrences | 操作出现次数             |
| Avg. time(us) | 操作平均时间(微秒)     |
| Custom Info | 自定义信息               |

### kernel_details.csv

包含在NPU上执行的所有算子的信息。若前端调用了schedule进行step打点，则会增加Step Id字段。

### minddata_pipeline_raw_{Rank_ID}.csv

记录dataset数据集操作的性能指标。

| 字段名                    | 字段解释         |
| ------------------------- | ---------------- |
| op_id                     | 数据集操作编号   |
| op_type                   | 操作类型         |
| num_workers               | 操作进程数量     |
| output_queue_size         | 输出队列大小     |
| output_queue_average_size | 输出队列平均大小 |
| output_queue_length       | 输出队列长度     |
| output_queue_usage_rate   | 输出队列使用率   |
| sample_interval           | 采样间隔         |
| parent_id                 | 父操作编号       |
| children_id               | 子操作编号       |

### minddata_pipeline_summary_{Rank_ID}.csv

记录更详细的dataset数据集操作性能指标，并根据性能指标给出优化建议。

| 字段名                   | 字段解释             |
| ------------------------ | -------------------- |
| op_ids                   | 数据集操作编号       |
| op_names                 | 操作名称             |
| pipeline_ops             | 操作管道             |
| num_workers              | 操作进程数量         |
| queue_average_size       | 输出平均大小         |
| queue_utilization_pct    | 输出队列使用率       |
| queue_empty_freq_pct     | 输出队列空闲频率     |
| children_ids             | 子操作编号           |
| parent_id                | 父操作编号           |
| avg_cpu_pct              | 平均CPU使用率        |
| per_pipeline_time        | 每个管道执行时间     |
| per_push_queue_time      | 每个推送队列时间     |
| per_batch_time           | 每个数据批次执行时间 |
| avg_cpu_pct_per_worker   | 平均每个线程CPU使用率 |
| cpu_analysis_details     | CPU分析详情          |
| queue_analysis_details   | 队列分析详情         |
| bottleneck_warning       | 性能瓶颈警告         |
| bottleneck_suggestion    | 性能瓶颈建议         |

### trace_view.json

建议使用MindStudio Insight工具或chrome://tracing/打开。

## 性能调优案例

性能调优流程：

1. 使用MindStudio Insight可视化工具定界性能问题
2. 根据定界结果进行针对性调优
3. 重跑训练、采集性能数据、查看调优效果
4. 重复以上过程直到解决性能问题

### 概览界面总览数据情况

- 在MindStudio Insight界面中选择"导入数据"按钮，导入采集的profiler数据
- 在概览界面展示所选通信域下每张卡的计算、通信、空闲时间占比情况

| 图例                 | 含义                                           |
| -------------------- | ---------------------------------------------- |
| 总计算时间           | 昇腾设备上的内核时间总和                       |
| 纯计算时间           | 纯计算时间 = 总计算时间 - 通信时间（被覆盖）   |
| 通信时间（被覆盖）   | 被覆盖的通信时长，即计算和通信同时进行的时长   |
| 通信时间（未被覆盖） | 未被覆盖的通信时长，即纯通信时长               |
| 空闲时间             | 未进行计算或通信的时长                         |

### 定界、分析问题

#### 计算问题

通常表现为通信域中总计算时间占比的极大值和极小值差异过大。若某些计算卡的计算时间明显超出了正常范围，可能意味着这张卡承担了过于繁重的计算任务，比如要处理的数据量过大，或者模型计算的复杂程度过高。

可以使用MindStudio Insight的卡间性能比对功能，设置两卡进入比对模式，在算子界面查看饼状图展示的各类算子耗时占比和表格展示的各类算子详细信息。

#### 调度问题

通常表现为通信域中空闲时间占比的极大值和极小值差异过大。若计算卡的空闲时间过长，说明任务分配可能不太均衡，或存在卡之间互相等待数据的情况。

需要到时间线界面将异常卡和正常卡进行比较，进一步定位出现问题的算子。选择HostToDevice连线类型，展示了CANN层算子到AscendHardware的算子的下发执行关系，以及CANN层算子到HCCL通信算子的下发执行关系。

- 倾斜的HostToDevice连线：调度任务安排合理，昇腾设备满负荷执行
- 竖直的HostToDevice连线：表示存在调度问题，昇腾设备未满负荷执行

#### 通信问题

若通信时间（未被覆盖）过长，表明计算和通信之间的协同出现了问题。需要进入通信界面进一步分析。

通信矩阵分析：

可以获取各个通信域下，卡间的带宽、传输大小、链路方式和传输时长情况。分析时可以先查看传输大小，分析是否存在差异；其次查看传输时长，确定是否有异常卡；最后查看带宽情况，如果不同卡间的带宽数据差异过大或异常，都意味着通信域中存在异常卡。

通信时长分析：

通信时长是指计算卡之间进行一次通信所花费的时间。用户选择具体通信域后，可在通信时长界面中查看通信域中各个计算卡的耗时汇总情况，以及每个通信算子的时序图和通信时长的分布图。

## 常见工具问题及解决办法

### 使用step采集性能数据常见问题

#### schedule配置错误问题

schedule配置相关参数有5个：wait、warmup、active、repeat、skip_first。每个参数大小必须大于等于0；其中active必须大于等于1，否则抛出警告，并设置为默认值1；如果repeat设置为0，Profiler会根据模型训练次数来确定repeat值，此时会多生成一个采集不完整的性能数据，最后一个step的数据用户无需关注。

#### schedule与step配置不匹配问题

schedule的配置应小于模型训练的次数，即`repeat*(wait+warmup+active)+skip_first`应小于模型训练的次数。如果schedule的配置大于模型训练的次数，Profiler会抛出异常警告，但这并不会打断模型训练，但可能存在采集解析的数据不全的情况。
