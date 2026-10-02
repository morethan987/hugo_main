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
lastmod: 2026-10-02
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

## Flow and Diffusion Models

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

### Flow model

To sumarize, our current task is to describe the evolution progress of how the noise distribution gradually turns into the expected data distribution.

With fortune, I'd like to say 🐸, we find the ordinary differential equations (ODEs) to be a suitable tool to describe the progress. And here I just reference the content from the [lecture_notes](https://diffusion.csail.mit.edu/2026/docs/lecture_notes.pdf) to keep the completeness of this blog.

Mathematically, a trajectory \(X: [0, 1] \to \mathbb{R}^d, \, t \mapsto X_t\) is driven by a time-dependent vector field \(u_t: \mathbb{R}^d \to \mathbb{R}^d\) specifying the velocity at each point and moment in time:


$$
\frac{d}{dt} X_t = u_t(X_t), \quad X_0 = x_0
$$

The solution to this equation across time is characterized by the flow \(\psi_t: \mathbb{R}^d \to \mathbb{R}^d\), which tracks the position \(\psi_t(x_0) = X_t\) satisfying \(\frac{d}{dt}\psi_t(x_0) = u_t(\psi_t(x_0))\) with \(\psi_0(x_0) = x_0\). 

The clever part of building a generative flow model is that the neural network is never asked to predict the entire trajectory or the flow \(\psi_t\) directly. Instead, we only parameterize the local vector field \(u_t^\theta(x) \approx u_t(x)\) using a network with weights \(\theta\). We inject randomness purely by sampling an initial condition \(X_0\) from a simple base distribution \(p_{\text{init}}\), such as a standard Gaussian \(\mathcal{N}(0, I_d)\). Our end goal is simply to have the terminal state \(X_1 = \psi_1^\theta(X_0)\) follow the true data distribution \(p_{\text{data}}\).

At inference time, because the neural vector field cannot be integrated analytically, generation is carried out by simulating the ODE numerically. Starting from \(X_0 \sim \mathcal{N}(0, I_d)\), we discretize time with a small step size \(h = 1/n\) and march forward along the vector field using the standard Euler method:


$$
X_{t+h} = X_t + h \cdot u_t^\theta(X_t)
$$

Iterating this from \(t = 0\) to \(1\) pushes the initial Gaussian noise along the learned velocity lines, delivering the final sample \(X_1 \approx z \sim p_{\text{data}}\).

### Diffusion model

Basically, the diffusion model is just like the flow model but with the evolution rule to be stochastic.

To make the deterministic trajectory random, we incorporate a continuous random walk driven by a standard Brownian motion (or Wiener process) \(W_t\), which possesses continuous paths and independent Gaussian increments \(W_{t+h} - W_t \sim \mathcal{N}(0, h I_d)\). Adding these stochastic kicks to an infinitesimal step of an ODE yields the standard Stochastic Differential Equation (SDE):


$$
dX_t = u_t(X_t)dt + \sigma_t dW_t, \quad X_0 \sim p_{\text{init}}
$$

Here, \(u_t(x)\) acts as the deterministic drift vector field, while \(\sigma_t \ge 0\) is a scalar diffusion coefficient modulating the magnitude of injected noise over time. 

Just like in flow models, the neural network only needs to learn the drift field \(u_t^\theta(x)\), leaving \(\sigma_t\) as a pre-designed, fixed schedule. Sampling from a diffusion model then mirrors the flow setting, but replaces the Euler integrator with its stochastic counterpart, the Euler-Maruyama method:


$$
X_{t+h} = X_t + h \cdot u_t^\theta(X_t) + \sigma_t \sqrt{h} \cdot \epsilon_t, \quad \epsilon_t \sim \mathcal{N}(0, I_d)
$$

At each interval, the state takes a small step in the direction of the vector field and simultaneously receives a random Gaussian nudge scaled by \(\sigma_t \sqrt{h}\). If we turn off this noise injection entirely by setting \(\sigma_t = 0\), the stochastic term vanishes and we immediately recover the deterministic flow model, revealing that flow models are simply zero-diffusion special cases of the broader SDE family.

> [!NOTE] Thought
> Why we introduce some stochastic factor? A converged explaination is about **diversity**. Flow model is simple and graceful but too rigrid and determinated. However, I have some reservation about this opinion which should be taken more seriouly.🤔
