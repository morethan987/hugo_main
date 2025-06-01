---
title: Notes on LLM Engineering Practices
weight: -65
draft: true
description: Recording important knowledge points in the process of LLM engineering practices
slug: llm-engineering-note
tags:
  - LLM
  - EngineeringPractice
  - Notes
series:
  - AI Engineering
series_order: 0
date: 2025-05-28
lastmod: 2025-06-01
authors:
  - Morethan
---
{{< lead >}}
This blog post is the zeroth article in the "AI Engineering" series. Besides serving as an index for the entire series, it also summarizes key knowledge points related to practical large model applications.
{{< /lead >}}

## Introduction

Currently, large model technology is developing rapidly. Open-source base models like DeepSeek are gradually catching up with closed-source models, and general-purpose large models are achieving increasingly higher accuracy across various tasks, pushing their capability limits higher.

However, these upstream technological breakthroughs do not necessarily mean a paradigm shift in downstream industries. Truly "deployable" large model projects remain few and far between—only the AI programming field has reached relative maturity, while other areas are far from being practically usable.

### Capability Jaggedness

The upper limit of large model capabilities continues to rise, but this does not mean that the lower limit is improving accordingly. Although large models have made breakthroughs in cutting-edge mathematics and science, they still fail to solve some basic problems.

This is known as the "capability jaggedness" phenomenon: the gap between the upper and lower limits of capabilities is too large. This issue manifests differently in different scenarios.

In the code domain, it is almost imperceptible because simple errors can be quickly resolved by programmers themselves, and they generally have the patience to fix them (or must have the patience 😌). In such cases, the requirement for the model's lower limit is not high.

In certain enterprise application scenarios, even minor errors can cause significant commercial losses. A typical example is when Cursor's Q&A model recently produced "hallucinations," misinterpreting user service terms and causing a large number of users to unsubscribe. In such cases, the model's lower limit is no longer trivial; it may even be a more important evaluation metric than the upper limit.

### Response Speed

In face-to-face human communication, response delays are almost zero when considering non-verbal communication. Under the backdrop of streaming technology, the response time of conventional large models is equivalent to the generation and transmission time of the first token, which is relatively acceptable.

However, for reasoning models, a response time of tens of seconds is standard. Although people are more tolerant of reasoning models, accelerating reasoning still provides users with a better experience.

### Reasoning and Fine-tuning

In general, answers generated after model reasoning should show noticeable improvement over those without reasoning.

However, in some cases, reasoned answers fall short of expectations. This involves the chain-of-thought technology behind reasoning models. The two key aspects of chain-of-thought expansion are direction and reasoning process.

Both of these key aspects are trained on general data. Without a deep understanding of specific problems, the chain-of-thought may expand in the wrong direction or produce severe hallucinations during reasoning. Therefore, specialized training, or fine-tuning, is required for specific niche fields.

Fine-tuning is not a new technique and can be divided into full fine-tuning and efficient fine-tuning. Full fine-tuning directly adjusts all parameters of the model, while efficient fine-tuning adds new parameters and adjusts them. Full fine-tuning faces the serious issue of "catastrophic forgetting," while efficient fine-tuning struggles with incomplete adjustments. Moreover, fine-tuning techniques still cannot match the capabilities of models trained from scratch.

### Persistent Memory

Currently, APIs provided by large model vendors do not include memory attributes. This approach has both advantages and disadvantages.

When isolation operations are needed, this memory-less API can conveniently isolate different tasks, avoiding interference between them.

However, when a task requires continuous understanding and adjustment, this memory-less API becomes problematic for users: they need to maintain the context themselves. This is akin to writing an essay—it’s not an easy task.

As a result, numerous commercial applications based on context maintenance have emerged, with Cursor being a prime example. The currently popular Agent pattern is similar.

Although AI programming is considered a relatively mature field, after in-depth experience, issues still become apparent: as demands become more detailed and work innovation increases, the accuracy of large model responses decreases, sometimes to an intolerable level.

Moreover, since the code is not written by the operator personally, on the positive side, it reduces human workload, but on the negative side, these codes are "orphans" from the moment they are born. AI finds it difficult to maintain and iterate on its own code over long periods, especially when the code length far exceeds its context window.

Although current large models have a context window of up to 128k tokens, through the practice of products like Cline and Cursor-like tools, performance significantly degrades when the context window occupancy exceeds 80%. This situation may be even more severe in niche fields.

