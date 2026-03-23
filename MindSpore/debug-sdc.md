# 特征值检测

## 概述

### 背景

模型训练期间，处理器可能发生特征值检测异常，导致计算错误且无法上报。这类异常会对模型训练产生严重的负面影响。

### 解决方案

MindSpore框架2.7.1版本引入了"特征值与CheckSum联合检测方案，能够更加准确地定位静默故障"。该方案采样参数梯度进行特征值检测，并在检测到多次特征值异常时，通过"三振出局"机制触发CheckSum校验。用户可以通过`MS_NPU_ASD_CONFIG`对联合检测进行配置。

### 使用建议与检测原理

关键发现包括：

- 并非所有特征值检测异常都影响模型收敛
- 反向传播中的异常影响远大于正向计算
- 过多检测点会影响性能
- MindSpore默认选择反向传播中的`Norm`激活值梯度作为检测特征，基于Llama 2-7B测试性能损失小于2%

### 使用限制

- 仅支持Atlas A2训练系列产品
- 仅支持检测8维以内Transformer类模型
- 支持bfloat16和float32数据类型
- 仅检测训练过程中的异常

联合检测方案仅支持自动并行或半自动并行模式。CheckSum仅针对bfloat16数据类型的MatMul算子进行校验。

## 特性开关及配置

环境变量`MS_NPU_ASD_CONFIG`对特征值和CheckSum联合检测进行配置，格式为key:value，逗号分隔各配置项：

- `enable`：特征值检测开关
- `with_checksum`：联动CheckSum开关
- `grad_sample_interval`：特征值采样间隔
- `upper_thresh1`和`upper_thresh2`：绝对阈值和相对阈值
- `cooldown`：特征值异常冷却时间
- `strikes_num`和`strikes_window`：触发CheckSum所需的异常次数和时间窗口
- `checksum_cooldown`：CheckSum冷却时间

默认配置：

```
MS_NPU_ASD_CONFIG="enable:false,with_checksum:false,grad_sample_interval:10,upper_thresh1:1000000,upper_thresh2:100,cooldown:5,strikes_num:3,strikes_window:480,checksum_cooldown:180"
```

## 使用用例

网络脚本示例（`silent_detect.py`）：

```python
"""Silent Detect Demo"""
import time
import numpy as np

import mindspore as ms
from mindspore import nn, Tensor, Parameter, context, ops, jit
from mindspore.communication import init, get_rank
from mindspore.nn import Momentum, TrainOneStepCell
from mindspore.parallel.auto_parallel import AutoParallel

context.set_context(mode=context.GRAPH_MODE)
init()
ms.set_seed(1)
np.random.seed(1)

class Net(nn.Cell):
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Dense(1, 8)
        self.fc2 = nn.Dense(8, 8)
        self.relu = ops.ReLU()
        self.eod_mask = ops.auto_generate.GenerateEodMaskV2()
        self.cur_step = Parameter(Tensor(-1, ms.int64), requires_grad=False)
        rank_id = get_rank()
        if rank_id == 2:
            self.flip_mode = 'bitflip_designed'
        else:
            self.flip_mode = 'multiply'

    def construct(self, x):
        x = self.fc1(x)
        x = self.relu(x)
        ele_pos = Tensor(0, ms.int64)
        seed = Tensor(0, ms.int64)
        offset = Tensor(0, ms.int64)
        start = 0
        steps = [5]
        error_mode = 'cycle'
        multiply_factor = 1.0
        bit_pos = 0
        flip_probability = 0.0
        self.cur_step = self.cur_step + 1
        x = self.eod_mask(x, ele_pos, self.cur_step, seed, offset, start, steps, error_mode, self.flip_mode,
                          multiply_factor, bit_pos, flip_probability)
        x = self.fc2(x)
        return x

if __name__ == '__main__':
    net = Net()
    optimizer = Momentum(net.trainable_params(), learning_rate=0.1, momentum=0.9)
    net = TrainOneStepCell(net, optimizer)
    net.set_train()

    @jit
    def compiled_one_step(inputs):
        net(inputs)

    parallel_net = AutoParallel(compiled_one_step, parallel_mode='semi_auto')
    for i in range(200):
        inputs = Tensor(np.random.rand(8, 1).astype(np.float32))
        parallel_net(inputs)
        time.sleep(1)
```

启动命令：

```bash
export MS_NPU_ASD_CONFIG="enable:true,with_checksum:true,grad_sample_interval:1,cooldown:1,strikes_num:1"
msrun --worker_num=8 --local_worker_num=8 --master_addr=127.0.0.1 --master_port=11235 --join=True python silent_detect.py
```

查看训练日志中的异常记录：

```bash
$ grep -m1 'Silent detect strike' worker_0.log
[WARNING] DEBUG(2950752,fffee7e591e0,python):2025-08-26-10:46:26.665.782 [mindspore/ccsrc/tools/silent_detect/silent_detector.cc:109] SilentDetect] Silent detect strike detected: StrikeRecord{timestamp: 1756176386, name: fc1.weight, value: inf, stat: StatData{avg: 6.44326e+12, pre_value: 6.441e+14, count: 6, none_zero_count: 6}}
$ grep -m1 'Global CheckSum result is' worker_0.log
[WARNING] DEBUG(2950752,fffda37fe1e0,python):2025-08-26-10:47:28.934.305 [mindspore/ccsrc/tools/silent_detect/silent_detector.cc:316] DoCheckSum] Global CheckSum result is 0
```

## 检测结果及处理

### 异常检测结果

未检测到异常时对训练无影响。

检测到数值异常后，训练失败并上报告警。定位故障设备的方法：

- 搜索应用日志，查询ERROR级别日志，关键字"accuracy sensitivity feature abnormal"
- 监控NPU健康状态：Health Status显示Warning，Error Code显示80818C00
- 查看MindCluster事件，上报错误码80818C00

联合检测产生的告警：

- 特征值异常日志关键字："Silent detect strike"
- 触发CheckSum校验日志关键字："Feature value detection strikes out"
- 静默故障识别日志关键字："CheckSum detects MatMul error on rank"和"SilentCheck detects SDC error"

### 故障处理

1. 将异常设备隔离，断点续训拉起继续训练
2. 在异常设备上通过Ascend-DMI工具执行AICore ERROR压测诊断
3. 若检测到故障卡，联系华为工程师维修更换
4. 若所有NPU均正常，则为软件问题，建议排查程序和算子原因
