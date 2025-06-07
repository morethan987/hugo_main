---
title: Notes on the Foundation of Large Models
weight: -110
draft: true
description: Notes written after reading the textbook "Foundation of Large Models" from Zhejiang University
slug: llm-foundation-notes
tags:
  - LLM
  - EngineeringPractice
  - Notes
series:
  - AI Engineering
series_order: 2
date: 2025-05-28
lastmod: 2025-05-28
authors:
  - Morethan
---
{{< lead >}}
Notes written after reading the textbook "Foundation of Large Models" from Zhejiang University
{{< /lead >}}

## Preface

The development of large model technology can be described as "rapidly evolving," with seemingly groundbreaking advancements happening every day. 🤔

In reality, how effective are these "groundbreaking advancements"? What is their actual value in engineering practice? Ultimately, these questions must be traced back to those "ordinary" classic principles.

It's certainly a good thing to keep up with the latest developments in large model technology: it helps us understand research directions and broaden our thinking. However, we should not only focus on the ever-evolving top-tier technologies but also pay attention to the underlying, unchanging fundamentals behind them.

This is the significance of taking the time to read textbooks in an era of rapid technological advancement.

This article primarily focuses on the book [“Foundation of Large Models”](https://github.com/ZJU-LLMs/Foundations-of-LLMs) written by Zhejiang University, which is still being continuously updated.

## Basics of Language Models

This is the most commonly taught knowledge in universities today: n-gram based statistical language modeling, RNN-based language models, and Transformer-based language models. Below is a flowchart of the Transformer architecture.

![Transformer](Transformer.png "引用自第 16 页")

The model architecture doesn't have much to say, but there are a few interesting questions that were a bit unclear to me before reading the book but became immediately clear afterward.

---

Q: It's said that Transformer is a parallel computing architecture, so why can tokens only be generated one after another in a serial manner?

A: The parallel computing of the Transformer is reflected in the processing of known sequences, where all known token vectors are fed into the model in parallel for processing. However, the token generation process remains serial and autoregressive.

---

Q: Does swapping the order of residual connections and layer normalization affect the model?

A: Networks that place layer normalization after residual connections are called Post-LN, while those that place layer normalization before residual connections are called Pre-LN. Post-LN has a stronger ability to handle representation collapse but is slightly weaker at addressing gradient vanishing, whereas Pre-LN is the opposite.

Q: What does representation collapse mean?

A: During training, the learned representations of the model become less discriminative, meaning different input samples are mapped to similar or even identical feature vectors, causing the model to lose its ability to distinguish between different samples.

---

Q: Why can the Encoder and Decoder parts of the Transformer be used independently to build a language model?

A:

---



## Architecture of Large Language Models

## Parameter-Efficient Fine-Tuning

## Model Editing
