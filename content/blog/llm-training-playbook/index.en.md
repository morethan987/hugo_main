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
lastmod: 2026-09-04
authors:
  - Morethan
---
{{< katex >}}

{{< lead >}} Once you understand how a network architecture is implemented, a natural question arises: how do I actually train a model like this? This section compiles all sorts of engineering know-how that answers that very question. {{< /lead >}}

## Preface

Large model architectures come in all shapes and sizes, but if you really want to understand one, you have to get your hands dirty. There are plenty of high-quality online courses on YouTube to learn from— [CS336: Language Modeling from Scratch](https://cs336.stanford.edu/) being a prime example—so this article essentially serves as my notes on CS 336. Here is my [code implementation](https://github.com/morethan987/cs336-assignment1-basics) and [Answers for Problems](https://github.com/morethan987/cs336-assignment1-basics/blob/main/Q%26A.md).

"Practice is the sole criterion of truth." 🫡

The content can be broadly divided into five parts:

1. [Fundamentals]({{< relref "#fundamentals" >}}): tokenizers, resource inventory, model architectures, training strategies
2. [Systems engineering]({{< relref "#systems-engineering" >}}): kernels, parallelism, inference
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

#### Detailed Questions

1. Why use bytes encoding instead of utf-8 encoding directly?

Ans: Actuall text data distribution on utf-8 encoding vocabulary is sparse while bytes encoding only has 256 vocabularies which means every vocabulary is amply sampled. However, bytes encoding is not perfect due to extremelly long seqeunce it produces, which is quite a burdon for model training.

2. Why pre-tokenize before BPE?

Ans: Original BPE needs to walk through whole corpus every merge leading to an overwhelming computational cost. Besides, there are some exact boundaries we don't expect the merging happen at, such as cross-word merging(e.g. "there" and "is" should be always two tokens) and trivial punctuations(e.g. dog! vs dog.).

3. How do you handle large datasets under memory constraints?

A: When training on massive datasets with limited RAM, the key is leveraging lazy-loading file reading mechanisms like `mmap` to effectively cap peak memory usage. When necessary, you should implement a full-pipeline streaming lazy-loading strategy—covering everything from file reading and encoding all the way to writing back to disk. Additionally, data structures and algorithms play a huge role in the tokenization stage; in my personal experience, priority queues and hash tables are the ones I rely on most.

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

Here is a rough memory footprint estimation for an MLP. Assuming the input, activations, and output all have a fixed dimension \(D\), the layer count is \(L\), and the batch size is \(B\), the total parameter count can be estimated as \(D^2\cdot L\) —meaning \(L\) layers of \(D\times D\) square matrices. Parameter memory occupies \(2D^2\cdot L\) bytes assuming each parameter is stored in bf 16 format. Gradient memory usage is identical to parameters (since only parameters require gradients, and every parameter needs one gradient value). Optimizer states require \(4D^2\cdot L\) bytes because optimizer states generally use fp 32 to maintain precision (for instance, the \(g^2\) parameter in the AdaGrad optimizer acts as the **denominator** to regulate the learning rate, requiring higher precision). Finally, activation memory matches that of the parameters at \(2D^2\cdot L\) bytes.

> [!NOTE] Note
> Optimizer states vary depending on the optimizer; for example, the Adam optimizer allocates 8 bytes of memory per parameter to store the first and second momentum values.

Some methods to reduce memory footprint include gradient accumulation—where a `micro_batch_size` parameter is used to compute and accumulate gradients across \(\frac{\text{batch\_size}}{\text{micro\_batch\_size}}\) steps before updating parameters—and selective activation storage, where you might only keep pre-activation data instead of both pre- and post-activation data, recomputing post-activation values on the fly when needed. More aggressive strategies even omit caching across matrix multiplications, though this will undoubtedly slow down training speed.

---

Here is the detailed upper bound on the peak memory for several characteristic modules in Transformer. Since this is an upper bound estimation, we simply sum up all the data that needs to be retained during training (in actual execution, dynamic memory deallocation occurs, making the actual footprint smaller than the upper bound) and ignore optimizations that use recomputation to reduce memory (during gradient calculation, some intermediate variables are recomputed because the recomputation overhead is smaller).

Overall, peak memory can be divided into four parts: trainable parameters (weights), optimizer states (depending on the type of optimizer; AdamW is used by default below), gradients, and activations. The first three are easy to understand: trainable parameters are explicitly defined in the code; for the AdamW optimizer, each parameter requires first and second moments, which is twice the number of trainable parameters; and for gradients, every trainable parameter has a corresponding gradient. But what exactly are activations? Activations are the intermediate variables (such as input tensors) required during gradient calculation. They are tightly coupled with the gradient computation process, and dynamic recomputation mechanisms can be used to save memory; therefore, the output of each elementary operation is generally accounted for as an activation.

**RMSNorm**: Contains a trainable scaling parameter of size \(d_{\text{model}}\), the optimizer occupies \(2 \times d_{\text{model}}\) parameters, gradients require \(d_{\text{model}}\) parameters, and the activation only needs the input tensor, which is \(\text{batch\_size}\times \text{context\_length}\times d_{\text{model}}\).

**Multi-Head Self-Attention**: Assuming \(d_{k}=\frac{d_{\text{model}}}{\text{num\_heads}}\) by default, the trainable parameters are the three matrices \(W_{Q}\), \(W_{K}\), and \(W_{V}\), each having a shape of \(d_{\text{model}}\times d_{\text{model}}\); the optimizer states take twice the number of trainable parameters, i.e., \(6\times d_{\text{model}}\times d_{\text{model}}\) parameters; gradients match the trainable parameters with \(3\times d_{\text{model}}\times d_{\text{model}}\) parameters; activations include the input tensor \(\text{batch\_size}\times \text{context\_length}\times d_{\text{model}}\), the three projection tensors \(Q\), \(K\), and \(V\) (each being \(\text{batch\_size}\times\text{context\_length}\times d_{\text{model}}\)), the attention scores and softmax (both producing tensors of shape \(\text{batch\_size}\times \text{num\_heads} \times \text{context\_length}\times \text{context\_length}\), the quadratic term), the weighted linear sum \(\text{batch\_size}\times \text{context\_length}\times d_{\text{model}}\), and the output linear projection which is also \(\text{batch\_size}\times \text{context\_length}\times d_{\text{model}}\).

**FFN**: Generally \(d_{ff} = \frac{8}{3}\times d_{\text{model}}\), so the number of trainable parameters is \(3 \times (\text{d\_model} \times d_{ff}) = 8 \times \text{d\_model}^2\); gradients and optimizer states are \(8 \times \text{d\_model}^2\) and \(16 \times \text{d\_model}^2\), respectively; activations include the input tensor \(\text{batch\_size}\times \text{context\_length}\times d_{\text{model}}\) and the computed outputs of the three projection matrices \(3 \times (\text{batch\_size} \times \text{context\_length} \times \frac{8}{3} \text{d\_model}) = 8 \times \text{batch\_size} \times \text{context\_length} \times \text{d\_model}\).

#### Tensor Computation

Let's look at the computational cost of tensor multiplication, using the following diagram as an example.

![img/tensor_multipule.png](img/tensor_multipule.png)

The total computational cost, in a nutshell: the resulting tensor has the shape \(B\times K\), giving us \(B\cdot K\) elements in total. Each element is the sum of \(D\) pairs of element-wise multiplications, meaning every element in the final result represents \(2D\) multiplication or addition operations. So the total computational cost is: \(2D\cdot B\cdot K\).

> [!NOTE] A clarification
> This is the simplest case, with no hardware-level optimizations factored in—for instance, all additions being fused into a single operation.

There's another important engineering metric to introduce here: MFU (Model FLOPs Utilization), which measures the actual compute utilization when the model is running. Simply put, in real-world engineering implementations, the GPU's total compute power is never 100% utilized—MFU captures that actual utilization rate.

During backpropagation, because gradients must be computed for both operands involved in the forward pass (the activations and the weights), the backward pass requires roughly twice as much compute as the forward pass.

### Model Architecture

As a model introduced back in 2017, the standard Transformer is, in a sense, showing its age—which is why Transformer models implemented in actual production are modernized variants. A high-level comparison of these modifications is shown below.

![transformer_original](img/transformer_original.png "Standard Transformer Architecture" )

![transformer_modern](img/transformer_modern.png "Modernized Transformer Architecture" )

A noteworthy point in the final output stage is the shape of the output tensor. Empirically, one might expect the final result to be a probability distribution over the entire vocabulary, i.e., a tensor of shape \(\text{batch\_size}\times \text{vocab\_size}\). In reality, however, the output shape is \(\text{batch\_size} \times \text{seq\_len} \times \text{vocab\_size}\), which means that the distribution of the next token is predicted for every token position (this is precisely where the highly parallel training capability of the Transformer originates). You might wonder: wouldn't this be very wasteful during inference? In fact, inference only requires the probability distribution at the very last token position. Indeed, directly copying the training logic would be quite wasteful; however, inference utilizes KV Cache technology, so this waste only occurs when generating the **first** token. During subsequent token-by-token generation, the sequence length is directly kept fixed at 1, as there is simply no need to repeatedly feed the preceding sequence into the model.

#### Normalization

First, Post-Normalization was replaced by Pre-Normalization, which is far more beneficial for numerical stability. The key lies in removing normalization from the residual stream—Layer Normalization can be placed either before or after the linear module, or even applied as "Double-Norm".

Next comes the choice of normalization function. Early LLMs generally used the standard LayerNorm formulation, such as in GPT-1/2/3:


$$
y=\gamma\frac{x-E(x)}{\sqrt{ Var(x)+\epsilon }} + \beta
$$

More recent models, however, opt for RMSNorm, which completely eliminates the mean and bias terms:


$$
y=\gamma\frac{x}{\sqrt{ \left \| x \right \|_{2}^2 }}
$$

So why use RMSNorm? Because while there is virtually no difference in final training performance (with RMSNorm sometimes even performing slightly better), RMSNorm is noticeably faster. But why optimize this module in the first place? Mathematically, normalization accounts for a meager \(0.17\%\) of total FLOPs, making optimization seem unnecessary at first glance; in reality, however, normalization accounts for up to \(25.5\%\) of actual execution time due to heavy data movement overhead.

Finally, bias terms in linear layers are discarded. The original FFN layer expression was:


$$
\text{FFN}(x) = max(0, xW_{1}+b_{1})W_{2}+b_{2}
$$

However, modern implementations generally use:


$$
\text{FNN}(x) = \sigma(xW_{1})W_{2}
$$

The main reason for this change is to save memory and improve training stability. See the [Training Stability]({{< relref "#training-stability" >}}) section for details regarding training stability.

#### Activation Functions

A wide variety of activation functions have emerged, but selecting the right one still requires careful consideration. Modern LLM models generally choose activation functions with gating mechanisms. The so-called gating mechanism is essentially an element-wise multiplication (Hadamard product, \(\otimes\)). Below is a concrete comparative example. The classic FF layer expression is as follows:


$$
\text{FF} = max(0, xW_{1})W_{2}
$$

In contrast, ReGLU adds a gating mechanism to the ReLU section using an additional set of parameters:


$$
\text{FF}_{\text{ReGLU}}(x, W_{1}, V, W_{2}) = (max(0, xW_{1})\otimes xV)W_{2}
$$

Each element of the parameter matrix \(V\) acts on the corresponding ReLU output, amplifying or attenuating specific activation values. Similarly, the expressions for GeGLU and SwiGLU activation functions are:


$$
\text{FF}_{\text{GeGLU}}(x, W_{1}, V, W_{2}) = (\text{GeLU}(xW_{1})\otimes xV)W_{2}
$$


$$
\text{FF}_{\text{SwiGLU}}(x, W_{1}, V, W_{2}) = (\text{Swish}(xW_{1})\otimes xV)W_{2}
$$

where \(\text{Swish}(x) = x\cdot\text{sigmoid}(x)\) is a foundational activation function similar to GeLU and ReLU.

Experiments ultimately show that gated activation functions are indeed effective. Although the gain is modest—such as a less than \(1\%\) improvement from ReLU's \(83.80\%\) to ReGLU's \(84.67\%\), this gain comes with virtually no extra cost (both VRAM and compute overhead are minimal). So why not enjoy a free lunch?

> [!NOTE] Note
> Because gated activation functions introduce an extra projection parameter matrix, the hidden dimensions of these projection matrices are typically set to \(\frac{2}{3}\) of non-gated activation functions to maintain parameter parity.

#### Layer Parallelism

Transformer blocks in standard LLMs are typically stacked sequentially, expressed as:


$$
y = x+\text{MLP}(\text{Norm}(x+\text{Attention}(\text{Norm}(x))))
$$

However, the GPT-J model proposed a parallel computation layout for Transformer modules:


$$
y = x + \text{MLP}(\text{Norm}(x)) + \text{Attention}(\text{Norm}(x))
$$

Although the GPT-J paper states that this parallel approach yields a \(15\%\) speedup in large-scale training with controllable quality loss at the 8 B scale and zero loss at the 62 B scale—making its impact on model quality largely neutral—the vast majority of models still stick to sequential Transformer blocks. After all, logically speaking, this parallel layout effectively cuts the depth of the model in half, which represents a rather risky architectural trade-off.

#### Positional Encoding

Currently, the most mainstream positional encoding technique is **Rotary Position Embedding** (RoPE), which is a relative positional encoding scheme independent of absolute token positions. Why do we need relative positional encoding? Because the attention mechanism in Transformers logically ought to depend only on the relative positions between tokens, rather than capturing absolute position information for logical inference.

Before RoPE, there were several relative position encoding schemes, but most of them suffered from drawbacks such as high computational overhead or being unfriendly to KV Cache implementations. The ideal relative position encoding: it can be applied to each token in an absolute position manner, and when calculating attention (inner product), it naturally becomes a function of relative position. What mathematical operation can satisfy both of these conditions? The answer is the rotation of 2D planar vectors, as shown in the figure below:

![img/RoPE.png](img/RoPE.png)

You might wonder: a 2D vector can obviously be rotated this way, but how do we rotate a high-dimensional vector? It is hard to imagine how a 4D vector should be rotated to satisfy the conditions above. In reality, however, we do not need to actually rotate a high-dimensional vector directly. Because the ultimate goal is to compute the inner product, we can simply group the components of the high-dimensional vector in pairs of two and treat each pair as a 2D planar vector. The specific mathematical formulation is as follows, introducing two index parameters: \(i\in\{0, 1, \dots ,\text{max\_seq\_len}-1\}\) denotes the absolute position index of the token (handling up to \(\text{max\_seq\_len}\) tokens), and \(k \in \{0,1, \dots ,\frac{d}{2}-1\}\) denotes the group index after partitioning the \(d\)-dimensional token embedding vector in pairs of two.


$$
\theta_{i,k} = \frac{i}{\Theta^{\frac{2k}{d}}}
$$


$$
R_k^i = \begin{pmatrix} \cos(\theta_{i,k}) & -\sin(\theta_{i,k}) \\ \sin(\theta_{i,k}) & \cos(\theta_{i,k}) \end{pmatrix}
$$


$$
R^i = \begin{pmatrix} R_1^i & 0 & 0 & \dots & 0 \\ 0 & R_2^i & 0 & \dots & 0 \\ 0 & 0 & R_3^i & \dots & 0 \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & 0 & \dots & R_{d/2}^i \end{pmatrix}
$$

In practical code implementations, we generally do not directly store \(\text{max\_seq\_len}\) distinct \(R^i\) matrices. A better approach is to store the sine and cosine matrices separately; both are 2D dense matrices and can be reused globally. During computation, we transform the input and then directly apply element-wise multiplication. An intuitive example is rotating a 2D vector \([x_0, x_1]^\top\) by an angle \(\theta\):


$$
\begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix} \begin{pmatrix} x_0 \\ x_1 \end{pmatrix} = \begin{pmatrix} x_0 \cos\theta - x_1 \sin\theta \\ x_1 \cos\theta + x_0 \sin\theta \end{pmatrix}
$$

