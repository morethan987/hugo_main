---
title: 流匹配与扩散模型
weight: -130
draft: false
description: 流匹配与扩散模型相关笔记
slug: flow-matching-and-diffusion
language: zh-cn
tags:
  - 架构
  - 生成模型
  - 笔记
series:
  - AI工程
series_order: 2
date: 2026-10-02
lastmod: 2026-10-07
authors:
  - Morethan
---
{{< katex >}}

去噪扩散模型与流匹配模型是如今顶尖图像、音频和视频生成模型的核心基石。因此，熟悉相关技术并建立直观的物理与数学直觉，既十分必要，又非常有趣。

本文主要是我在学习 MIT 一门极棒的课程 [MIT 6.S184](https://diffusion.csail.mit.edu/2026/index.html) 时的学习笔记。

## 概述

首先，我们的初衷是想“生成点东西”，比如一张图片或者一段视频。这是我们工作的最初动机。但究竟什么是所谓的“东西”？在数学上我们又该如何形式化地表达它？

实际上，所有我们感兴趣的对象都可以表示为一个向量 \(z \in \mathbb{R}^d\)，或者能够展平为一个向量。因此，与其说“我想要一张图片”，现在我们可以说“我想要这张图片对应的向量”。这就引出了核心观点之一：

> **万物皆向量（An object is a vector）**


下一个不太严谨的词是“生成（generation）”。直观上讲，当你想到“一张小狗的图片”时，脑海中会浮现出无数张不同小狗的模样，你很难说哪一张才是唯一正确的，只能评判哪一张更好看、更合理。注意到这与从某个数据分布中抽取一个样本的行为如出一辙，我们就可以得出结论：“生成”本质上就是“采样”。所以这里的第二个核心思想是：

> **生成即采样（Generation can be viewed as sampling）**


现在目标就清晰多了：我们需要找到目标向量所在的概率分布。再往前推一步，当我们想要生成不同的东西时，我们可能需要“切换”分布——比如从“小狗分布”切换到“小猫分布”。

*因此，最终形式化的目标，就是从条件分布 \(p_{\text{data}}(\cdot|y)\) 中采样一个向量 \(z\)，其中 \(y\) 是描述我们需求的提示词（prompt）。*

## 思路

如果我们仅仅想生成小狗的图片，该如何表示这个分布呢？这是最终目标的一个简化版本。最朴素的想法是用一个神经网络来充当这个分布：它就像一台吃进随机种子（random seeds）就能吐出小狗图片的老虎机。

这个想法非常自然，也正是过去十年里大家一直在探索的路线。但最核心的挑战在于：如何将随机种子与真实的小狗图片绑定起来？比如摆在你面前的是一张哈士奇的照片，那它对应的随机种子到底该是哪一个？如果你忽略这一点，直接用均方误差（MSE）损失函数去暴力训练模型，你最终只会得到一只“平均化”的狗——糊成一团，甚至都很难辨认出那到底是不是一只狗。🤣

为了解决这个问题，我们需要重新定义挑战：其实我们根本没必要去较真哪个种子非得对应哪只“哈士奇”，我们真正需要做的，是训练我们的模型分布，使其在整体上**尽可能接近**真实的“小狗分布”。

这自然而然地将目标从“比对单个像素”转向了“度量两个概率分布之间的距离”。一旦我们不再纠结于像素级别的强制配对，而是引入巧妙的设计——比如对抗性判别器（GAN）或多步去噪（Diffusion），将生成分布作为一个整体拉向真实分布，那种模糊的“幽灵平均图”就会彻底消失，逼真、清晰的小狗便脱颖而出。

但问题随之而来：即便我们放弃了像素级的 MSE，转向分布匹配博弈（如 GAN），强求一个神经网络**一步跨越**——直接把一个简单的高斯噪声团变换成极其复杂、支离破碎的真实图像流形，这在数学上简直是**残酷至极**（mathematically brutal）。这种单步映射通常是极度不连续且病态（ill-conditioned）的。这种“一步到位”的跨越，正是早期生成模型饱受训练极度不稳定、模式崩溃（mode collapse）或严苛架构限制困扰的根源所在。

这正是 MIT 6.S 184 所带来的重大范式转变——**流与扩散：

> **与其尝试一次不可能的惊天一跃，何不架起一座连续渐进的桥梁？**


如果从纯高斯噪声 \(z \sim \mathcal{N}(0, I)\) 一步跨到一张清晰的哈士奇图像 \(x \sim p_{\text{data}}\) 过于困难，我们完全可以把这段旅程拆解为沿时间轨迹 \(t \in [0, 1]\) 展开的平滑增量步骤。在路径上的任意一点，网络不再需要凭空捏造出一整只狗，它只需要告诉我们**局部应该朝哪个方向推动样本一点点**——无论是预测局部的速度向量（流匹配 Flow Matching），还是抹去一丁点微弱的噪声（扩散模型 Diffusion）。通过将混乱的全局非线性映射转化为一系列简单、局部稳定的回归任务，我们终于赢得了稳定的训练过程和令人惊艳的生成质量。

> [!NOTE]+ Note
> 这样做还附带了一个绝佳的好处：在流或扩散的过程中，我们的提示词（prompt）可以被极其自然、合理地融入引导。你可以想象一下，如果试图通过单步映射直达目标，要把提示词的语义和最终分布融合起来该有多么复杂。

从更宏观的视角来看，这种方法可以被视作是在**时间维度**上拓展“深度”，而非单纯在模型架构的层数维度上堆叠深度。

## 流模型

概括来说，我们当下的任务是去描述一个简单的噪声分布是如何逐步演化成目标数据分布的。

幸运的是 🐸，常微分方程（ODE）为这种变换过程提供了一个极其优雅的数学框架。具体而言，如果我们能找到一个与时间相关的向量场 \(u_t(x)\)，使其能够在 \(t = 0\) 到 \(t = 1\) 的时间跨度内驱动生成我们期望的概率密度路径 \(p_t(x)\)，那么我们就可以训练一个参数为 \(\theta\) 的神经网络 \(u_t^\theta(x)\) 来对其进行逼近拟合。

顺理成章地，该回归训练目标，即 **流匹配损失（Flow Matching loss）**，可以形式化为：


$$
\mathcal{L}_{\text{FM}}(\theta) = \mathbb{E}_{t \sim \text{Unif}[0, 1], \, x \sim p_t} \left[ \| u_t^\theta(x) - u_t(x) \|^2 \right]
$$

然而，一个根本性的困境随即浮现：在实际应用中，\(u_t(x)\) 是完全不可解的。我们能拿到的仅仅是来自真实数据分布 \(p_{\text{data}}\) 的经验样本，又该如何确定推动这整个分布演化的边缘向量场 \(u_t(x)\) 呢？

为了厘清 \(u_t(x)\) 的数学定义以及计算方式，我们不妨将视角从单个粒子的动力学行为推广到宏观的概率分布上。

单个粒子的运动轨迹 \(X: [0, 1] \to \mathbb{R}^d, \, t \mapsto X_t\) 由一个时变向量场 \(u_t: \mathbb{R}^d \to \mathbb{R}^d\) 决定，该向量场指明了空间中任意位置与时间点的瞬时速度：


$$
\frac{d}{dt} X_t = u_t(X_t), \quad X_0 = x_0
$$

在整个时间跨度上的连续解由流映射（flow map）\(\psi_t: \mathbb{R}^d \to \mathbb{R}^d\) 刻画，它追踪粒子的位置 \(\psi_t(x_0) = X_t\)，并满足：


$$
\frac{d}{dt}\psi_t(x_0) = u_t(\psi_t(x_0)), \quad \text{with } \psi_0(x_0) = x_0
$$

从物理直觉来看，\(u_t\) 无非是一个输入时刻 \(t\) 与坐标 \(x\)，并输出粒子在该处瞬时速度向量的函数。

而当我们把视野放大到由无数粒子汇聚而成的整体、并在时间维度上形成一个连续的**概率分布** \(p_t(x)\)（即**概率密度路径**）时，事情就变得非常有意思了。

由于概率质量守恒（粒子既不会凭空产生，也不会凭空消失），由向量场 \(u_t(x)\) 推动的空间密度 \(p_t(x)\) 必然满足**连续性方程**：


$$
\frac{\partial p_t(x)}{\partial t} = - \nabla \cdot \big( p_t(x) u_t(x) \big)
$$

既然从全局视角直接处理边缘分布 \(p_t(x)\) 和向量场 \(u_t(x)\) 异常棘手，我们不妨引入一个条件变量 \(z\)（例如目标数据点 \(x_1 \sim p_{\text{data}}\)，或者初末状态对 \((x_0, x_1)\)）：


$$
p_t(x) = \int p_t(x \mid z) p(z) \, dz
$$

此时，每一条条件路径 \(p_t(x \mid z)\) 都有其形式简单、定义清晰的条件向量场 \(u_t(x \mid z)\)，并满足属于它自己的连续性方程：


$$
\frac{\partial p_t(x \mid z)}{\partial t} = - \nabla \cdot \big( p_t(x \mid z) u_t(x \mid z) \big)
$$

对边缘分布 \(p_t(x)\) 求关于时间的偏导数：


$$
\begin{aligned} \frac{\partial p_t(x)}{\partial t} &= \int \frac{\partial p_t(x \mid z)}{\partial t} p(z) \, dz \\ &= - \nabla \cdot \int p_t(x \mid z) u_t(x \mid z) p(z) \, dz \\ &= - \nabla \cdot \left( p_t(x) \int u_t(x \mid z) \frac{p_t(x \mid z) p(z)}{p_t(x)} \, dz \right) \\ &= - \nabla \cdot \left( p_t(x) \int u_t(x \mid z) p_t(z \mid x) \, dz \right) \end{aligned}
$$

将此推导与最初的边缘连续性方程 \(\frac{\partial p_t(x)}{\partial t} = - \nabla \cdot \big( p_t(x) u_t(x) \big)\) 进行对比，我们便能极其自然地将边缘向量场定义为：


$$
u_t(x) = \int u_t(x \mid z) p_t(z \mid x) \, dz = \mathbb{E}_{z \sim p_t(z \mid x)} \left[ u_t(x \mid z) \right]
$$

这一至关重要的结论表明：看似复杂莫测的全局向量场 \(u_t(x)\)，本质上不过是那些容易求解的条件向量场 \(u_t(x \mid z)\) 的后验期望整合。

重温我们最初的流匹配优化目标：


$$
\mathcal{L}_{\text{FM}}(\theta) = \mathbb{E}_{t \sim \text{Unif}[0, 1], \, x \sim p_t} \left[ \| u_t^\theta(x) - u_t(x) \|^2 \right]
$$

把边缘向量场 \(u_t(x) = \mathbb{E}_{z \sim p_t(z \mid x)} [u_t(x \mid z)]\) 直接代入该损失函数并展开平方项，便能揭示出一个神奇的事实：\(\mathcal{L}_{\text{FM}}(\theta)\) 与条件流匹配（Conditional Flow Matching）目标仅相差一个与 \(\theta\) 无关的常数项：


$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathbb{E}_{t \sim \text{Unif}[0, 1], \, z \sim p(z), \, x \sim p_t(x \mid z)} \left[ \| u_t^\theta(x) - u_t(x \mid z) \|^2 \right]
$$

具体来说，可以严格证明：


$$
\mathcal{L}_{\text{CFM}}(\theta) = \mathcal{L}_{\text{FM}}(\theta) + C \quad \implies \quad \nabla_\theta \mathcal{L}_{\text{FM}}(\theta) = \nabla_\theta \mathcal{L}_{\text{CFM}}(\theta)
$$

具体证明过程可参见[课程讲义](https://diffusion.csail.mit.edu/2026/docs/lecture_notes.pdf)，其核心前提正是我们在前面已经证毕的结论：\(u_t(x)\) 可以表示为条件向量场的后验期望。

既然两者对 \(\theta\) 的梯度完全相同，那么去最小化易于求解的条件目标 \(\mathcal{L}_{\text{CFM}}(\theta)\)，在数学上就完全等价于最小化无法直接求解的边缘目标 \(\mathcal{L}_{\text{FM}}(\theta)\)。在实际工程落地时，我们只需采样条件 \(z\)、按照可自由定义的简单演化形式采样 \(x \sim p_t(x \mid z)\)、计算出解析形式的条件速度 \(u_t(x \mid z)\)，便可以通过最平凡的均方误差（MSE）回归来训练网络了。

## 扩散模型

Score Matching的数学体系从一开始其实就是完全自洽独立的。我们将边缘得分定义为对数密度的梯度 \(\nabla \log p_t(x)\)。尽管真实的边缘概率密度 \(p_t(x)\) 难以捉摸，但我们拥有一条优美的边缘化恒等式：


$$
\nabla \log p_t(x) = \int \nabla \log p_t(x|z) \frac{p_t(x|z)p_{\text{data}}(z)}{p_t(x)} \mathrm{d}z
$$

通过将针对该边缘得分的均方误差展开，无法直接计算的边缘目标便能直接化简为条件去噪得分匹配损失：\(\mathbb{E}[\|s_t^\theta(x) - \nabla \log p_t(x|z)\|^2]\)。在理论上，仅凭这一个恒等式就足够了：得分匹配即便完全脱离向量场或流模型的概念，也能够独立构建公式、完成训练并闭环。

然而，训练出一个得分模型仅仅只走完了一半路程，因为单凭得分函数本身并不足以生成新样本。得分函数指明的仅仅是概率密度局部递增的方向，它本身并不会主动将概率质量从初始噪声 \(p_{\text{init}}\) 运送到数据分布 \(p_{\text{data}}\)。如果我们想要通过模拟随机微分方程（SDE）来生成样本，同时保证 \(X_t\) 的边缘分布能够严格沿着我们期望的概率路径 \(p_t\) 演化，福克-普朗克方程（Fokker-Planck equation）要求该轨迹必须满足如下形式：


$$
\begin{aligned} \mathrm{d}X_t &= u_t(X_t) \mathrm{d}t + \frac{\sigma_t^2}{2} \nabla \log p_t(X_t) \mathrm{d}t + \sigma_t \mathrm{d}W_t \\ &= \left[ u_t(X_t) + \frac{\sigma_t^2}{2} \nabla \log p_t(X_t) \right] \mathrm{d}t + \sigma_t \mathrm{d}W_t \end{aligned}
$$

在此，布朗运动项 \(\sigma_t \mathrm{d}W_t\) 注入了随机扰动，而 \(\frac{\sigma_t^2}{2} \nabla \log p_t(X_t)\) 项则起到向内修正的作用，用于抵消噪声所带来的扩散与发散。但仔细观察漂移项（drift）就会发现：它明确地**同时**需要确定性速度场 \(u_t(X_t)\) 与得分函数 \(\nabla \log p_t(X_t)\)。仅有得分只能提供拉回的修正力，却无法提供基准的前进轨迹。

这就引出了一个尴尬的现实：对于一条任意设计的概率路径而言，速度场 \(u_t\) 和得分场 \(\nabla \log p_t\) 是两个截然不同的数学对象，二者之间并没有简单的对应关系。倘若我们随意选取分布路径，就会陷入工程层面的噩梦——仅为了运行一次 SDE 采样循环，我们必须**训练两个独立的神经网络**，一个用流匹配去学 \(u_t\)，另一个用得分匹配去学 \(\nabla \log p_t\)。

这一尴尬的困境，恰恰解释了为什么实际落地中的扩散模型几乎普遍约定俗成地采用高斯概率路径 \(p_t(x|z) = \mathcal{N}(x; \alpha_t z, \beta_t^2 I_d)\)。在高斯路径下，条件速度场 \(u_t(x|z)\) 和条件得分 \(\nabla \log p_t(x|z)\) 恰好全都是关于 \(x\) 和 \(z\) 的仿射函数。与后验积分之后，二者都会塌缩为对同一个后验均值 \(\mathbb{E}[z \mid x]\)（即去噪器）的线性重参数化。由此，架起了一座解析形式的桥梁：


$$
u_t(x) = a_t \nabla \log p_t(x) + b_t x
$$

正得益于这种完全等价的关系，训练单个网络就能让我们免费白嫖到另一个物理量。

> [!NOTE] 思考
> 为什么我们要引入随机因素？一个普遍公认的解释在于**多样性**。流模型虽然简洁优雅，但未免过于死板且具备确定性。此外，流模型在训练中不可避免地会产生一些在理论上难以完全消除的逼近误差。通过引入随机因素，我们能够借助调节 SDE 中的 \(\sigma\) 参数，期望在生成质量与多样性上取得更佳的表现。

## 引导生成

前面讨论的算法并没有考虑用户的提示。通常，我们希望生成的结果能顺着我们给出的提示或其他额外信息的方向走。

最朴素的带引导条件的流匹配目标函数是：


$$
\mathcal{L}_{\mathrm{CFM}}^{\mathrm{guided}}(\theta) = \mathbb{E}_{(z,y) \sim p_{\mathrm{data}}(z,y), \, t \sim \mathrm{Unif}, \, x \sim p_t(\cdot | z)} \| u_t^\theta(x|y) - u_t(x|z) \|^2.
$$

这个损失函数与我们之前的版本相比，唯一的区别就是加入了标签 \(y\)。我们期望最终生成的 \(X_1\) 能够忠实地遵循带条件的分布 \(p_{\mathrm{data}}(\cdot|y)\)。然而，这个方法理论上完美，实践中却不尽人意。下图就是一个例子：左边的图片质量明显低于右边的。

![img/vanilla_guidance_vs_classifier_guidance.png](img/vanilla_guidance_vs_classifier_guidance.png)

这里引用一下[课程讲义](https://diffusion.csail.mit.edu/2026/docs/lecture_notes.pdf)（第 35 页底部）的一段话：

> 导致这种情况的原因可能有很多：模型可能欠拟合（即我们没有真正学到边际向量场），或者我们的数据本身不完美（例如，来自互联网的图文对存在大量错误）。因此，为了生成更贴合提示的样本，我们必须找到一种方法来人为地**强化**提示变量 \(y\) 的作用。


为此，我们从速度场和分数的关系式入手，然后应用贝叶斯定理来推导结果。


$$
\begin{aligned} u_t(x|y) &= a_t \nabla_x \log p_t(x|y) + b_t x \\ &= a_t \nabla_x \log \left[ \frac{p_t(x) p_t(y|x)}{p_t(y)} \right] + b_t x \\ &= b_t x + a_t \left[ \nabla_x \log p_t(x) + \nabla_x \log p_t(y|x) \right] \\ &= u_t(x) + a_t \nabla_x \log p_t(y|x) \end{aligned}
$$

这个等式告诉我们，朴素的引导速度场 \(u_t(x|y)\) 等于无引导的速度场 \(u_t(x)\) 加上一个分数项 \(\nabla_x \log p_t(y|x)\)。要强化 \(y\) 的影响，一个简单的方法就是给这个分数项乘上一个大于 1 的缩放因子 \(w\)：


$$
\tilde{u}_t(x|y)=u_t(x) + w\cdot a_t \nabla_x \log p_t(y|x)
$$

这就是**分类器引导模型**的核心思想。但为什么叫“分类器”呢？实际上，为了得到这个强化后的速度场 \(\tilde{u}_t(x|y)\)，我们需要两个神经网络：一个用来预测标准的速度场，另一个则用来预测分数场 \(\nabla_x \log p_t(y|x)\)。第二个网络的作用是根据给定的 \(x\) 来预测 \(y\) 的分布，这其实就是在对“加噪”的中间状态 \(x\) 进行分类，判断它属于哪个标签 \(y\)。因此，这种模型被称为分类器引导模型。

可以想象，训练一个网络在加噪的 \(x\) 上做分类任务，既麻烦又不稳定。因此，我们需要把这个棘手的分数项变得更简单一些。回想一下，当基础分布是高斯分布时，分数只是速度场的一种重新参数化的形式。那么，我们应该有机会将分数转换回速度场。


$$
\begin{aligned} \tilde{u}_t(x|y) &= u_t(x) + w a_t \nabla \log p_t(y|x) \\ &= u_t(x) + w a_t (\nabla \log p_t(x|y) - \nabla \log p_t(x)) \\ &= u_t(x) - (w b_t x + w a_t \nabla \log p_t(x)) + (w b_t x + w a_t \nabla \log p_t(x|y)) \\ &= (1 - w) u_t(x) + w u_t(x|y) \end{aligned}
$$

这个等式告诉我们，强化后的速度场 \(\tilde{u}_t(x|y)\) 其实是标准速度场 \(u_t(x)\) 和引导速度场 \(u_t(x|y)\) 的一个线性组合。那么问题来了：我们能不能把这个线性组合操作移到数据层面呢？换句话说，我们能否用一个类似drop out的技巧，让单个神经网络同时估算出 \(u_t(x)\) 和 \(u_t(x|y)\)？答案是肯定的，下面是证明。

为了形式化地描述这个过程，我们首先扩展标签空间为 \(\tilde{Y} = Y \cup \{\emptyset\}\)，其中 \(\emptyset\) 代表一个特殊的空（null）标识，表示没有条件。在训练时，对于每个数据对 \((z, y)\)，我们以 \(\eta\) 的概率随机决定是否丢弃标签 \(y\)。这样，输入给模型的条件变量 \(c\) 就有两种可能：以 \(1-\eta\) 的概率为 \(y\)，以 \(\eta\) 的概率为 \(\emptyset\)。

这里的关键在于，丢弃标签的决策是一个独立的随机过程，所以事件 \(c = \emptyset\) 与样本状态 \(x\) 在统计上是独立的。对中间的加噪分布应用贝叶斯定理，我们得到：


$$
p_t(x \mid c = \emptyset) = \frac{p(c = \emptyset \mid x) p_t(x)}{p(c = \emptyset)} = \frac{\eta \cdot p_t(x)}{\eta} = p_t(x).
$$

这个恒等式表明，当条件是空标签 \(\emptyset\) 时，中间数据分布 \(p_t(x \mid c=\emptyset)\) 与真正的无条件边际分布 \(p_t(x)\) 完全相同。

现在，我们考虑用一个单一的向量场网络 \(u_t^\theta(x|c)\) 在这个扩展后的数据分布上进行训练，使用的仍然是标准的均方误差损失函数：


$$
\mathcal{L}_{\mathrm{CFM}}^{\mathrm{CFG}}(\theta) = \mathbb{E}_{(z, c), \, t \sim \mathrm{Unif}, \, x \sim p_t(\cdot|z)} \| u_t^\theta(x|c) - u_t(x|z) \|^2.
$$

在 L 2 回归的目标下，对于任何给定的输入 \((x, c)\)，模型的最优解是目标向量场的后验条件期望，即 \(u_t^*(x|c) = \mathbb{E}[u_t(x|z) \mid x_t = x, c]\)。我们分别在两种标签情况下考察这个最优解：

当条件为 \(y\) 时： $\(u_t^*(x \mid y) = \mathbb{E}[u_t(x|z) \mid x_t = x, c = y] = u_t(x|y),\)$ 这恰好就是我们想要的条件向量场。

而当条件为空标签 \(\emptyset\) 时： $\(u_t^*(x \mid \emptyset) = \mathbb{E}[u_t(x|z) \mid x_t = x, c = \emptyset] = \mathbb{E}[u_t(x|z) \mid x_t = x] = u_t(x),\)\(由于\)\emptyset$ 的条件独立性，模型自然地学到了无引导的边际向量场。

这证明了，通过简单地在训练数据中引入一个统计独立的空条件，单个神经网络 \(u_t^\theta(x|c)\) 就能学会在不同的条件下输出不同的目标场。在采样时，我们只需要用同一个网络，分别输入真实提示 \(y\) 和空提示 \(\emptyset\) 进行两次评估，然后通过 \(\tilde{u}_t(x|y) = (1 - w) u_t^\theta(x|\emptyset) + w u_t^\theta(x|y)\) 来计算最终的引导轨迹，完全不需要训练两个独立的模型。

注意到我们再也不需要分类器了，因此这种模型被称为**无分类器引导模型**。

你可能会想：我们能不能通过调整数据分布，让模型直接学习 \((1 - w)\) 和 \(w\) 这两个权重呢？答案是不能。因为我们想要的 \(w\) 通常大于 1，这意味着 \(1-w\) 是负数，而负数是不能作为概率的。

最后还需要强调一点：强化后的结果 \(\tilde{X}_1\) 不一定完全符合原始的条件分布 \(p_{\mathrm{data}}(\cdot|y)\)。实际上，模型学到的是一个被**锐化**了的分布，它的多样性更低，但精准度更高。下面是课程讲义中的一段评论：

![img/lecture_note_comment_on_classifier_free.png](img/lecture_note_comment_on_classifier_free.png)

## 自编码器

在前面的章节中，我们已经了解了如何训练模型来拟合速度场 \(u_t(x)\)。在推理阶段，我们可以通过数值模拟 ODE/SDE 得到最终的目标分布。然而在实际模拟 ODE/SDE 时会遇到一个现实挑战：如果我们希望生成高分辨率的高清图像，\(u_t(x)\) 就必须在一个极高维的空间中运行，这不仅计算代价极其昂贵，而且模型也极难训练。

> [!NOTE]+ 补充说明
> 在图像生成的语境下，\(u_t(x)\) 本质上是在指导我们如何修改图像所有通道上的**每一个像素**。

此外，在真实的图像生成任务中，其实存在大量的空间与语义冗余。例如在下图中，黑色的背景部分无论如何都应该保持全黑。这启发我们可以将原始图像压缩到更紧凑的形式，让生成模型在更低维的表示上进行建模。

![img/redundancy_example.png](img/redundancy_example.png)

这便引出了**自编码器**：它包含一个将图像映射到低维空间（**潜空间 / latent space**）的编码器，以及一个将潜空间中的点重构回原始图像的解码器。

![img/autoencoder.png](img/autoencoder.png)

### 标准自编码器

形式化地，我们记编码器为 \(\mu_{\phi}: \mathbb{R}^d \to \mathbb{R}^k\)，解码器为 \(\mu_{\theta}: \mathbb{R}^k \to \mathbb{R}^d\)。标准自编码器直接采用朴素的重构损失进行训练：


$$
\mathcal{L}_{\mathrm{Recon}}(\phi, \theta) = \mathbb{E}_{x \sim p_{\mathrm{data}}} \left[ \| \mu_\theta(\mu_\phi(x)) - x \|^2 \right]
$$

这个损失函数很直观。但我们的核心目标并不只是得到一个单纯的自编码器，而是要为生成模型构建一个规整的潜空间。这就意味着我们需要对潜空间的分布形态拥有更多的控制权。

### 变分编码器

相比标准自编码器，变分自编码器（VAE）最大的转变在于：它将编码和解码映射视为了在潜在概率分布上的**采样**操作。因此，编码器和解码器分别被定义为条件概率分布：


$$
q_\phi(z|x) = \mathcal{N}(z; \mu_\phi(x), \operatorname{diag}(\sigma_\phi^2(x))), \quad p_\theta(x|z) = \mathcal{N}(x; \mu_\theta(z), \sigma_\theta^2(z)I_d)
$$

其中 \(\mu_\phi(x) \in \mathbb{R}^k\)、\(\sigma_\phi^2(x) \in \mathbb{R}_{\ge 0}^k\)、\(\mu_\theta(z) \in \mathbb{R}^d\) 以及 \(\sigma_\theta^2(z) \in \mathbb{R}_{\ge 0}\) 均由神经网络参数化实现，\(\operatorname{diag}\) 表示对角矩阵。变量的编码与解码则通过采样来完成：


$$
z \sim q_\phi(\cdot|x), \ x \sim p_\theta(\cdot|z)
$$

在此视角下，重构损失被定义为负对数似然：


$$
\mathcal{L}_{\text{VAE-Recon}}(\phi, \theta) = -\mathbb{E}_{x \sim p_{\text{data}}(x), z \sim q_\phi(\cdot|x)} \left[ \log p_\theta(x|z) \right]
$$

它的直观含义是：对于从条件分布 \(q_\phi(\cdot|x)\) 中采样出的 \(z\)，模型在解码出原始数据 \(x\) 时的对数概率应该尽可能大。代入高斯分布后，该损失展开为：


$$
\mathcal{L}_{\text{VAE-Recon}}(\phi, \theta) = \mathbb{E}_{x \sim p_{\text{data}}(x), z \sim q_\phi(z|x)} \left[ \frac{1}{2\sigma_\theta^2(z)} \|x - \mu_\theta(z)\|^2 + \frac{d}{2} \log \sigma_\theta^2(z) \right] + \mathrm{const}
$$

在工程实践中，为了避免病态行为并获得更好的数值稳定性，通常会将解码方差固定为常数。此时损失函数简化为：


$$
\mathcal{L}_{\text{VAE-Recon}}(\phi, \theta) = \mathbb{E}_{x \sim p_{\text{data}}(x), z \sim q_\phi(z|x)} \left[ \frac{1}{2\sigma_\theta^2} \|x - \mu_\theta(z)\|^2 \right] + \mathrm{const}
$$

可以看出，此时的重构损失在数学形式上已等价于标准的均方误差重构损失。而为了得到性质更优良的潜空间分布，我们首先需要明确何为“优良”。高斯分布凭借其极佳的数学性质在各类工作中被广泛验证，因此我们直接将标准高斯分布定义为理想的目标分布。接着，我们在总损失中引入正则化项，促使潜空间分布尽可能逼近标准高斯分布：


$$
\begin{align*} \mathcal{L}_{\text{VAE}}(\phi, \theta) &= \mathcal{L}_{\text{VAE-Recon}}(\phi, \theta) + \beta \mathcal{L}_{\text{VAE-Prior}}(\phi) \\ &= -\mathbb{E}_{x \sim p_{\text{data}}(x), z \sim q_\phi(z \mid x)} \left[ \log p_\theta(x \mid z) \right] + \beta \mathbb{E}_{x \sim p_{\text{data}}(x)} \left[ D_{KL}(q_\phi(\cdot \mid x) \parallel p_{\text{prior}}) \right] \\ &= \mathbb{E}_{x \sim p_{\text{data}}(x), z \sim q_\phi(z \mid x)} \left[ \underbrace{\frac{1}{2\sigma_\theta^2(z)} \|x - \mu_\theta(z)\|^2}_{\text{重构误差}} + \underbrace{\frac{d}{2} \log \sigma_\theta^2(z)}_{\text{解码置信度}} + \underbrace{\frac{\beta}{2}\mathcal{K}(\sigma_\phi^2(x))}_{\text{促使潜变量方差趋近 1}} + \underbrace{\frac{\beta}{2} \|\mu_\phi(x)\|^2}_{\text{促使潜变量均值趋近 0}} \right] \end{align*}
$$

其中，\(\beta\) 调节正则项的权重占比，\(D_{KL}\) 为衡量两个分布差异的 KL 散度。

值得注意的是，采样得到的 \(z\) 本身依赖于待训练的参数，直接采样操作无法进行反向传播求导。不过得益于高斯分布良好的数学特性，我们可以借助**重参数化技巧**将网络参数提取出来，使得梯度可以直接回传：


$$
\mathcal{L}_{\text{VAE}}(\phi, \theta) = \mathbb{E}_{x \sim p_{\text{data}}(x), \epsilon \sim \mathcal{N}(0, I_k)} \left[ \frac{1}{2\sigma_\theta^2(z)} \|x - \mu_\theta(\mu_\phi(x) + \sigma_\phi(x)\epsilon)\|^2 + \frac{d}{2} \log \sigma_\theta^2(z) + \frac{\beta}{2} \mathcal{K}(\sigma_\phi^2(x)) + \frac{\beta}{2} \|\mu_\phi(x)\|^2 \right]
$$

## 离散扩散模型

我们为什么需要离散扩散模型？因为并非所有现实数据都天然适合被建模为欧氏空间 \(\mathbb{R}^d\) 中的连续向量。诸如文本或 DNA 序列这类离散数据，更自然的做法是将其视作离散状态空间 \(S\) 中的元素。

回顾前面的内容，ODE 和 SDE 描述的是连续变量的演化过程；而此时的核心任务是构建一套对应的“离散版本 ODE/SDE”，这在概率论中被称为**连续时间马尔可夫链（Continuous-Time Markov Chain, CTMC）**：


$$
\frac{\mathrm{d}}{\mathrm{d}h} p_{t+h|t}(X_{t+h} = y \mid X_t = x) \Big|_{h=0} = Q_t(y \mid x) \quad \text{for all } x, y \in S, 0 \le t
$$

其中 \(Q_t(y \mid x)\) 定义了状态转移的**速率矩阵（rate matrix）**，刻画了从状态 \(x\) 跳变到状态 \(y\) 的瞬时速率；而 \(p_{t+h|t}(X_{t+h} = y \mid X_t = x)\) 则给出了在微小时间步长 \(h\) 内从状态 \(x\) 转移到 \(y\) 的转移概率。

对转移方程进行泰勒展开，我们便能够对其离散步长进行模拟：


$$
p_{t+h|t}(X_{t+h} = y \mid X_t = x) = p_{t|t}(X_t = y \mid X_t = x) + h Q_t(y \mid x) + R_t(h) = 1_{y=x} + h Q_t(y \mid x) + R_t(h)
$$

类似于 Flow 模型，通过在前向时间上模拟该 CTMC 过程，我们便能如愿将初始的噪声分布 \(p_{\text{init}}\) 逐步演化至真实数据分布 \(p_{\text{data}}\)。

然而，\(Q_t(y \mid x)\) 所处的全状态空间极其庞大。具体来说，当词表大小为 \(V\)、序列长度为 \(d\) 时，状态空间的大小为 \(|S| = V^d\)。为了让计算在工程上可行，我们必须对模型施加结构约束，最常用的策略就是**因子分解（factorization）**。下图展示了这一直观思路：

![img/factorized_ctmc_model.png](img/factorized_ctmc_model.png)

经过因子分解后，速率矩阵的计算开销大幅降低至可承受范围：


$$
x \mapsto \{Q_t^\theta(y \mid x)\}_{y \in N(x)} = \begin{pmatrix} Q_t^\theta(v_1, 1 \mid x) & \dots & Q_t^\theta(v_V, 1 \mid x) \\ \vdots & \ddots & \vdots \\ Q_t^\theta(v_1, d \mid x) & \dots & Q_t^\theta(v_V, d \mid x) \end{pmatrix}
$$

矩阵中的每个元素表示：在给定当前被掩码/带噪序列 \(x\) 的条件下，位置 \(i\) 转变（跳转）为词元 \(v_j\) 的瞬时速率。以上便是推理采样阶段的全部核心机制。

至于训练阶段，其整体逻辑与 Flow Matching 几乎完全一致。首先，我们求解条件概率下的转移速率：


$$
\begin{align*} Q_t^z(y \mid x) &= \left(Q_t^z(v_i, j \mid x_j)\right)_{v_i, j} \\ Q_t^z(v_i, j \mid x_j) &= \frac{\dot{\kappa}_t}{1 - \kappa_t} \left(\delta_{z_j}(v_i) - \delta_{x_j}(v_i)\right) \\ &= \frac{\dot{\kappa}_t}{1 - \kappa_t} \begin{cases} 0 & \text{if } x_j = z_j \\ 1 & \text{if } v_i = z_j, \, x_j \neq z_j \\ 0 & \text{if } v_i \neq z_j, \, x_j \neq z_j \\ -1 & \text{if } v_i = x_j, \, x_j \neq z_j \end{cases} \end{align*}
$$

对展开项的直观物理理解如下：

1. 若当前位置 \(x_j\) 已经是目标 token \(z_j\)，则转移速率为 \(0\)，概率保持不变；
2. 若候选词 \(v_i\) 是真实目标 \(z_j\)，但当前 \(x_j \neq z_j\)，则转移速率为正（\(=1\)），概率质量向其流入；
3. 若 \(v_i\) 与 \(x_j\) 均不是目标 \(z_j\)，则保持不动；
4. 若当前位置 \(x_j\) 为 \(v_i\)，但该值并非目标 \(z_j\)，则转移速率为负（\(=-1\)），概率质量向外流出。

随后，我们对所有已知样本取加权平均，即可得到边缘转移速率矩阵：


$$
Q_t(y \mid x) = \sum_{z \in S} Q_t^z(y \mid x) p_{1|t}(z \mid x)
$$

这里唯一未知的量是后验概率项 \(p_{1|t}(z \mid x)\)，它需要通过神经网络来进行参数化估计。它的物理含义是：在给定中间带噪状态 \(x\) 的情况下，预测其还原到最终真实样本 \(z\) 的概率。这与 BERT 模型所做的事情如出一辙——即给定被掩码的文本序列去预测被遮盖的真实词。因此，其训练目标也可以直接采用经典的交叉熵损失：


$$
\mathcal{L}_{\mathrm{DFM}}(\theta) = \mathbb{E}_{z \sim p_{\mathrm{data}}, \, t \sim \mathrm{Unif}_{[0, 1]}, \, x \sim p_t(\cdot \mid z)} \left[ \sum_{j=1}^d -\log p_{1|t}^\theta (z_j \mid x) \right]
$$

这个推导过程并不难理解，但或许会让人产生一丝疑问：文本生成真的有必要采用这种范式吗？它的优势与劣势究竟何在？

这在当前仍是一个开放性课题（Open Problem）：一部分人认为，基于扩散的文本生成采样更快，并且天然赋予了模型编辑已生成文本的能力；但另一部分人则指出，离散扩散模型难以适配诸如 KV Cache 等主流的高效推理加速机制，且相比于经典的自回归生成范式，目前尚未展现出显著的性能优势。🤔
