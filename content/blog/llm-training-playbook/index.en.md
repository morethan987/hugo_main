---
title: Practical LLM Training
weight: -105
draft: true
description: Documents various engineering aspects related to LLM training.
slug: llm-training-playbook
language: en
tags:
  - Architecture
  - LLM
  - EngineeringPractices
series:
  - AI Engineering
series_order: 0
date: 2025-05-28
lastmod: 2026-06-24
authors:
  - Morethan
---
{{< katex >}}

{{< lead >}} Once you understand how a network architecture is implemented, a natural question arises: how do I actually train a model like this? This section compiles all sorts of engineering know-how that answers that very question. {{< /lead >}}

## Preface

Large model architectures come in all shapes and sizes, but if you really want to understand one, you have to get your hands dirty. There are plenty of high-quality online courses on YouTube to learn from— [CS336: Language Modeling from Scratch](https://cs336.stanford.edu/) being a prime example—so this article essentially serves as my notes on CS 336.

"Practice is the sole criterion of truth." 🫡

The content can be broadly divided into five parts:

1. Fundamentals: tokenizers, resource inventory, model architectures, training strategies
2. Systems engineering: kernels, parallelism, inference
3. Scaling laws
4. Data engineering: pretraining data engineering—messy, but absolutely critical
5. Model alignment: RL post-training fine-tuning

## Fundamentals

### Tokenizers

Karpathy has an excellent [video](https://youtu.be/zduSFxRajkE?si=X3hpizNlFcFiIXQt) on tokenizers that dives deep into everything you need to know about them.

Functionally speaking, a tokenizer is a peripheral component independent of the Transformer body. Its main job is to encode a long string of text into integers—the data type that Transformer models actually process: a sequence of integers.

A rather interesting question to ponder: from a low-level perspective, if the only goal is to map characters to integers, digitized text doesn't really need explicit encoding, because the vast majority of digital text is already UTF-8 encoded—these characters are natively represented as integers at the hardware level.

```python
>>> test_string = "hello, 世界！"
>>> utf8_encoded = test_string.encode("utf-8")
>>> print(utf8_encoded)
b'hello, \xe4\xb8\x96\xe7\x95\x8c\xef\xbc\x81'
>>> list(utf8_encoded)
[104, 101, 108, 108, 111, 44, 32, 228, 184, 150, 231, 149, 140, 239, 188, 129]
```

As you can see from the example above, a very short piece of text gets encoded into 16 integers—clearly far too granular. A Transformer's attention window is limited in size and extremely expensive, so the tokenizer's purpose is to chunk and aggregate this fragmented low-level encoding, thereby shortening the length of the integer sequence and improving the model's computational efficiency.

> [!NOTE] Food for thought
> A question worth exploring: assuming we had abundant compute, would training directly on raw UTF-8 sequences yield better results? Put another way, we could view the tokenizer as a simple spatial mapping that projects data from the noisy space of UTF-8 encoding into a more semantically meaningful space. We already know this mapping is effective—after all, every major LLM is trained this way—but how much does it actually improve things, and can we quantify that theoretically?

In terms of concrete algorithms, most models use the BPE algorithm that OpenAI originally adopted, with tiktoken being the representative library. Google's sentencepiece is also widely used across many large models, though configuring it is considerably more complex compared to tiktoken.

### Resource Inventory

Broadly, available resources can be categorized as: per-unit-time compute capacity (FLOP/s), time, data volume, memory size, and the compute cost of tensor operations. The purpose of a resource inventory is to estimate training time—or the maximum model size you can train—given known resource constraints.

#### Memory Footprint

Almost all data is stored using tensors—arrays of floating-point numbers arranged according to certain rules. Depending on the floating-point format used, the memory footprint of model training or inference varies. In general, floating-point formats include the following:

1. float 32: 32-bit float, occupying 4 bytes of memory

![img/float32.png](img/float32.png)

2. float 16: 16-bit float, occupying 2 bytes of memory. Its dynamic range is smaller than float 32, which can cause numerical underflow when gradients produced during backpropagation become very small. (This is precisely why bfloat 16 was introduced.)

![img/float16.png](img/float16.png)

3. bfloat 16: brain floating point, also occupying 2 bytes. It sacrifices precision to expand the dynamic range, minimizing weird numerical overflow issues. (In deep learning, precision matters far less.)

![img/bfloat16.png](img/bfloat16.png)

In practice, mixed-precision training is the norm: float 32 for optimizer states, bfloat 16 for parameters, activations, and gradients. Of course, there are more aggressive precision formats like float 8 and float 4, but they're generally not used for actual model training. And when float 4 is used, it doesn't directly represent a single parameter value; instead, neighboring parameters are packed together and share a scaling factor. This trick relies on the local similarity of parameters—adjacent parameters tend to have similar orders of magnitude, so you can factor out the shared scale and use a batch of low-precision numbers plus the scaling factor to represent a batch of high-precision numbers.

> [!NOTE] Difference from quantization
> Quantization aligns a model trained at high precision to a lower-precision space. It's considerably easier to pull off than training directly at low precision.

#### Tensor Computation

Let's look at the computational cost of tensor multiplication, using the following diagram as an example.

![img/tensor_multipule.png](img/tensor_multipule.png)

The total computational cost, in a nutshell: the resulting tensor has the shape \(B\times K\), giving us \(B\cdot K\) elements in total. Each element is the sum of \(D\) pairs of element-wise multiplications, meaning every element in the final result represents \(2D\) multiplication or addition operations. So the total computational cost is: \(2D\cdot B\cdot K\).

> [!NOTE] A clarification
> This is the simplest case, with no hardware-level optimizations factored in—for instance, all additions being fused into a single operation.

There's another important engineering metric to introduce here: MFU (Model FLOPs Utilization), which measures the actual compute utilization when the model is running. Simply put, in real-world engineering implementations, the GPU's total compute power is never 100% utilized—MFU captures that actual utilization rate.

### Training Strategies