We can rewrite this in the form of element-wise multiplication: $\(x \odot \cos(\theta) + \tilde{x} \odot \sin(\theta)\)$

where \(\tilde{x} = [-x_1, x_0, -x_3, x_2, \dots]\), which swaps adjacent elements in pairs and applies a negative sign; this result can be obtained using batched matrix/tensor operations.

#### Hyperparameter Selection

Hyperparameter selection generally revolves around these core questions:

1. How many times the model dimension \(d_{\text{model}}\) should the FF layer dimension \(d_{\text{ff}}\) be set to?
2. How should the number of attention heads be chosen? Should \(d_{\text{model}} = d_{\text{head\_num}} \cdot d_{\text{head}}\) strictly hold?
3. How should the aspect ratio between model width and depth be determined?
4. How should the vocabulary size be chosen?
5. Is regularization necessary to prevent overfitting when the data volume far exceeds the parameter count?

First, the relationship \(d_{\text{ff}} = 4d_{\text{model}}\) holds true for the vast majority of current models, with very few exceptions. For example, in GLU-variant FF layers, due to the third projection matrix introduced, \(d_{\text{ff}}\) is set to \(\frac{2}{3}\) of the standard size, giving \(d_{\text{ff}} = \frac{8}{3}d_{\text{model}}\) in this family—which in a sense isn't even a true exception. The real outlier is Google's T 5 model, which uses an exaggerated \(d_{\text{ff}} = 64d_{\text{model}}\), though the paper explains this choice was driven by TPU utilization considerations.

