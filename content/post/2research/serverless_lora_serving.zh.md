---
title: "LoRA 微调的云上漂流"
date: 2025-05-25T10:39:49Z
update: 2025-05-28T10:39:49Z
draft: false
author: ["Martin"]
tags: 
- 机器学习系统
- 推理服务
- LoRA微调
description: "这篇文章是我们在 ServerlessLLM 框架中，把 LoRA 微调和推理服务化的思考记录。这是一场漂流，也是一场构建。"
weight: 
slug: "serverless-lora"
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "/img/mlsys-ft/serverlessllm.png"
    caption: ""
    alt: ""
    relative: true
mermaid: true
---
# 写在前面
> Serverless + 大模型推理，有什么难的？

在传统的serverless架构中（如 AWS Lambda、Google Cloud Functions），系统设计的核心诉求是：
- 快速启动、短暂执行、高并发调度，主要针对 CPU-bound 轻量计算任务。
- 比如调用一个图像压缩函数、一个用户注册校验、一个数据库查询包装器等，这类请求大多数只需几百毫秒甚至几十毫秒，CPU 足够胜任。

但当我们把同样的「serverless」理念搬到大模型推理场景时，立刻面临三重冲击：
1. 启动代价远大于传统函数
传统 serverless 函数的启动时间可忽略（几十 ms），但大模型需要加载几十GB的模型权重，冷启动时间常常达到几十秒甚至几分钟。这意味着传统 serverless 模式中的「按需调度」和「弹性启动」如果直接套用在大模型场景，会导致极高的冷启动延迟，严重影响响应时间。
2. 推理请求处理是计算密集型任务
大模型推理本质是深度神经网络逐token的推理，哪怕一个简单的chat/completions请求也可能涉及几千次 GEMM（矩阵乘法）计算，单个推理请求可持续几百毫秒到数秒，需要 GPU 支持高吞吐。而传统serverless更偏向 I/O + 轻计算任务，不需要为每个函数分配专用加速卡。
3. 请求上下文深且难以迁移
大模型推理过程中还涉及 KV Cache、prompt执行状态等上下文，导致请求一旦启动就必须持续绑定 GPU 内存。而传统 serverless 函数天然是无状态的，可以任意横向扩缩或迁移执行实例；这在大模型场景中则成为限制迁移、影响弹性调度的一大障碍。

# ServerlessLLM简介
ServerlessLLM 是一个专为大语言模型设计的开源Serverless框架，通过整合多级存储、推理过程实时迁移（live migration）、启动时间感知调度以及创新的模型加载格式，有效解决了大模型冷启动慢、资源复用难的问题。它将模型加载格式重新设计为更高效的 chunk 结构，支持快速定位和并行加载，从而将冷启动时间从分钟级缩短至秒级，并大幅减少I/O带宽需求。借助这一机制，ServerlessLLM显著提升资源利用率和服务弹性。

不同于通用 Serverless 架构仅适用于轻量 CPU 任务，ServerlessLLM 针对大模型推理过程中的高算力需求与复杂上下文管理进行了全栈优化，全面支持 HuggingFace、vLLM 等推理引擎。无论是研究者构建实验集群，还是企业部署多租户 LLM 服务，它都提供了一种低延迟、高性能、低成本的全新推理范式。ServerlessLLM 的设计不仅重塑了大模型推理的部署方式，也为后续大模型系统的可扩展性与可维护性提供了强有力的系统支撑。

