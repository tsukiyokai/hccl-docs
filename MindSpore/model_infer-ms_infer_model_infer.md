# MindSpore大语言模型带框架推理

## 特性背景

2022年末，OpenAI发布了ChatGPT大语言模型，为人工智能带来了新的研究方向。基于Transformers结构的大语言模型展现了超过预期的AI能力。

在大语言模型研究中，提高其在实际应用中的性价比是重要方向：

- 大语言模型通常是百亿、千亿级别参数量，一次推理的计算量巨大，消耗大量资源，导致单次推理成本高
- MindSpore框架提供了大语言模型推理能力，深度优化部署和推理，实现模型推理成本最优

## 模型原理

大语言模型主要分为训练和推理两个阶段：

- 训练：模型在海量文本数据中学习，将文本元素的位置关系和出现频率记录在模型权重中
- 推理：针对用户给出的文本，找到最强相关的文本元素返回

实际文本处理中使用tokenize技术，将文本分解为token（单词和标点符号）的组合。大语言模型通过多轮迭代，每轮生成一个token，逐步完成文本生成。

### 推理示例

| 推理迭代 | 推理输入 | 输入向量 | 推理结果 |
|---------|---------|---------|---------|
| 1 | 中国的首都 | [中国，的，首都] | 北京 |
| 2 | 中国的首都北京 | [中国，的，首都，北京] | 真 |
| 3 | 中国的首都北京真 | [中国，的，首都，北京，真] | 美丽 |
| 4 | 中国的首都北京真美丽 | [中国，的，首都，北京，真，美丽] | END |

## 关键步骤

MindSpore大语言模型推理的关键步骤包括：

1. 权重准备：获取和准备对应模型的权重文件
2. 模型加载：根据优化技术构建模型主干网络
3. 状态判断：判断是否需要继续推理
4. 推理前处理：对推理数据进行处理（tokenizer转换、构建特殊输入）
5. 模型推理：进行模型推理，返回token的概率分布
6. 推理后处理：计算下一个token，转换为文本

## 主要特性

### 全量/增量推理

大语言模型核心是Transformer自注意力机制。通过KVCache优化缓存已计算的key和value值：

- 全量推理：用户输入的第一轮迭代，需要计算全部key和value值
- 增量推理：后续迭代只需计算最近一个token的key和value值

### Attention优化

- Flash Attention：通过分块处理，避免多次数据搬入搬出，提升Attention计算性能
- Paged Attention：基于Linux页表原理优化KVCache，按特定大小的块存储Key和Value，节省显存

### 模型量化

支持A16W8、A16W4、A8W8量化以及KVCache量化技术，减少模型资源占用，提升推理吞吐量。

## 推理教程

本章节基于Qwen2-7B-Instruct模型演示如何使用MindSpore大语言模型推理能力进行文本生成。

### 环境准备

创建虚拟环境并安装MindSpore：

```bash
export PYTHON_ENV_NAME=mindspore-infer-py311
conda create -n ${PYTHON_ENV_NAME} python=3.11
conda activate ${PYTHON_ENV_NAME}
pip install mindspore
```

安装Ascend开发环境：

```bash
pip install ${ASCEND_HOME}/lib64/te-*.whl
pip install ${ASCEND_HOME}/lib64/hccl-*.whl
pip install sympy
```

安装Transformers软件包（用于tokenizer能力）：

```bash
pip install transformers
```

若使用模型量化能力，安装mindspore_gs包：

```bash
pip install mindspore_gs
```

### 权重准备

从Hugging Face官网下载Qwen2模型权重和tokenizer：

```bash
git lfs install
git clone https://huggingface.co/Qwen/Qwen2-7B
```

下载完成后的目录结构包含：

```
|- config.json
|- generation_config.json
|- model-00001-of-00004.safetensors
|- model-00002-of-00004.safetensors
|- model-00003-of-00004.safetensors
|- model-00004-of-00004.safetensors
|- model.safetensors.index.json
|- tokenizer_config.json
|- tokenizer.json
|- vocab.json
```