As for the number of attention heads, \(d_{\text{model}} = d_{\text{head\_num}} \cdot d_{\text{head}}\) holds true almost all the time, with the main exceptions again being certain Google models.

> [!NOTE] Detailed Explanation
> You might wonder: if \(d_{\text{model}} \neq d_{\text{head\_num}} \cdot d_{\text{head}}\), wouldn't the token embedding dimension change? How could it then be fed into the next Transformer Block? To resolve this issue, we simply wrap the final output with an additional linear layer \(W_{O}\) to project the output dimension back to \(d_{\text{model}}\). Furthermore, another purpose of \(W_{O}\) is to mix the representations produced across different attention heads; therefore, a \(W_{O}\) layer is applied even when \(d_{\text{model}} = d_{\text{head\_num}} \cdot d_{\text{head}}\).

Regarding the aspect ratio (width vs. depth), extensive empirical data indicates that \(d_{\text{model}}/n_{\text{layer}} \approx 128\) is a sweet spot. Models are not strictly better the deeper they get; extremely deep models are notoriously hard to parallelize and suffer from high latency.

![aspect_ratio](img/aspect_ratio.png "Aspect Ratio Line Chart" )

Vocabulary size depends on whether the model is monolingual or multilingual: monolingual models typically use 30–50 k, while multilingual vocabularies reach 100–250 k. As a side note, Transformers with multimodal generation capabilities feature even larger vocabularies.