In summary, if you want stable and maintainable software code, high-quality human programmers must continuously supervise: repeatedly emphasizing the core concepts of the project, stressing code maintainability, and reviewing hard-coded issues that AI itself might not recognize...

In a sense, human workload doesn't seem to decrease; coding ability has turned into code review ability, and the time spent writing code has become debugging time😢

### Data Bottleneck

Regardless of what kind of AI model, its foundation is data, and currently, all internet data has been consumed by large models.

There are various attempts now: adding multi-modal data to large models, creating interactive virtual environments for models (reinforcement learning), synthesizing data...

I don’t know much about this area, so I will make some very subjective predictions. Personally, I am more inclined to believe that data should be streamed from reality, combining the collection of training data with the training process itself. This way, a large amount of first-hand knowledge from niche fields can be used for training. This could also address the current difficulty of fine-tuning.

Of course, this idea essentially points directly at the century-old problem of "catastrophic forgetting." However, given that humans themselves can achieve lifelong learning, "catastrophic forgetting" should not be an unsolvable problem🤔

### Summary

Large model technology is indeed very powerful and is the most promising core technological breakthrough for the next industrial revolution. However, compared to real-time learning and adaptability of humans, large models trained on massive amounts of internet data appear somewhat "clumsy."

In short, there is still a long way to go for the implementation of large model technology. Identifying problems is just the foundation; only by "getting personally involved" can we uncover the truth. 🫡 

This is also the core idea of this series: AI engineering is about moving away from empty theoretical speculation and into practice, into the code, and into the problems.

## Large Model Basic Theory

This section is actually a summary of large model technology, so if you already have a good foundation, you don't need to read it.

What exactly constitutes a "good foundation"? Below are some self-assessment questions, which are also common interview questions:

1. Write out the forward propagation process of the Transformer model on a piece of paper.
2. What do Encoder and Decoder mean?
3. How do you estimate the expressive power of a model?
4. What is the attention mechanism? What is layer normalization?
5. How do language models sample based on probability?

Related articles:

- [Large Model Foundation Reading Notes]({{< ref "/blog/llm-foundation-notes/" >}})

## Model Training

This section covers some unavoidable issues in the process of large model training:

1. Why do most models today use a Decoder-only architecture?
2. How do you estimate the amount of data a model needs? What are the requirements for the data?
3. How do you estimate training duration?
4. How do you evaluate computational power needs for both training and deployment phases?
5. How do you choose different model fine-tuning methods? How do you estimate the computational power required for fine-tuning?

Related articles:

- [Practical LLM Training]({{< ref "/blog/llm-training-playbook/" >}})

## Model Deployment



## Application Development

This section mainly records large model practices oriented towards application development on top of foundational models.

### Some Reflections

When we have already gotten used to casually opening an AI chat assistant: DouBao, Kimi, DeepSeek... Such a seemingly simple thing appears as though it wouldn't be too difficult to create one ourselves, right?

This was my "dream starting point"😢 It sounds ridiculous, but that's exactly what I thought at the time. And thus began a long journey of development.

If you have thoughts similar to mine, the content below might be helpful to you🤗

After staying up so many nights, I must admit that "building an LLM application" is a very vast topic, especially for an independent developer. If you choose to develop independently or form a small team, it simultaneously means you've chosen a tough path: there's no "mentor" to help you explore the technical route; everything must be figured out by yourself.

**However**, this is also the fun of independent development: you have complete control over the direction of your product, watching it evolve and optimize step by step.

You might ask: "What if the product turns out poorly?" "What if development stalls?" "If no one uses it, isn't my time wasted?"🤔

It must be said that the investment in independent development is truly substantial, and the aforementioned questions are unavoidable, even inevitable for most independent developers.

But if you really take "independent development" seriously, regardless of the outcome, I believe you will gain something: if you succeed, congratulations here! Your hard work has paid off🥳 If you fail or give up, I understand your frustration and disappointment, even anger.

But no one can find joy or success in a complete denial of their past. Forget those pains for now and move on to the next step in life.

No road in life is walked in vain; every step counts🫡

### Discovering Needs

Draft draft draft...

### Exploring Implementation Solutions

Draft draft draft...

### Project Construction

Draft draft draft...

### Publishing Services

- [Cloud Service Deployment]({{< ref "/blog/cloud-server-build/" >}})
