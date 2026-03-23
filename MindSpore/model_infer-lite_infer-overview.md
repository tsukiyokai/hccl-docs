# MindSpore Lite推理概述

## 特性背景

MindSpore Lite是一个轻量化推理引擎，专注于离线模型的高效推理部署方案和端上设备的高性能推理。该框架支持将MindSpore、ONNX、TF等多种AI框架的模型转换成MindSpore Lite格式的IR。

针对不同硬件后端，MindSpore Lite支持两种模型格式：

- 云侧推理：导出`.mindir`模型，兼容MindSpore训练框架及ONNX、TFLite、Pb等开源格式，适配昇腾加速卡和X86/ARM架构CPU。
- 端侧推理：导出`.ms`模型，支持通用CPU与Kirin NPU，作为Harmony OS内置轻量化AI引擎，提供Android/iOS平台支持。

## 推理方案

MindSpore Lite推理框架通过`converter_lite`转换工具将各类模型转换成MindSpore Lite格式，并部署到不同硬件后端。推理方案主要包括三个方面：

### 1. 转换工具

开发者可通过`converter_lite`工具将其他格式的模型文件转换成`.mindir`或者`.ms`文件进行推理部署，过程中会进行模型结构优化和融合算子使能。

### 2. 运行时

- 针对昇腾硬件：提供高效的内存/显存管理和混合并行能力
- 针对麒麟NPU和端上CPU：提供轻量化运行时、内存池、线程池等高性能推理能力

### 3. 算子库

提供高性能的CPU算子库、麒麟NPU Ascend C库，以及昇腾Ascend C算子库。

## 关键能力

1. 支持昇腾硬件推理
2. 支持鸿蒙
3. 训练后量化
4. 轻量化Micro推理部署
5. 基准调试工具

## 推理教程

MindSpore Lite推理部署包含两个主要步骤：

1. 模型转换：将模型转换成`.mindir`或`.ms`格式文件
2. 集成部署：通过MindSpore Lite推理API完成模型推理集成

针对`.ms`模型的推理教程可参考端侧推理快速入门文档，针对`.mindir`模型可参考使用Python接口执行云侧推理文档。