![vocabulary_size](img/vocabulary_size.png "Vocabulary Size Statistics" )

On overfitting and regularization: logically, overfitting shouldn't be a concern when data scale vastly exceeds parameter count, as overfitting rarely occurs under these conditions. SGD-style optimizers perform only a single pass over the corpus, making it hard for the model to memorize text snippets. Indeed, Dropout has largely been phased out. However, weight decay has been retained: although weight decay yields worse results in most cases compared to no decay, combining it with a small dynamic learning rate ultimately outperforms non-decayed setups (as seen in the bottom-right of the second figure below, where the blue dashed line achieves the optimal training loss).

![img/why_weight_decay.png](img/why_weight_decay.png)

#### Training Stability

What is training stability? It might sound a bit abstract, but the figure below illustrates it well: although the blue curve achieves lower loss, it frequently exhibits severe spikes—meaning you might end up with a terrible model when training completes. The blue curve is a classic sign of training instability.

![train_stability](img/train_stability.png "Training Stability Diagram" )

Where does this instability stem from? First is the Softmax function at the output layer, where the division operation can be dangerous:


$$
\log(P(x)) = \log\left(\frac{e^{U_r(x)}}{Z(x)}\right) = U_r(x) - \log(Z(x))
$$


$$
Z(x)=\sum_{r'=1}^{|V|} e^{U_{r'}(x)}
$$

where \(U_{r}\) represents the scoring function and \(|V|\) denotes the vocabulary size. To stabilize this term, \(\log(Z(x))\) can be transformed into the following form—squaring it and tuning it with \(\alpha\) so that the division term approaches 0 as closely as possible. This method is called Z-loss stability.


$$
\begin{aligned} L &=\sum_i \left[ \log(P(x_i)) -\alpha(\log(Z(x_i))-0)^2 \right] \\ &=\sum_i \left[ \log(P(x_i)) -\alpha\log^2(Z(x_i)) \right] \end{aligned}
$$

Second is the instability within the attention module's Softmax. However, the solution here isn't to modify Softmax directly, but to apply Normalization to the QK inputs—the widely known "QK Norm" method. In standard Transformer attention, Q and K undergo an inner product immediately after computation; "QK Norm", on the other hand, normalizes Q and K before computing their inner product, ensuring the values fed into Softmax remain on a far more consistent scale.

#### Attention Overhead During Inference

Most of our previous discussion focused on model training. However, when deploying models, we must consider inference-time overhead—most notably, the attention computational cost. Let us define an attention module where \(d\) is the model hidden dimension, \(b\) is the batch size, \(n\) is the input sequence length, \(h\) is the number of attention heads, and \(k=\frac{d}{h}\) is the head dimension.

Let us first analyze the arithmetic operations of the attention module during training. The input \(X\) has shape \(b\times n\times d\), and each head projection matrix has shape \(d\times k\), so the QKV projection FLOPs equal \(3\times(b\cdot n\cdot d\cdot k)\times h = 3bnd^2\). Next, computing the inner product of Q and K (both of shape \(b\times n\times k\)) takes \(bhkn^2 = bdn^2\) operations. While there are subsequent computations, they are non-dominant for large \(d\) and \(n\), so we can simply represent the computational complexity of the attention module as \(O(bnd^2+bdn^2)\).

