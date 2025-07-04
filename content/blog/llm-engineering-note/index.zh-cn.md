---
title: LLM工程实践札记
weight: -65
draft: true
description: 记录LLM工程实践过程中的重要知识点
slug: llm-engineering-note
language: zh-cn
tags:
  - LLM
  - 工程实践
  - 笔记
series:
  - AI工程
series_order: 0
date: 2025-05-28
lastmod: 2025-06-01
authors:
  - Morethan
---
{{< lead >}}
这篇博客是“AI工程”系列的〇号文章，除了作为整个系列的索引之外，也总结一下大模型实战过程中的相关知识点；
{{< /lead >}}

## 前言

目前正式大模型技术蓬勃发展的时候，以 DeepSeek 为代表的开源基座大模型的能力逐渐追上闭源大模型，通用大模型在各项任务中的准确率不断提升，能力上限越来越高。

然而，这些上游的技术突破并不意味着下游产业的范式变革，真正能够"落地"的大模型项目却少之又少——只有 AI 编程领域较为成熟，其他的领域远未达到真正实用的地步。

### 能力锯齿

现在大模型的能力上限不断提高，然而上限的提高并不意味着下限的提高。尽管大模型已经能够在前沿数学和科学领域取得突破，但仍然有些基础的问题无法解决。

这就是所谓的"能力锯齿"现象：能力的上限和下限差距过大。这个问题在不同的情况下会有不同的"体感"。

在代码领域几乎无法感觉，因为简单的错误编程者自己就能快速解决，并且一般都很有耐心解决(甚至是必须有耐心解决🙄)，在这种情况下对于模型下限的要求并不高。

而在某些企业应用场景中，任何细小的错误都会造成巨大的商业损失。典型例子就是前段时间 Cursor 的问答模型产生了"幻觉"，将用户服务条款错误解读，导致大量用户退订。在这种情况下，模型的能力下限不再是无关紧要的东西，甚至是比能力上限更重要的一个评估指标。

### 响应速度

人类面对面交流时，在考虑肢体语言交流的情况下，响应延迟几乎为零。在流式传输技术的背景下，常规大模型的响应耗时等价于第一个 token 的生成和传输耗时，相对而言已经可以接受了。

然而对于推理模型，一个数十秒的响应耗时简直是标配。尽管人们对于推理模型更加包容，但推理加速仍然能够给用户一个更加良好的体验感。

### 推理与微调

在一般情况下，模型思考后产生的回答应该比不思考产生的回答有明显的提升。

然而在有些情况下，思考后的回答反而不及预期。这其中就涉及到推理模型背后的思维链技术。思维链的展开有两个关键点：展开方向和推理过程。

这两个关键点都是在通用数据上训练的，如果没有对于特定问题的深入的理解，思维链将按照错误的方向展开，或者在推理过程中产生严重的幻觉。因此对于特定细分领域还需要进行专业的训练，也就是所谓的微调。

微调这个技术并不新鲜，细分为全量微调和高效微调。全量微调会直接调整整个模型的参数，而高效微调则是新增参数并调整。全量微调面临严重的"灾难性遗忘"，而高效微调则面临调整不彻底的问题。并且微调技术还是无法与从头训练的模型能力相媲美。

### 持久记忆

目前大模型供应商提供的 API 都是不带记忆属性的。然而这种做法有利有弊。

当我们需要进行隔离操作的时候，这种无记忆的 API 能够非常方便地将不同的事务进行隔离，避免不同事务之间产生干扰。

然而当某一个事务需要持续跟进地理解、不断变化地调整，这种无记忆的 API 将会让用户比较为难：用户需要自行维护好上下文。就和写作文一样，这并不是一个容易的事。

于是衍生出了大量的基于上下文维护的商业应用，典型案例仍然是 Cursor。目前爆火的 Agent 模式也是类似的。

尽管 AI 编程算是比较成熟的领域，但经过深度体验之后仍然能够发现其中的问题所在：当需求越来越细化，工作的创新程度越来越高，大模型的响应准确率越来越低，甚至低到难以忍受。

并且由于代码并非操作者亲自编写，从正面来说减轻了人类的工作量，从反面来说这些代码一出生就是"孤儿"。AI 很难做到对它自己写的代码进行长时间的维护和迭代，特别是代码的长度远远超过自身上下文窗口的情况下。

尽管现在大模型的上下文窗口有足足 128k 的 token 长度，但经过 Cline 等类 Cursor 产品的实践，当上下文窗口的占用率超过 80%后就会产生严重的性能下降。在细分领域使用过程中，这种情况可能更加严重。

