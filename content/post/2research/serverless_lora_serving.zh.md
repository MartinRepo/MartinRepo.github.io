---
title: "Serverless微调与推理服务"
date: 2025-05-26T10:39:49Z
draft: false
author: ["Martin"]
tags: 
- 机器学习系统
- LoRA推理服务
- 微调服务
- 大模型
description: ""
weight: 
slug: "serverless-lora"
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: ""
    caption: ""
    alt: ""
    relative: true
mermaid: true
---
# 写在前面：Serverless + 大模型推理，有什么难的
在传统的serverless架构中（如 AWS Lambda、Google Cloud Functions），系统设计的核心诉求是：
- 快速启动、短暂执行、高并发调度，主要针对 CPU-bound 轻量计算任务。
- 比如调用一个图像压缩函数、一个用户注册校验、一个数据库查询包装器等，这类请求大多数只需几百毫秒甚至几十毫秒，CPU 足够胜任。

但当我们把同样的「serverless」理念搬到大模型推理场景时，立刻面临三重冲击：
1. 启动代价远大于传统函数
传统 serverless 函数的启动时间可忽略（几十 ms），但大模型需要加载几十GB的模型权重，冷启动时间常常达到几十秒甚至几分钟。这意味着传统 serverless 模式中的「按需调度」和「弹性启动」如果直接套用在大模型场景，会导致极高的冷启动延迟，严重影响响应时间。
2. 请求处理是重计算密集型
大模型推理本质是深度神经网络逐token的推理，哪怕一个简单的chat/completions请求也可能涉及几千次 GEMM（矩阵乘法）计算，单个推理请求可持续几百毫秒到数秒，需要 GPU 支持高吞吐。而传统serverless更偏向 I/O + 轻计算任务，不需要为每个函数分配专用加速卡。
3. 请求上下文深且难以迁移
大模型推理过程中还涉及 KV Cache、prompt执行状态等上下文，导致请求一旦启动就必须持续绑定 GPU 内存。而传统 serverless 函数天然是无状态的，可以任意横向扩缩或迁移执行实例；这在大模型场景中则成为限制迁移、影响弹性调度的一大障碍。

# ServerlessLLM简介
ServerlessLLM 是一个专为大语言模型设计的开源Serverless框架，通过整合多级存储、推理过程实时迁移（live migration）、启动时间感知调度以及创新的模型加载格式，有效解决了大模型冷启动慢、资源复用难的问题。它将模型加载格式重新设计为更高效的 chunk 结构，支持快速定位和并行加载，从而将冷启动时间从分钟级缩短至秒级，并大幅减少I/O带宽需求。借助这一机制，ServerlessLLM显著提升资源利用率和服务弹性。

不同于通用 Serverless 架构仅适用于轻量 CPU 任务，ServerlessLLM 针对大模型推理过程中的高算力需求与复杂上下文管理进行了全栈优化，全面支持 HuggingFace、vLLM 等推理引擎。无论是研究者构建实验集群，还是企业部署多租户 LLM 服务，它都提供了一种低延迟、高性能、低成本的全新推理范式。ServerlessLLM 的设计不仅重塑了大模型推理的部署方式，也为后续大模型系统的可扩展性与可维护性提供了强有力的系统支撑。

感兴趣的朋友可以查阅ServerlessLLM [论文](https://www.usenix.org/system/files/osdi24-fu.pdf) / [博客](https://github.com/ServerlessLLM/ServerlessLLM/blob/main/blogs/serverless-llm-architecture/README.md)

# 为什么要做LoRA的Serverless微调与推理服务
2024年夏天，在利物浦大学计算生物小组的实习经历让我深刻体会到了算力鸿沟的现实。当时我的工作是评测各类开源大模型在生物医学文献理解任务中的表现，并对候选模型进行LoRA微调（由于算力限制，我们不得不先对模型惊醒压缩量化，甚至对文章内容进行压缩（summarization），然后再评测、微调）。记得当Llama-3.1-405B这样的千亿参数模型发布时，面对其庞大的计算需求，我们“省了又省”的方法被彻底击碎，即使是把它压缩到int6也难以运行。——直到导师申请到两块NVIDIA GH200，我才第一次感受到"充足算力"带来的震撼。这段经历让我意识到：在日益膨胀的大模型生态中，像我们这样缺乏计算资源的模型开发者，需要的从来不是独占顶级硬件，而是一个能快速验证想法的工具平台。

2024年秋天，当ServerlessLLM开源社区的招募邮件出现在我的收件箱时，这个项目的设计哲学像是一颗来自夏天的子弹，瞬间击中眉心，它正是我理想中的解决方案。通过参与社区贡献，我逐渐理解了机器学习系统与Serverless架构的深层协同，而其中最令人兴奋的发现是：LoRA微调与Serverless简直是天作之合：
- 计算效率的突破：LoRA通过冻结原始参数、仅训练低秩矩阵（通常参数量只有原模型的2-4%），将微调所需的GPU显存降低70%以上。这意味着单块消费级显卡甚至可完成7B模型的微调，而且微调任务时长从小时级缩短到分钟级（实测bloomz-560m仅需1-2分钟）
- 与Serverless特性的完美契合：
    - 弹性计费：短时微调任务按秒级资源占用付费，成本仅为常驻实例的1/10
    - 冷启动优势：MB级的LoRA适配器加载速度远超完整模型检查点
    - 动态部署：PEFT库已实现适配器热插拔，支持同一基础模型快速切换多个领域适配器
- 得益于ServerlessLLM非常具有创新性的模型格式设计（一种比safetensor格式快5-10倍的模型加载格式），推理时LoRA adapter权重的加载和注入也变的得心应手。

在社区成员的全力帮助下，ServerlessLLM现在已经支持基本的LoRA微调与推理服务，不过还有一些需要继续开发和优化的地方：
- 微调任务的独立后端（已经在做了）
- 多个adapter融合注入
- 支持全量参数微调

欢迎朋友们查看[源代码](https://github.com/ServerlessLLM/ServerlessLLM)和[相关社区文档](https://serverlessllm.github.io/docs/stable/intro)。期待听到大家的意见！

> 让创新回归本质：把这套方案送给2024年夏天的自己

在ServerlessLLM上实现这套方案后，我们不难想象它未来将如何改变开发者的工作范式：
- 评测者不再需要为GPU发愁，通过API即可对比不同适配器在基准测试中的表现
- 研究者可以像调用函数一样启动分布式微调实验，专注算法而非基础设施

> 参数高效微调不是在妥协，而是在重新定义可能性。而Serverless架构，正是让这种可能性普惠化的关键催化剂。