Memory cache usage during training is straightforward to compute: first, matrices like input and QKV of shape \(b\times n \times d\) require \(O(bnd)\) memory access; then Softmax requires \(O(bhn^2)\) cache access, and weight parameters require \(d^2\) cache access, leading to a total memory access complexity of \(O(bnd+bhn^2+d^2)\).

Thus, the arithmetic intensity of the training attention module is \(O\left( \left( \frac{1}{d+n}+\frac{hn}{d^2+dn}+\frac{d}{bnd+bn^2} \right)^{-1} \right)\), which clearly becomes increasingly compute-dense as sequence length \(n\) grows.

Inference, however, is fundamentally different because tokens must be generated sequentially. In inference, we introduce KV Cache to avoid redundant computations, keeping computational complexity at \(O(bnd^2+bdn^2)\), but driving memory access complexity up to \(O(bn^2d+d^2)\).

The arithmetic intensity converges toward 1 as sequence length \(n\) grows—an extremely low intensity considering a fully loaded A 100 has an arithmetic intensity of around 156. In other words, LLM inference is a classic memory-bound task.

To reduce memory bandwidth intensity, we can employ Grouped-Query Attention (GQA), which groups query heads together so that heads in the same group share a single set of KV Cache. Denoting the number of groups as \(g\), GQA reduces memory access complexity to \(O\left( \frac{1}{g}bn^2d+d^2 \right)\).

Another technique for reducing attention computational complexity is grouped sliding-window attention. Simply put, full attention across the entire sequence is computed only once every four attention layers, while intermediate layers only compute attention over tokens within a fixed sliding window.

#### Attention Mechanism

There are two main optimization approaches for attention mechanisms: keeping the \(O(n^2)\) complexity while optimizing constants, or fundamentally redesigning the mechanism to achieve linear \(O(n)\) complexity. The former works remarkably well when context lengths aren't exceptionally long (say, under 2 M tokens)—take ChatGPT's local sliding-window attention as an example. Alternatively, you can push constant-factor optimization to the extreme through hardcore systems engineering; FlashAttention is a prime example here, reaching roughly 4 x the speed of standard PyTorch implementations.

However, if we want to process even longer input sequences, we have to tackle the \(O(n^2)\) complexity directly and reduce it to \(O(n)\). The core idea stems right from the standard quadratic attention formula:


$$
\text{Attn(Q,K,V)} = \text{softmax}(QK^T)V
$$

If the softmax function could be omitted, attention would naturally become linear. Below is the batched expression, which is convenient for training:


$$
\text{Attn(Q,K,V)} = (QK^T)V = Q(K^TV)
$$

We can also express it in a recurrent form for inference, which looks strikingly similar to an RNN:


$$
\begin{matrix} S_{t} = S_{t-1} + k_{t}v_{t}^T \\ y_{t} = q_{t}^TS_{t} \end{matrix}
$$

Linear attention layers are indeed deployed in real-world models. For instance, MiniMax-M 1 adopts a hybrid setup with 7 linear attention layers paired with 1 quadratic attention layer, balancing the recall issues of pure linear attention against the heavy compute costs of pure quadratic attention. To give you a clearer picture of the recall loss brought by linear attention, here is a figure from ByteDance's survey paper on hybrid linear attention:

![linear_attention_performance_loss](img/linear_attention_performance_loss.png "Performance loss brought by linear attention mechanisms" )

As seen in the right chart, pure linear attention suffers a steep decline in recall. Curiously, the left chart shows that some pure linear attention models actually outperform standard quadratic attention in language capabilities (note that the model evaluation score here is a composite metric of perplexity and benchmarks). This suggests that pure recall tasks aren't fully aligned with real-world language tasks, where exact recall isn't always strictly required. Another practical takeaway from the right chart is that a 3:1 hybrid ratio achieves recall performance on par with standard attention—which explains why this 3:1 hybrid scheme is so widely used in practice.

Building on the recurrent formulation of linear attention, we can add a memory decay factor \(\gamma_{t} = f(x_{t})\) to clear out stale memory states, along with a gated output term \(v_{t}^TD\) (allowing certain information to bypass state compression and output directly). This gives us the Mamba-2 formulation. The Mamba family is undoubtedly one of the most famous linear attention designs out there; in Nemotron-3, it is combined with standard attention in a 3:1 ratio, delivering superior performance compared to pure linear setups.


$$
\begin{matrix} S_{t} = \gamma_{t} S_{t-1} + k_{t}v_{t}^T \\ y_{t} = q_{t}^TS_{t} + v_{t}^TD \end{matrix}
$$

Applying a directional Delta correction to the Mamba-2 expression, along with an input gate \(\beta_{t} = f(x_{t})\) for new memory entries, yields the Gated DeltaNet formulation:


$$
\begin{matrix} S_{t} = \gamma_{t} (I-\beta_{t}k_{t}k_{t}^T) S_{t-1} + \beta_{t}k_{t}v_{t}^T \\ y_{t} = q_{t}^TS_{t} \end{matrix}
$$

Here, \((I-\beta_{t}k_{t}k_{t}^T)\) erases parts of the state related to the current \(k_{t}\), making room for the new memory item \(\beta_{t}k_{t}v_{t}^T\). The largest-scale application of this approach to date is the Qwen-3.5 model—though again, it uses a 3:1 hybrid format rather than pure linear attention to get the best of both worlds. A clear trend emerges: linear attention mechanisms are gradually converging toward LSTM-like architectures.