### 模型构建

构建模型并加载权重：

```python
import os
import mindspore as ms
from qwen2 import Qwen2Config, Qwen2ForCausalLM, CacheManager
from mindspore import Tensor, mint

# set mindspore context and envs
os.environ["MS_INTERNAL_DISABLE_CUSTOM_KERNEL_LIST"] = "PagedAttention"

ms.set_context(infer_boost="on")
ms.set_context(mode=ms.context.PYNATIVE_MODE)

model_path = "/path/to/model"
input_str = ["I love Beijing, because", "Hello, Qwen2"]
batch_size = len(input_str)
max_new_tokens = 64
block_size = 128
max_seq_lens = block_size * 10
block_num = (max_seq_lens * batch_size) // block_size

config = Qwen2Config.from_json(model_path + "/config.json")

model = Qwen2ForCausalLM(config)
# load weight
model.load_weight(model_path)

cache_manager = CacheManager(config, block_num, block_size, batch_size)
```

环境变量说明：

- MS_INTERNAL_DISABLE_CUSTOM_KERNEL_LIST：设置PagedAttention使用MindSpore支持TH拉平的算子
- infer_boost：开启推理优化，使能FlashAttention、PagedAttention等融合算子
- mode：设置执行模式为动态图模式

参数说明：

- input_str：需要推理的原始文本
- model_path：模型目录路径
- max_new_tokens：最大推理单词数量
- block_size：PagedAttention中KVCache的block大小
- max_seq_lens：模型推理支持的最大长度

### 模型推理

#### 前处理

利用tokenizer将文本转换为token id：

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained(model_path, trust_remote_code=True)

input_str = ["I love Beijing, because", "Hello, Qwen2"]

input_ids = tokenizer(input_str)["input_ids"]

print(input_ids)
```

输出：

```
[[40, 2948, 26549, 11, 1576], [9707, 11, 1207, 16948, 17]]
```

#### 整网计算

将迭代推理封装为generate函数：

```python
from typing import List
from mindspore import ops, mint, Tensor, dtype
from qwen2 import Qwen2Config, Qwen2ModelInput, Qwen2ForCausalLM, CacheManager, sample

def generate(model: Qwen2ForCausalLM, config: Qwen2Config, cache_manager: CacheManager, input_ids: List, max_new_tokens: int, max_seq_lens: int, eos_token_id: int):
    batch_size = len(input_ids)
    assert max_seq_lens >= max(map(len, input_ids))

    cur = min(map(len, input_ids))
    is_prefill = True
    it = 0

    decode_q_seq_lens = Tensor([1 for _ in range(batch_size)], dtype=dtype.int32)
    decode_mask = ops.zeros((1, 1), dtype=config.param_dtype)
    attn_mask = None
    q_seq_lens = None

    while cur <= max_seq_lens and it < max_new_tokens:
        batch_valid_length = Tensor([cur for _ in range(batch_size)], dtype=dtype.int32)
        if is_prefill:
            inp = Tensor([input_ids[i][:cur] for i in range(batch_size)], dtype=dtype.int32)
            pos = mint.arange(cur).astype(dtype.int32)
            block_tables, slot_mapping = cache_manager.step(0, cur)
            attn_mask = ops.logical_not(ops.sequence_mask(pos + 1, cur)).astype(config.param_dtype)
            q_seq_lens = None
        else:
            inp = Tensor([[input_ids[i][cur - 1]] for i in range(batch_size)], dtype=dtype.int32)
            pos = Tensor([[cur - 1] for _ in range(batch_size)], dtype=dtype.int32).view(-1)
            block_tables, slot_mapping = cache_manager.step(cur - 1, 1)
            attn_mask = decode_mask
            q_seq_lens = decode_q_seq_lens

        model_input = Qwen2ModelInput(
            input_ids=inp,
            positions=pos,
            batch_valid_length=batch_valid_length,
            is_prefill=is_prefill,
            attn_mask=attn_mask,
            k_caches=cache_manager.k_caches,
            v_caches=cache_manager.v_caches,
            block_tables=block_tables,
            slot_mapping=slot_mapping,
            q_seq_lens=q_seq_lens
        )

        logits = model(model_input)

        next_tokens = sample(logits)

        for i in range(batch_size):
            if cur >= len(input_ids[i]):
                input_ids[i].append(int(next_tokens[i]))

        cur += 1
        it += 1
        if is_prefill:
            is_prefill = False

    for i in range(batch_size):
        if eos_token_id in input_ids[i]:
            eos_idx = input_ids[i].index(eos_token_id)
            input_ids[i] = input_ids[i][: eos_idx + 1]

    return input_ids
