---
title: Notes on LLM Engineering Practices
weight: -65
draft: true
description: Documenting key knowledge points in the process of LLM engineering practices
slug: llm-engineering-note
tags:
  - LLM
  - Notes
  - EngineeringPractice
series:
  - AI Engineering
series_order: 0
date: 2025-05-28
lastmod: 2025-05-28
authors:
  - Morethan
---
{{< lead >}}
This blog post is the zeroth article in the "AI Engineering" series. Besides serving as an index for the entire series, it also summarizes relevant knowledge points from practical experiences with large models.
{{< /lead >}}

## Introduction



## Foundation of Large Models

This section is essentially a review of large model technologies, so if you have a solid foundation, you might not need to read it.

Of course, what does having a solid foundation mean? Below are some self-assessment questions, which are also commonly asked in interviews:

1. Write out the forward propagation process of the Transformer model on scratch paper.
2. What do Encoder and Decoder mean?
3. How do you estimate the expressive power of a model?
4. What is the attention mechanism? What is layer normalization?
5. How do language models sample based on probability?

Related articles:

- [Large Model Foundation Reading Notes]({{< ref "/blog/llm-foundation-notes/" >}})

## Model Training

This part involves some unavoidable issues in the training process of large models:

1. Why do most models today adopt a Decoder-only architecture?
2. How to estimate the amount of data a model needs? What are the requirements for the data?
3. How to estimate the training duration?
4. How to evaluate the computational power requirements for the training and deployment phases?
5. How to choose among different model fine-tuning methods? How to estimate the computational power needed for model fine-tuning?

Related articles:

- [Practical LLM Training]({{< ref "/blog/llm-training-playbook/" >}})

## Model Deployment



## Application Development

This section mainly documents the practical experiences of large models oriented towards application development based on foundational models.

### Some Reflections

When we have become accustomed to casually opening an AI chat assistant: DouBao, Kimi, DeepSeek... something that appears so simple, it might seem like creating one ourselves wouldn't be too difficult, right?

This was my "dream starting point"😢 It sounds ridiculous, but I really thought that at the time. And then came the long journey of development.

If you have similar thoughts, the content below may be helpful to you🤗

After staying up so many nights, I must admit that "building an LLM application" is a very extensive topic, especially for an independent developer. If you choose to do independent development or form a small team, it also means you've chosen a tough path: there's no "senior" to help you explore the technical route; everything has to be figured out by yourself.

**However**, this is where the fun of independent development lies: you can fully control the direction of your product and watch it gradually iterate and improve.

You might ask: "What if the product doesn't perform well?"; "What if the development gets stuck?"; "If no one uses it, isn't my time wasted?"🤔

It must be said that the investment in independent development is truly substantial, and these questions are unavoidable, even inevitable for most independent developers.

But if you really take "independent development" seriously, regardless of the outcome, I believe you will gain something: if you succeed, congratulations to you here🥳 Your hard work has paid off; if you fail or give up, I understand your frustration and even anger.

But no one can find happiness or success in completely negating their past. Forget those pains for now and move on to the next step in life.

There are no wrong turns in life; every step counts🫡

### Identifying Needs

Draft draft draft...

### Exploring Implementation Solutions

Draft draft draft...

### Project Construction

Draft draft draft...

### Service Release

- [Cloud Service Deployment]({{< ref "/blog/cloud-server-build/" >}})