Besides hidden-state approaches, another route is sparse activation, best exemplified by DSA (DeepSeek Attention). DSA uses a lightweight index to compute and retrieve the Top-k most relevant tokens, keeping the costs of \(O(n^2)\) attention manageable. This mechanism is adopted in both DeepSeek-v 3.2 and GLM-5, yielding impressive real-world results.

> [!NOTE]- A Side Note
> While benchmark scores look great on paper, in practical use you'll notice severe context-constraint forgetting. Personally, I don't feel fully confident delegating highly complex tasks to DeepSeek models—if you audit their outputs closely, you'll find they often fail to follow constraints mentioned earlier in the prompt. In short, it can feel a bit clunky 🤔, especially when compared side-by-side with Anthropic's models.

#### MoE

Mixture of Experts (MoE) is currently one of the most mainstream architectural paradigms. It allows scaled-up models to run with manageable compute costs, reaping the performance gains of parameter expansion while keeping inference costs in check—a true win-win. Of course, MoE wasn't an instant hit; its architecture and training balance are notoriously tricky to tune. In a sense, MoE represents less of a pure algorithmic leap and more of a major win for LLM systems engineering.

MoE can be applied to either Feed-Forward (FF) layers or Attention layers, but virtually all implementations focus on FF layers. By default, "MoE" refers to FF-layer MoE, a convention we'll follow here.

MoE consists of a few key components: routing functions, expert count, and training objectives. Routing functions and expert counts are straightforward, but training objectives serve a dual purpose: training the language model itself and maintaining routing balance so every expert receives adequate training.

Common routing functions include Top-k and Hash routing, alongside RL-based routing policies that frame load balancing as a linear assignment problem. Among these, Top-k is by far the most popular. Its mathematical formulation is as follows:


$$
h_t^l = \sum_{i=1}^{N} \left( g_{i,t}\operatorname{FFN}_i(u_t^l) \right) + u_t^l
$$


$$
g_{i,t} = \begin{cases} s_{i,t}, & s_{i,t}\in \operatorname{TopK} \left( \{s_{j,t}\mid 1\le j\le N\}, K \right),\\ 0, & \text{otherwise}, \end{cases}
$$


$$
s_{i,t} = \operatorname{Softmax}_i \left( (u_t^l)^T e_i^l \right)
$$

While this wall of math might look daunting, plain English puts it simply: the input \(u_{t}^l\) takes the inner product with each expert's key vector \(e_{i}^l\), followed by softmax normalization to obtain a matching probability distribution. Next, the gating function \(g_{i,t}\) selects the Top-k experts and zeroes out the rest (a batching trick that avoids conditional branching when calculating \(h_{t}^l\)). From there, standard feedforward computation takes over, aggregating expert outputs via simple summation before adding the residual connection.

Taking things a step further, experts can be split into finer sub-experts alongside dedicated shared parameters. This allows non-shared experts to specialize further, achieving better ensemble effects. This design is widely known as the DeepSeekMoE architecture.

![DeepSeek_MoE](img/DeepSeek_MoE.png "DeepSeek_MoE Architecture Diagram" )

Logically, both shared and non-shared experts should contribute to model performance. However, ablation studies from OLMoE reveal that performance gains actually stem from finer partitioning of non-shared experts, rather than the shared experts themselves.

![OrlMoE_shared_experts_ablation](img/OrlMoE_shared_experts_ablation.png "OLMoE Ablation Study on Shared Experts" )

The exact expert division ratios across architectures are detailed in the table below, which also highlights that DeepSeek-v 1 was indeed the pioneer in granular expert re-segmentation.

Model | Routed | Active | Shared | Fine-grained ratio
:----- | :-----: | :-----: | :-----: | :-----:
GShard | 2048 | 2 | 0 | -
Switch Transformer | 64 | 1 | 0 | -
ST-MOE | 64 | 2 | 0 | -
Mixtral | 8 | 2 | 0 | -
DBRX | 16 | 4 | 0 | -
Grok | 8 | 2 | 0 | -
DeepSeek v 1 | 64 | 6 | 2 | 1/4
Qwen 1.5 | 60 | 4 | 4 | 1/8
DeepSeek v 3 | 256 | 8 | 1 | 1/14
OLMoE | 64 | 8 | 0 | 1/8
MiniMax | 32 | 2 | 0 | ~1/4
Llama 4 (maverick) | 128 | 1 | 1 | 1/2

Finally, let's look at MoE training. The primary obstacle in MoE training is that the routing function is non-differentiable and inherently prone to instability. Early attempts tried modeling routing as an RL task with little success; adding noise to Top-k sampling improved stability, but at a slight cost to performance. Currently, the most effective solution is a regularization-like approach that introduces a heuristic load-balancing auxiliary loss:


$$
\text{loss} = \alpha \cdot N \cdot \sum_{i=1}^{N} f_i \cdot P_i
$$


$$
f_i = \frac{1}{T} \sum_{x \in \mathcal{B}} \mathbb{1}\{\operatorname{argmax} p(x) = i\}
$$


$$
P_i = \frac{1}{T} \sum_{x \in \mathcal{B}} p_i(x)
$$

Here, \(f_{i}\) denotes the actual proportion of tokens assigned to expert \(e_{i}\), while \(P_{i}\) represents the routing function's target probability for \(e_{i}\). This formulation uses the non-differentiable \(f_{i}\) to guide the differentiable \(P_{i}\), applying gradient penalties to unbalanced allocations:


$$
\frac{\partial \text{loss}}{\partial p_i(x)} = \frac{\alpha N}{T^2} \sum_{x \in \mathcal{B}} \mathbb{1}_{\operatorname{argmax} p(x) = i}
$$

