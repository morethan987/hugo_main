---
title: RAG技术调研报告
weight: -70
draft: true
description: 详细调研了各种主流RAG技术的实现原理以及典型产品
slug: rag-report
language: zh-cn
tags:
  - AI
  - Web
series:
  - 项目报告
series_order: 3
date: 2025-03-04
lastmod: 2025-03-04
authors:
  - Morethan
---

## 实际问题

一个技术的出现都是基于实际问题的。RAG 技术的出现主要是为了应对生成式预训练语言模型的如下缺陷：

1. **知识局限**：大语言模型在训练完成后，其权重参数被固定，模型的性能和表现主要依赖于训练阶段的学习成果，时效性不强
2. **知识更新成本**：传统的全参数训练方法成本极其高昂，难以将快速产生的新数据迅速集成到模型的参数中
3. **幻觉问题**：大语言模型的输出基于概率，因此可能会产生与事实不符的“幻觉”输出

因此大语言模型在实际生产实践中需要一个技术，能够**低成本、准确、快速**更新模型的的专业知识。

## 解决方案

针对上述问题，学术界提出了不少的解决方案。这些解决方案其实都是围绕着大语言模型产业链的各个环节来进行的：

1. 从**数据**层面入手，扩展模型的知识来源
2. 从**底层架构**入手，改进 Transformer 架构，让其内部参数能够实时动态调整
3. 从**后训练过程**入手，引入模型微调技术
4. 从**查询**层面入手，根据用户查询从外部数据库中检索出相关知识，进行增强查询

原始的大模型工作流：

{{< mermaid >}}
graph LR;

p1(Data)-->p2(Transformer)-->p4(Generation);
p3(Query)-->p2;
{{< /mermaid >}}

改进后的工作流：

{{< mermaid >}}
graph LR;
datatype1(File);datatype2(Image);datatype3(Audio);datatype4(HTML);data(data);
datatype1-->data;datatype2-->data;datatype3-->data;datatype4-->data;

architecture1(Transformer);architecture2(Dynamic Transformer);architecture3(Fine-tuned Transformer);
data-->architecture1;data-->architecture2;

query(Query);retrieval(Retrival);
query-->retrieval-->architecture3;
retrieval-->architecture2;

finetuning(Fine tuning);data2(Specific data);
architecture1-->finetuning-->architecture3;
data2-->finetuning;

generation1(Generation);generation2(Generation);
architecture3-->generation1;
architecture2-->generation2;
{{< /mermaid >}}

## 学术定义

**RAG**（Retrieval-Augmented Generation，检索增强生成）是一种结合信息检索技术与语言生成模型的人工智能技术。它通过从外部知识库中检索相关信息，并将其作为提示输入给大型语言模型（LLMs），以增强模型处理知识密集型任务的能力，如问答、文本摘要、内容生成等。RAG模型由Facebook AI Research（FAIR）团队于2020年首次提出，并迅速成为大模型应用中的热门方案。原文链接：[Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401)