```

generate函数的核心步骤：

模型输入准备 -- 构造Qwen2ModelInput对象，参数包括：

- input_ids：输入的词表id列表
- positions：输入词表在推理语句中的位置信息，用于rope旋转位置编码
- batch_valid_length：当前推理的语句长度
- is_prefill：是否是全量推理
- attn_mask：注意力分数计算时的掩码矩阵
- k_caches/v_caches：KVCache对象
- block_tables：每个batch使用的block
- slot_mapping：token在block中的位置
- q_seq_lens：注意力中query的长度

模型计算 -- 调用模型启动计算逻辑

采样结果 -- 通过sample函数获取下一个单词的id

更新下一轮输入 -- 更新词表list进入下一迭代

调用generate函数：

```python
output = generate(
    model=model,
    config=config,
    cache_manager=cache_manager,
    input_ids=input_ids,
    max_new_tokens=max_new_tokens,
    eos_token_id=tokenizer.eos_token_id,
    max_seq_lens=max_seq_lens
)
```

#### 后处理

将token id转换回文本：

```python
result = [tokenizer.decode(a) for a in output]
print(result)
```

输出示例：

```
<s>I love Beijing, because it is a city that is constantly changing. I have been living here for 10 years and I have seen the city changes so much. ...
```

### 模型并行

对于参数规模超过单卡内存的大语言模型（如Llama2-70B、Qwen2-72B），需要采用多卡并行推理。MindSpore支持将模型切分成N份可并行的子模型。

主流并行方法：

- 数据并行（data parallel）：将数据切分成多份，分别在多卡上并行计算，通常通过batch实现
- 模型并行（tensor parallel）：将算子按网络脚本定义的方式切分，切分数量通常等于卡数
- 流水并行（pipeline parallel）：将模型按层数切分成多个实例，实现流水计算
- 专家并行（expert parallel）：MoE类模型特有，将不同专家分发到不同计算实体

实现模型并行的步骤：

1. 模型适配：根据卡数切分模型，主要涉及Attention中query、key、value的linear操作
2. 权重适配：切分权重加载，减少不必要权重加载占用显存，主要涉及embedding和linear模块
3. 模型推理：多卡推理需要同时启动多个进程，可使用msrun并行运行工具

### 模型量化

MindSpore支持以下量化技术：

- A16W8/A16W4量化：对权重进行量化，将float16权重用8-bits的int8或4-bits的int4保存，计算前反量化，降低显存占用，提升并发度
- A8W8量化：对整网进行量化，将float16计算转为int8计算，提升计算效率，需特定量化算子支持
- KVCache量化：对KVCache进行float16到int8的量化，通过融合算子降低量化开销

使用Golden Stick进行模型量化：

1. 模型量化：利用量化算法将数据类型从高bit转为低bit
2. 模型推理：加载模型，进行量化改造，加载量化后权重，调用推理

## 高级用法

### 使用自定义算子优化模型推理

MindSpore支持用户自定义算子接入，实现特定场景的算子优化或网络中的算子融合，通过修改网络脚本的算子API来实现。

### 大语言模型离线推理

虽然MindSpore推荐使用灵活的在线推理（权重CKPT+网络脚本），但在端侧或边缘侧特定场景，用户可使用MindSpore Lite离线推理方案，需将模型导出为MindIR文件传给MindSpore Lite运行时。