Empirical results confirm that this heuristic approach works remarkably well, effectively balancing load across experts. Standard DeepSeekMoE further supplements this with device-level load-balancing regularization.

![hurestic_load_balancing_ablation](img/hurestic_load_balancing_ablation.png "Ablation study on heuristic load balancing" )

The later DeepSeekMoE-v 2 built upon this by introducing a Top-M device routing mechanism and inter-device communication penalty terms to curb communication overhead. To be honest, this adds quite a lot of regularization terms—it's effective, but arguably ungraceful. DeepSeekMoE-v 3 attempted to remove these loss terms by introducing dynamic bias terms, though it didn't eliminate them entirely.

Of course, training stability and fine-tuning in MoE differ markedly from standard dense FF modules—and frankly, are much trickier. Training instability largely stems from the softmax function in routing (nine out of ten instability issues trace back to softmax 🤣), which can be mitigated using the Z-loss method covered in the [Training Stability]({{< relref "#training-stability" >}}) section. On the fine-tuning front, MoE models are far more prone to overfitting on small datasets than dense counterparts. DeepSeek's remedy? Fine-tune with massive datasets.

Overall, MoE carries a heavy systems engineering flavor—which brings us back to my point at the start: MoE's true impact shines brightest in systems engineering. At its core, MoE uses pragmatic engineering tricks to push the boundaries of scaling laws, giving us a straightforward path to continue squeezing performance out of sheer model scale.

## Systems engineering

The core of this section is grasping the fundamental mechanics of modern GPUs and leveraging these insights to guide model architecture choices and algorithmic breakthroughs. On the architecture front, a prime example is the NanoGPT Speedrun project, where the single most dramatic speedup came from simply padding the model's vocabulary size up to the nearest multiple of 64.

![img/nanogpt-speedrun.png](img/nanogpt-speedrun.png)

On the algorithmic side, FlashAttention is the undisputed poster child—its performance gains are measured in integer multiples rather than mere percentages.

### GPU Architecture

A GPU’s internal architecture and scheduling mechanics can be neatly unpacked through two orthogonal lenses: the "Programming & Memory Perspective" and the "Hardware & Scheduling Perspective." The programming and memory view breaks down as follows:

![img/gpu_memory_model.png](img/gpu_memory_model.png)

1. Thread: The fundamental unit of parallel execution; all threads run identical instructions across different slices of input data.
2. Block: A cluster of threads designed to collaborate and communicate through fast shared memory.
3. Grid: A collection of blocks coordinated via shared global memory and static memory.
4. Host: The CPU host and its system memory. Data moves between host memory and GPU VRAM across PCIe; because of bandwidth and latency penalties, cross-host communication and data transfers are notoriously expensive.

From the hardware and scheduling perspective, the following concepts take center stage:

1. SM (Streaming Multiprocessor): The GPU's primary physical compute engine, equipped with its own warp schedulers, register files, shared memory, and execution pipelines to host and execute thread blocks.
2. Warp: The lowest-level physical execution unit on hardware, strictly fixed at 32 threads. The distinction from a block is crucial: blocks are logical abstractions defined by developers with flexible sizes, but the hardware slices them into warps where all 32 threads execute scheduler instructions in lockstep; the primary purpose of warps is latency hiding via zero-overhead context switching (e.g., when Warp 0 stalls on memory reads, the scheduler immediately fires compute instructions from Warp 1).
3. Wave: The macro-level scheduling batch across the entire device, representing the maximum number of blocks that all SMs on the chip can concurrently execute; the GPU dispatches and completes work strictly in complete wave increments.

Intuitively, GPU architecture mirrors classic CPU paradigms, so anyone with computer architecture foundations will feel right at home with the multi-tier cache hierarchy. Similarly, TPU and GPU designs share substantial DNA, with direct conceptual equivalents across both camps. The real divergence lies in TPUs featuring leaner control units, massive matrix multiplication units (MXUs), and ultra-fast L 1 scratchpads (Shared Memory in GPUs vs. Vector Memory in TPUs).

![img/tpu-gpu.png](img/tpu-gpu.png)

Across GPU generations, different subsystems evolve at vastly disparate rates: arithmetic compute capability has scaled exponentially faster than cache and memory bandwidth, as charted below. This is precisely why modern systems engineering obsesses over memory hierarchy optimizations—compute is so far ahead of memory loading that eliminating a single memory round-trip can unlock a 10,000 x (or even higher) boost in operational throughput.

![img/gpu_component_develop_speed.png](img/gpu_component_develop_speed.png)

### Optimization Techniques

#### Branch Control

In short: replace branching (if-else) instructions with masks. This necessity stems from the core SIMT constraint: every thread within a warp must follow identical instructions. When conditional paths diverge on different data, conflicting execution requirements emerge. The hardware’s brute-force resolution is straightforward: both branches get executed sequentially, and unselected results are masked and thrown away.

![img/gpu_diverge_excute.png](img/gpu_diverge_excute.png)

#### Low-Precision Computing

The premise is straightforward: trade numeric precision for massive data throughput. Pushing training to sub-byte levels, however, requires clever tricks. The prevailing industry standard groups low-precision values under a shared scaling factor, best exemplified by the MXFP 8 format. Yet this introduces a notorious headache: how do you handle transpositions? Because values are blocked and scaled row-wise, transposing swaps rows to columns, shattering the original scale-factor layout. There is no truly elegant in-place workaround, so training pipelines typically just duplicate and maintain a dedicated transposed copy of the tensor in memory.

![img/MXFP8.png](img/MXFP8.png)

