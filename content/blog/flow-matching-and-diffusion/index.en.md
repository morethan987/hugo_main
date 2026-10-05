---
title: Flow matching and diffusion model
weight: -130
draft: true
description: Some notes for flow matching and diffusion model
slug: flow-matching-and-diffusion
language: en
tags:
  - Architecture
  - Notes
  - GenerativeModel
series:
  - AI Engineering
series_order: 2
date: 2026-10-02
lastmod: 2026-10-05
authors:
  - Morethan
---
{{< katex >}}

Denoising diffusion and flow matching models are the backbone of the best image, audio and video generation models. So being familiar with the relevant technics and build some intuition for that is both significant and interesting.

This article is mainly a note taking for a fantastic course [MIT 6.S184](https://diffusion.csail.mit.edu/2026/index.html).

## Overview

Firstly, we want to generate something which could be an image or an vedio. That's the original purpose of our work. But actually what does "something" mean and how can we represent it formally?

Actually, all the object we interest can be represented as a vector \(z \in \mathbb{R}^d\) or can be flatten to a vector. Instead of saying I wanna an image, now we can say that I wanna the vector of image. So here is the key idea:

> **An object is a vector.**


The next informal word is "generation". Intuitively, there should be a lot of figures in your mind when you thinking "a picture of dog" and you cannot tell which one is correct but which one is better. Noticing the similarity of rolling an instance from a data distribution, we can tell that "generation" is the same with "sampling". So the key idea here is:

> **Generation can be viewed as sampling.**


And now the target is much more clear: we need to find the distribution of the vector we need. Taking a step more, we may need to "switch" the distribution when we want something different such as swtiching from the "dog distribution" to the "cat distribution".

*So the final formal target is to sampling a vector \(z\) from the conditional distribution \(p_{\text{data}(\dot{}|y)}\) where \(y\) is a prompt describes what we want.*

## Motivation

How to represent a distribution if we just want to generate the images of dog? This is a weaken version of our final target and simply we can apply a neural network to be the distribution. It's just like a bandit that takes stochastic seeds as input and simply generate pictures of dog.

This idea is quite reasonable and is exactly the things what we do in the past decade. The key challenge is how to bind the random seeds with the actual dog images: here is an image of husky and what random seed of it? If you ignore this and directly train a model with MSE loss you will get an "averaged" dog. That may be something you can hardly tell it is a dog. 🤣

To solve this problem, we can redefine the challenge since it's not neccessary to tell which seed is "husky seed" and what we actually need is to train our distribution (model) to be **as near as possible** to the "dog distribution".

This naturally shifts our goal from comparing individual pixels to measuring the "distance" between two probability distributions. Once we stop obsessing over exact pairings and instead use clever setups, like an adversarial critic (GANs) or multi-step denoising (Diffusion), to pull the generated distribution towards the real one as a whole, that blurry average ghost disappears, and realistic, sharp dogs finally pop right out.

But here comes the catch: even if we drop pixel-to-pixel MSE and turn to distribution-matching games (like GANs), asking a network to morph a simple Gaussian blob into a wildly complex, disconnected image manifold in a single leap is **mathematically brutal**. The required mapping is horribly discontinuous and ill-conditioned. This single-step leap is precisely why older generative models suffered from notorious instability, mode collapse, or severe architectural restrictions.

This brings us to the grand paradigm shift in MIT 6.S 184, **Flows and Diffusion**:

> **Instead of taking an impossible giant leap, why not build a continuous bridge?**


If jumping directly from pure noise \(z \sim \mathcal{N}(0, I)\) to a crisp husky \(x \sim p_{\text{data}}\) in one step is too hard, we can break the journey down into smooth, incremental steps along a time trajectory \(t \in [0, 1]\). At any point along the path, the network no longer needs to predict the entire dog out of thin air; it only needs to tell us **which local direction to nudge the sample**—whether that means predicting a local velocity vector (Flow Matching) or removing a tiny speck of noise (Diffusion). By turning a chaotic global mapping into a sequence of easy, locally stable regressions, we finally get stable training and astonishing sample quality.

> [!NOTE]+ Note
> This also brings us some bonus. With flow or diffision progress, your prompt can be reseaonablly injected. You can imagine how complex the mapping function will be when attempting to get the final distribution in one step.

From a general perspective, this method can be view as getting "depth" in the time dimension instead of in the model dimension.

## Algorithm

### Flow model

To summarize, our current task is to describe the evolution of how a simple noise distribution gradually turns into the target data distribution.

Fortunately 🐸, Ordinary Differential Equations (ODEs) provide an elegant framework to describe this transformation. Specifically, if we can find a time-dependent vector field \(u_t(x)\) that generates the desired probability density path \(p_t(x)\) from \(t = 0\) to \(t = 1\), we can train a neural network \(u_t^\theta(x)\) with weights \(\theta\) to approximate it.

Naturally, the regression objective, the **Flow Matching loss**, is formulated as:


$$
\mathcal{L}_{\text{FM}}(\theta) = \mathbb{E}_{t \sim \text{Unif}[0, 1], \, x \sim p_t} \left[ \| u_t^\theta(x) - u_t(x) \|^2 \right]
$$

However, a fundamental dilemma arises immediately:  \(u_t(x)\) is completely intractable in practice. We only have access to empirical samples from \(p_{\text{data}}\), so how can we determine the marginal vector field \(u_t(x)\) that moves the entire distribution?

To understand how \(u_t(x)\) is defined and how we can compute it, let us formalize the dynamics from single particles to probability distributions.

A particle's trajectory \(X: [0, 1] \to \mathbb{R}^d, \, t \mapsto X_t\) is governed by a time-dependent vector field \(u_t: \mathbb{R}^d \to \mathbb{R}^d\) specifying the velocity at each point and moment in time:


$$
\frac{d}{dt} X_t = u_t(X_t), \quad X_0 = x_0
$$

The continuous solution across time is characterized by the flow map \(\psi_t: \mathbb{R}^d \to \mathbb{R}^d\), which tracks the position \(\psi_t(x_0) = X_t\), satisfying:


$$
\frac{d}{dt}\psi_t(x_0) = u_t(\psi_t(x_0)), \quad \text{with } \psi_0(x_0) = x_0
$$

Physically, \(u_t\) is simply a function that takes a time \(t\) and a coordinate \(x\), and returns the instantaneous velocity vector of a particle at that location.

Things become interesting when we consider a whole ensemble of particles forming a continuous **probability distribution** \(p_t(x)\) over time (termed a **probability density path**). 

Because probability mass is conserved (particles cannot be created or destroyed), the spatial density \(p_t(x)\) carried along by the vector field \(u_t(x)\) must satisfy the **continuity equation**:


$$
\frac{\partial p_t(x)}{\partial t} = - \nabla \cdot \big( p_t(x) u_t(x) \big)
$$

Since the marginal distribution \(p_t(x)\) and vector field \(u_t(x)\) are difficult to work with globally, we introduce a conditioning variable \(z\) (e.g., target data points \(x_1 \sim p_{\text{data}}\) or pairs \((x_0, x_1)\)):


$$
p_t(x) = \int p_t(x \mid z) p(z) \, dz
$$

Here, each conditional path \(p_t(x \mid z)\) has its own simple, well-defined conditional vector field \(u_t(x \mid z)\) satisfying its own continuity equation:


$$
\frac{\partial p_t(x \mid z)}{\partial t} = - \nabla \cdot \big( p_t(x \mid z) u_t(x \mid z) \big)
$$

Taking the time derivative of the marginal distribution \(p_t(x)\):


$$
\begin{aligned} \frac{\partial p_t(x)}{\partial t} &= \int \frac{\partial p_t(x \mid z)}{\partial t} p(z) \, dz \\ &= - \nabla \cdot \int p_t(x \mid z) u_t(x \mid z) p(z) \, dz \\ &= - \nabla \cdot \left( p_t(x) \int u_t(x \mid z) \frac{p_t(x \mid z) p(z)}{p_t(x)} \, dz \right) \\ &= - \nabla \cdot \left( p_t(x) \int u_t(x \mid z) p_t(z \mid x) \, dz \right) \end{aligned}
$$

Comparing this with the original marginal continuity equation \(\frac{\partial p_t(x)}{\partial t} = - \nabla \cdot \big( p_t(x) u_t(x) \big)\), we can define the marginal vector field simply as:


$$
u_t(x) = \int u_t(x \mid z) p_t(z \mid x) \, dz = \mathbb{E}_{z \sim p_t(z \mid x)} \left[ u_t(x \mid z) \right]
$$

This crucial result shows that the complex global vector field \(u_t(x)\) is simply the aggregate expectation of tractable, conditional vector fields \(u_t(x \mid z)\).

Recall our original Flow Matching objective:


$$
\mathcal{L}_{\text{FM}}(\theta) = \mathbb{E}_{t \sim \text{Unif}[0, 1], \, x \sim p_t} \left[ \| u_t^\theta(x) - u_t(x) \|^2 \right]
$$

Substituting the marginal field \(u_t(x) = \mathbb{E}_{z \sim p_t(z \mid x)} [u_t(x \mid z)]\) directly into this loss and expanding the square reveals a remarkable fact: \(\mathcal{L}_{\text{FM}}(\theta)\) differs from the Conditional Flow Matching objective by only a constant term independent of \(\theta\):


$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathbb{E}_{t \sim \text{Unif}[0, 1], \, z \sim p(z), \, x \sim p_t(x \mid z)} \left[ \| u_t^\theta(x) - u_t(x \mid z) \|^2 \right]
$$

Specifically, one can show that:


$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathcal{L}_{\text{FM}}(\theta) + C \quad \implies \quad \nabla_\theta \mathcal{L}_{\text{FM}}(\theta) = \nabla_\theta \mathcal{L}_{\text{CFM}}(\theta)
$$

The proof of this can be find in [lecture_notes](https://diffusion.csail.mit.edu/2026/docs/lecture_notes.pdf) and the key preliminary is \(u_t(x)\) can be represented as the aggregate expectation of conditional vector fields which has been proved before.

Since their gradients with respect to \(\theta\) are identical, minimizing the tractable conditional objective \(\mathcal{L}_{\text{CFM}}(\theta)\) is mathematically equivalent to minimizing the intractable marginal objective \(\mathcal{L}_{\text{FM}}(\theta)\). All we need in practice is to sample \(z\), sample \(x \sim p_t(x \mid z)\) which is a simpile evolution we can freely define, compute the closed-form conditional velocity \(u_t(x \mid z)\), and train our network via simple mean squared error regression.

### Diffusion model

The mathematics of score matching is actually completely self-contained right from the start. We define the marginal score as the gradient of the log-density, \(\nabla \log p_t(x)\). Even though the true marginal density \(p_t(x)\) is intractable, we have the elegant marginalization identity:


$$
\nabla \log p_t(x) = \int \nabla \log p_t(x|z) \frac{p_t(x|z)p_{\text{data}}(z)}{p_t(x)} \mathrm{d}z
$$

By expanding the mean squared error against this marginal score, the intractable marginal objective reduces directly to the conditional denoising score matching loss: \(\mathbb{E}[\|s_t^\theta(x) - \nabla \log p_t(x|z)\|^2]\). In theory, this single identity is all we need: score matching can be formulated, trained, and completed entirely on its own without knowing anything about vector fields or flow models.

However, training a score model is only half the story, because having the score function alone is not enough to generate new samples. A score function merely indicates the local direction of increasing probability density; it does not inherently transport probability mass forward from the initial noise \(p_{\text{init}}\) to the data distribution \(p_{\text{data}}\). If we want to generate samples by simulating a SDE while ensuring that the marginal distribution of \(X_t\) strictly follows the desired probability path \(p_t\), the Fokker-Planck equation dictates that the trajectory must follow:


$$
\begin{aligned} \mathrm{d}X_t &= u_t(X_t) \mathrm{d}t + \frac{\sigma_t^2}{2} \nabla \log p_t(X_t) \mathrm{d}t + \sigma_t \mathrm{d}W_t \\ &= \left[ u_t(X_t) + \frac{\sigma_t^2}{2} \nabla \log p_t(X_t) \right] \mathrm{d}t + \sigma_t \mathrm{d}W_t \end{aligned}
$$

Here, the Brownian motion \(\sigma_t \mathrm{d}W_t\) injects stochastic kicks, while the term \(\frac{\sigma_t^2}{2} \nabla \log p_t(X_t)\) serves as an inward correction to counteract the dispersion caused by the noise. But look closely at the drift: it explicitly requires *both* the deterministic velocity field \(u_t(X_t)\) and the score function \(\nabla \log p_t(X_t)\). Score alone only gives the corrective pull, not the baseline forward trajectory.

This leads to an awkward reality: for an arbitrary probability path, the velocity field \(u_t\) and the score field \(\nabla \log p_t\) are fundamentally distinct mathematical objects with no simple relation. If we chose an arbitrary distribution, we would be trapped in an engineering nightmare where we must train *two separate neural networks*, one via flow matching to learn \(u_t\), and another via score matching to learn \(\nabla \log p_t\), just to run a single SDE sampling loop.

This awkward dilemma is precisely why practical diffusion models almost universally settle on Gaussian probability paths, \(p_t(x|z) = \mathcal{N}(x; \alpha_t z, \beta_t^2 I_d)\). Under a Gaussian path, both the conditional velocity field \(u_t(x|z)\) and the conditional score \(\nabla \log p_t(x|z)\) happen to be affine functions of \(x\) and \(z\). Once integrated against the posterior, both collapse into linear reparameterizations of the exact same posterior mean \(\mathbb{E}[z \mid x]\) (the denoiser). This yields the closed-form bridge:


$$
u_t(x) = a_t \nabla \log p_t(x) + b_t x
$$

Because of this exact equivalence, training a single network gives us the other quantity entirely for free.

> [!NOTE] Thought
> Why we introduce some stochastic factor? A converged explaination is about **diversity**. Flow model is simple and graceful but too rigrid and determinated. Moreover, there are some errors for flow model training which cannot be tackled theoritically. By introducing the stochastic factor, we hope to achieve a better performance by adjusting the \(\sigma\) of SDE.