综上所述，如果想要一个稳定并且可维护的软件代码，仍然需要高质量人类程序员的时刻不停的监督：反复向 AI 强调项目的核心理念，反复强调代码的可维护性，审查一些 AI 自己都没意识到的硬编码问题……

从某种程度上来说，人类的工作量似乎并没有少，只是把编码的能力变成了审查代码的能力，把写代码的时间变成了 Debug 的时间😢

### 数据瓶颈

不管是什么 AI 模型，其能力的基础就是数据，而目前整个互联网的数据都已经被大模型消耗殆尽。

虽然目前有各种各样的尝试：为大模型添加多模态的数据，为模型创建可交互的虚拟环境(强化学习)，合成数据……

这里确实了解不多，因此就发表一些非常主观的预测。我自己更倾向于认为数据应该从现实中流式地获取，把训练数据的收集和训练本身合为一体，这样就能够利用大量的细分领域的一手知识来进行训练。这也能够解决"微调困难"的现状。

当然，这一想法可以说是将矛头直接指向了"灾难性遗忘"这个世纪难题。但鉴于人类本身就能够实现终生学习，因此"灾难性遗忘"应该并非一个不可解决的问题🤔

### 小结

大模型技术确实非常强悍，并且是最有希望成为下一次工业革命的核心的技术突破。然而，与人类自身的实时学习能力和适应性相比，在海量互联网数据上训练的大模型貌似有些"笨重"。

总之，大模型技术的落地还有很长的路要走。看见问题只是基础，只有"躬身入局"，在实践探索中才能收获真理🫡

这也是本系列的核心思想，AI 工程，就是要脱离空洞的理论猜测，到实践中去，到代码中去，到问题中去。

## 大模型基础理论

这部分内容其实就是大模型技术综述，如果你基础较好其实就不用看了。

当然，什么叫基础好呢？下面是一些自测问题，当然也是面试常考题：

1. 在草稿纸上写出 Transformer 模型的前向传播过程
2.  Encoder，Decoder 是什么意思？
3. 如何估计一个模型的表达能力？
4. 注意力机制是什么？层归一化是什么？
5. 语言模型如何依据概率进行采样？

相关文章：

- [大模型基础读书笔记]({{< ref "/blog/llm-foundation-notes/" >}})

## 模型训练

这部分涉及一些大模型训练过程中无法逃避的问题：

1. 为什么现在大部分模型都采用 Decoder-only 架构？
2. 如何估计模型需要的数据量？对于数据有什么要求？
3. 如何估计训练时长？
4. 如何评估训练和部署两个阶段的算力需求？
5. 不同的模型微调方法如何选择？模型微调需要的算力如何估计？

相关文章：

- [LLM训练实战]({{< ref "/blog/llm-training-playbook/" >}})

## 模型部署



## 应用开发

这部分主要记录在在基础模型之上，以应用开发为导向的大模型实践。

### 一些感想

当我们已经习惯了随手点开一个 AI 聊天助手：豆包、Kimi、DeepSeek……功能如此简单的一个东西，貌似自己写一个也不会很困难吧？

这就是属于我的那个"梦开始的地方"😢听着很扯，但我当时真是这么想的。然后就有了后面漫长的开发之旅。

如果你也有和我类似的想法，那么下面的内容可能对你有所帮助🤗

在熬过了这么多夜之后，我必须承认"构建一个 LLM 应用"是一个非常庞大的话题，特别是对于一个独立开发者而言。如果你选择了做独立开发或者组建一个微型团队，那么也同时意味着你选择了一个艰辛的道路：没有"前辈"帮你探索技术路线，一切的一切都要靠你自己去摸索。

**但是**，这也是独立开发的乐趣所在：你能够完全掌控你的产品的走向，看着它一点一点迭代优化。

你可能会问："产品做出来效果不好怎么办？"；"开发不下去了怎么办？"；"如果没什么人用那我的时间不是打水漂了？"🤔

不得不说，独立开发的投入真的很高昂，而且上述疑问都是无法避免的，甚至对于大部分独立开发者来说是必然会经历的。

但是，如果你真的认真对待了"独立开发"这件事情，不管结果如何，我相信你都有所收获：如果你成功了，我在这里对你表达祝贺🥳你的辛勤付出有了回报；如果你失败了、放弃了，我理解你的不甘与沮丧，甚至是愤怒。

但是没人能够在对自己过往的全盘否定中收获快乐与成功，暂时忘掉那些伤痛，继续下一步的人生吧。

人生没有白走的路，每一步都算数🫡

### 发现需求

草稿草稿草稿……

### 探索实现方案

草稿草稿草稿……

### 项目构建

草稿草稿草稿……

### 发布服务

- [云服务搭建]({{< ref "/blog/cloud-server-build/" >}})