Another promising angle omitted in Stanford's CS 336—likely due to its complexity and lack of battle-testing in production-grade frontier models—is detailed in this [paper](https://arxiv.org/abs/2509.00404), which achieved end-to-end FP 4 training on an 8 B LLM. Prior to this, NVIDIA possessed internal FP 4 training recipes but kept the blueprints proprietary. The authors reverse-engineered NVIDIA’s technical blog posts by simulating FP 4 training mechanics via BF 16, meticulously matching target precision, training loss curves, and convergence dynamics in an impressively rigorous baseline study.

At its core, this research tackles a fundamental dilemma: neural network weights exhibit wide dynamic ranges, meaning naive scaling-factor quantization squashes subtle values under large outliers (classic quantization error), as an inflated dynamic range widens the quantization grid steps and collapses tiny near-zero values straight to zero. Intuitively, dominant outliers and subtle residuals demand decoupled representations. The authors achieve this by decomposing weight matrices via SVD, where \(\sigma_{i}\) denotes descending singular values:


$$
M = \sum_{i=1}^{r} \sigma_i \cdot (u_i v_i^T)
$$

Experimental results reveal that roughly the top 3% of singular values dominate the upper bound of the magnitude distribution (spanning one to two orders of magnitude). Stripping away these few dominant singular values leaves a residual matrix with an exceptionally flat, uniform distribution ready for tight quantization grids, while the isolated dominant components naturally compress into a shared scale factor coupled with well-behaved singular vectors—making it uniquely tailor-made for FP 4 quantization.

![img/Pasted image 20260903012451.png](img/Pasted image 20260903012451.png)

It’s worth highlighting that SVD-driven quantization wasn't invented in a vacuum; an earlier precursor was [SVDQuant](https://arxiv.org/abs/2411.05007). While both leverage SVD, SVDQuant feels somewhat unpolished in practice, shuttling high-precision matrices back and forth and wrestling with sparse matrix multiplication overheads, whereas the newer paper cleanly absorbs the heavy lifting directly into scaling factors and singular vectors.

#### Operator Fusion

Fusing multiple kernel operations into a unified pass avoids repeatedly ping-ponging intermediate tensors to and from HBM, slashing memory access overhead—this serves as the absolute cornerstone of FlashAttention. The diagram below captures the rationale intuitively: load data once, churn through as many arithmetic operations as possible locally, and avoid premature write-backs.

![img/operator_fusion.png](img/operator_fusion.png)

#### Recomputation

Literally what it says on the tin: discard intermediate activations instead of parking them in cache, and recompute them on the fly from primary inputs during the backward pass. In the toy workflow illustrated below, this clever trade-off slashes 8 memory accesses down to just 5.

![img/recomputation.png](img/recomputation.png)

#### Memory Coalescing

This optimization taps into raw silicon mechanics: global memory (DRAM) accesses are wide and concurrent, meaning memory controllers must activate and pull an entire DRAM row buffer into staging before selecting target bytes. In other words, you pay the full latency and power cost of fetching a wide memory segment regardless; your only leverage is coalescing concurrent thread requests to consume as much of that pre-fetched segment as humanly possible.

![img/memory_coalescing.png](img/memory_coalescing.png)

For instance, accessing a row-major matrix with threads distributed across different row indices produces fragmented, strided fetches; aligning threads across contiguous column elements along the same row index allows a single coalesced DRAM transaction to satisfy all threads simultaneously.

#### Tiling

Sharing the identical philosophy behind operator fusion, tiling aims to minimize redundant memory round-trips by maximizing arithmetic work per byte loaded into high-speed memory.

![img/tiling.png](img/tiling.png)

As depicted above, without tiling, every input element must be fetched from slow global memory \(N\) times; under a tiled scheme, global reads drop to \(N/T\) per element, supplemented by \(T\) subsequent reads from ultra-fast on-chip Shared Memory. Since shared memory operates at near-register speeds, this effectively reduces DRAM pressure by roughly \(\frac{1}{T}\) —an immense architectural win.

Crucially, the efficacy of tiling hinges dramatically on matrix geometries—poorly conditioned shapes incur catastrophic efficiency penalties. The first pitfall is bad tiling: edge tiles containing barely any real data end up spinning SM execution pipelines entirely in idle waste, as illustrated below.

![bad_tiling](bad_tiling.png ""Cite from [NVIDIA Blog](https://docs.nvidia.com/deeplearning/performance/dl-performance-matrix-multiplication/index.html#tile-quant))

The second pitfall is memory alignment: well-aligned dimensions allow tiled fetches to fully exploit DRAM concurrent burst transfers, whereas mismatched alignments trigger memory fragmentation that wrecks theoretical throughput.

![layout_align](img/layout_align.png "Cite from the [blog](https://www.thonking.ai/p/what-shapes-do-matrix-multiplications)" )

#### Performance Benchmark Teardown

![flops_for_square_matmul](img/flops_for_square_matmul.webp "Cite from the [blog](https://www.thonking.ai/p/what-shapes-do-matrix-multiplications)" )

Armed with these principles, the bizarre cliffs and jagged steps in this benchmark plot suddenly make crystal-clear sense: the macro trend reflects arithmetic intensity scaling with matrix dimension; the abrupt performance cliffs highlight pathological shape misalignment (as broken down in the [Tiling]({{< relref "#tiling" >}}) section), where clean alignment alone can yield a nearly 2 x throughput boost; and finally, the mysterious periodic sawtooth dips stem from GPU wave quantization—when matrix dimensions cross specific boundaries, an extra wave is spawned with barely enough thread blocks to fill it, forcing SMs to run a nearly empty tail wave at full cost.

![img/wave_quantization_example.png](img/wave_quantization_example.png)