感兴趣的朋友可以查阅ServerlessLLM [论文](https://www.usenix.org/system/files/osdi24-fu.pdf) / [博客](https://github.com/ServerlessLLM/ServerlessLLM/blob/main/blogs/serverless-llm-architecture/README.md)

# 我的故事
> 为什么要做LoRA的Serverless微调与推理服务?

2024年夏天，在利物浦大学计算生物平台([Computational Biology Facility](https://www.liverpool.ac.uk/computational-biology-facility/))的实习经历让我深刻体会到了算力鸿沟的现实。当时我的工作是评测各类开源大模型在生物医学文献理解任务（主要是LLM辅助文献筛选与标注）中的表现，并对候选模型进行LoRA微调。由于算力限制，我们不得不先对模型进行压缩量化，甚至对文章内容进行压缩（summarization），然后再评测、微调。记得当`Llama-3.1-405B`这样的千亿参数模型发布时，面对其庞大的计算需求，我们“省了又省”的方法被彻底击碎，即使是把它压缩到int6也难以运行。直到导师申请到两块NVIDIA GH200，我才第一次感受到"充足算力"带来的震撼。这段经历让我意识到：在日益膨胀的大模型生态中，像我们这样缺乏计算资源的模型开发者，需要的其实不是独占顶级硬件，而是一个能快速验证想法的工具平台。对于从事AI4Science研究的朋友们应该更是这样，他们更关注AI如何为其上层领域赋能，而无需过多担心底层架构的支持。

2024年秋天，当ServerlessLLM开源社区的招募邮件出现在我的收件箱时，这个项目的设计哲学，像是来自夏天的回旋镖击中了我，它正是我理想中的解决方案。随着参与社区讲座、撰写文档的深入，我逐步建立起对机器学习系统和 Serverless 架构的系统性理解。而其中令人兴奋的发现是，LoRA微调似乎很适合做成Serverless：
- 计算效率的突破：LoRA通过冻结原始参数、仅训练低秩矩阵（通常参数量只有原模型的2-4%），将微调所需的GPU显存降低70%以上。这意味着单块消费级显卡甚至可完成7B模型的微调，而且微调任务时长从小时级缩短到分钟级。
- 与Serverless特性的完美契合：
    - 弹性计费：短时微调任务按秒级资源占用付费，成本低于常驻实例的1/10
    - 冷启动优势：MB级的LoRA适配器加载速度远超完整模型检查点
    - 动态部署：PEFT库已实现LoRA adapter热插拔，支持同一基础模型快速切换多个领域适配器
- 得益于ServerlessLLM非常具有创新性的模型格式设计（一种比safetensor格式快5-10倍的模型加载格式），推理时LoRA adapter权重的加载和注入也变的得心应手。
- 应用场景明确，未来可期：在当前阶段，ServerlessLLM 平台可以为模型开发者提供一站式的微调与评测能力，降低实验门槛，加速模型迭代。但更重要的是，随着大模型逐步走向终端与个人（如 Apple Intelligence 等系统级集成），对“每个人都拥有自己模型偏好”的需求将持续扩大。从企业定制到用户级个性化，LoRA 提供了一个既轻量又易于切换的解法，而 Serverless 架构则让这种能力可被广泛获取、快速部署，真正实现「人人可调、按需即用」的智能服务体系。

# 当前工作&未来计划
目前实现了如下features：
- ✅ 支持在 Transformers 后端进行 LoRA 微调，并将 LoRA adapter 保存为sllm格式(一种高效加载格式)
- ✅ 支持从Huggingface Hub 拉取 LoRA adapters，并保存为sllm格式
- ✅ 支持加载、卸载、切换多个 LoRA adapters，无需重启模型服务即可实现 LoRA adapter 热插拔（不影响基础模型服务）

感谢社区成员们的帮助，让这个feature得以顺利实现。ServerlessLLM现在已经支持基本的LoRA微调与推理服务，不过还有一些需要优化和继续开发的地方：
- 微调任务的独立后端（已经在做了）
- 为 vLLM 后端实现推理阶段的 LoRA adapter 动态注入功能
- 支持全量参数微调...

# 写在最后
> 让创新回归本质：把这套方案送给2024年夏天的自己

在ServerlessLLM上实现这套方案后，我们不难想象它未来将如何改变开发者的工作范式：
- 评测者不再需要为GPU发愁，通过API即可对比不同适配器在基准测试中的表现
- 研究者可以像调用函数一样启动分布式微调实验，专注算法而非基础设施

> 参数高效微调不是在妥协，而是在重新定义可能性。而Serverless架构，正是让这种可能性普惠化的关键催化剂。

欢迎朋友们查看[开源代码](https://github.com/ServerlessLLM/ServerlessLLM)和[相关文档](https://serverlessllm.github.io/docs/stable/features/peft_lora_serving)了解更多细节。期待听到大家的宝贵意见！

# 相关工作&参考工作
[1] [ServerlessLLM](https://www.usenix.org/conference/osdi24/presentation/fu)

[2] [Huggingface PEFT LoRA](https://huggingface.co/docs/peft/main/en/developer_guides/lora)

[3] [TogetherAI](https://docs.together.ai/docs/fine-tuning-quickstart)

[4] [vLLM LoRA](https://docs.vllm.ai/en/stable/features/lora.html)

[5] [dLoRA](https://www.usenix.org/conference/osdi24/presentation/wu-bingyang)

[6] [S-LoRA](https://arxiv.org/abs/2311.03285)

[7] [Punica](https://proceedings.mlsys.org/paper_files/paper/2024/file/054de805fcceb78a201f5e9d53c85908-Paper-Conference.pdf)