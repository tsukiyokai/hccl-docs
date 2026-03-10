# Ascend C算子开发文档集

来源：CANN社区版8.5.0 (hiascend.com)
原始数据：omniai/ascendskills/ascendskills-master/AscendC_knowledge

## 目录结构

| 目录             | 内容                                                                                                       | 文档数 | 图片数 |
| ---------------- | ---------------------------------------------------------------------------------------------------------- | ------ | ------ |
| api_reference/   | Ascend C算子开发API参考 -- 基础数据结构、数据搬运、向量/矩阵/标量计算、高阶API(Matmul/Softmax/HCCL)、Utils | 661    | 274    |
| basic_knowledge/ | 基础知识 -- 入门教程、编程模型、编程指南、调试调优、框架适配、FAQ                                          | 92     | 98     |
| best_practice/   | 最佳实践 -- 异构计算基础、SIMD算子实现、性能分析与优化、MC²融合算子、Matmul调优案例                        | 54     | 77     |

合计：810个文档，449个图片，约18MB

## 每个目录的组织方式

```
<category>/
├── INDEX.md    # 完整索引(含每页标题、源URL、表格/图片数)
├── pages/      # 逐页markdown文档
└── images/     # 文档引用的PNG图片
```

## 与hccl/的关系

`hccl/ascendc-hccl.md`包含Ascend C中HCCL高阶API的15个核心接口文档（来源：CANN商用版8.0.RC3）。
本目录`api_reference/`中也包含HCCL相关API页面（来源：CANN社区版8.5.0），版本更新、内容更详细。
两者互为补充，不建议删除任何一份。
