---
title: 笔记转博客项目分析报告
weight: -40
draft: false
description: 一个针对笔记转博客项目的分析报告
slug: note-to-blog-report
language: zh-cn
tags:
  - Hugo
  - 博客
  - 报告
series:
  - 项目报告
series_order: 1
date: 2025-01-10
lastmod: 2025-01-21
authors:
  - Morethan
---

{{< lead >}}
熟悉项目分析流程，是一个练手性质的分析报告
{{< /lead >}}

我在初步完成自己的[小插件]({{< ref "/blog/plugin-writing-experience/" >}})之后才发现：**笔记转博客**这个领域似乎没有一个一键式解决方案，为此我写了这篇文章来分析是否要创建一个新的项目来填补这个空白。

**项目没有优劣之分，只有合适与否**，这篇文章中涉及的评价标准都是围绕**是否符合博客网页要求**进行的，因此可能有些项目并不适合做博客网页，但是这丝毫不影响项目本身的价值。

## 领域定义

在所有分析开始之前，需要对于**笔记转博客**这个领域有一个清晰的界定。

- **笔记**：此处的笔记特指 Obsidian 中的笔记，含有 Obsidian Flavored Markdown，这对笔记向网页转换的过程提出了不小的要求。

- **博客**：博客，顾名思义是一种获得流量的手段，创作者花费时间创作笔记，然后经由转换软件，生成排版精美、功能丰富的博客网页

评价细则如下：

- **隐私方面**
	- 是否本地运行
	- 是否开源

- **使用舒适性**
	- 与 Obsidian 语法的适配程度
	- 服务部署复杂程度
	- 帮助文档撰写详细程度
	- 个性化调整复杂设置

- **网页功能完整性**
	- 默认配套网页的核心功能是否齐全（搜索、日间/夜间模式等）
	- 默认网页美观程度
	- 是否支持 SEO
	- 对 Obsidian 原生语法的转化效果是否生硬（例如：展示性链接内是否存在不能翻译的代码块、是否舍弃部分 Obsidian 语法特性等）

## 项目概述

在我的设想中，这个项目的功能就是构建一个 Obsidian 插件，实现从 Obsidian 笔记到 Hugo 博客网页的无缝导出，支持 Obsidian 的所有基本核心功能。

成效：大大降低了创建博客网页的成本，只要能够使用 Obsidian 就能够拥有属于自己的博客网页。

## 市场和用户可行性分析

### 市场需求分析

#### 概述

- 基本需求：搭建个人网站并持续输出的需求，包括自我提升、自我表达、创造独特全面的个人能力展示平台（对接**企业招聘**）等等

- 目标用户群：重度 Obsidian 用户且有分享笔记的需求；想要搭建个人博客但是因为技术难度而放弃的知识创作者

#### 相关数据

- Flowershow：截至 2024 年 10 月插件下载量为 3355；截至 2025 年 1 月插件下载量为 4594，同时最高下载量的插件有 3211992 的下载量；
- Quartz 截至 2025 年 1 月收获 GitHub start 数量为 7.7k

### 已有方案

#### Quartz

推荐指数：❤️‍🔥❤️‍🔥❤️‍🔥❤️‍🔥❤️‍🔥

##### 介绍

[Quartz](https://github.com/jackyzha0/quartz) 是一个将 Obsidian 笔记转化为网页的工具集。Quartz 的最新版本是 `v4` 版本，相较于 `v3` 版本， `v4` 版本从底层完全重构了代码，去除了对于 `Hugo` 的依赖，优化了用户自定义的体验。目前 `v4` 版本主要使用 `TypeScript` 构建，原本 `hugo` 的 `template` 也改用 `JSX` 替换。

因此，现在的 Quartz 几乎可以说和 `Hugo` 没什么关系了，但是目前国内的很多信息还是宣传 Quartz 的底层是 `Hugo`

官方样例网站：[Welcome to Quartz 4](https://quartz.jzhao.xyz/)

##### 评述

- 优点
	- 功能非常非常完整
	- 所有相关套件中唯一成功解决了展示性 wiki 链接的一个
	- 配套文档很详细

- 缺点
	- 几乎没有缺点，唯一一个值得提点的地方就是没有中文的文档

总结：非常优秀的项目，所有在 Obsidian 内显示的样式就是网页中显示的样式，确实也收获了所有现成方案中最多的 GitHub start 数量

#### Flowershow

推荐指数：❤️‍🔥❤️‍🔥❤️‍🔥❤️‍🔥🩶

##### 介绍

[Flowershow](https://flowershow.app/) 是一个基于 Obsidian 的整体发布服务，它可以将你的 Obsidian 笔记按照目录结构，转换为一个在线的数字花园网站。而 [Vercel](https://vercel.com/) 是一个针对前端的云服务，它实现了通过 Github 免服务器快速部署前端服务，每次提交内容都会触发一次自动部署。Flowershow 的官方部署文件中需要使用 Vercel，国内使用可以考虑 [Netlify](https://www.netlify.com/) 替换

参考教程：[Flowershow：免费的 Obsidian 笔记发布服务，实现你的数字花园网站](https://utgd.net/article/20663/)

主观上来讲，这个项目的主创团队是一支很有激情和使命感的队伍，关于 Flowershow 的[简介](https://flowershow.app/about)里面包含了很多主创团队的核心理念。

客观上来讲，Flowershow 的项目定位非常准确，就是一个基于 Obsidian 的博客网页生成平台，因此最终效果从前端和后端两个角度来说都是非常好的。

##### 评述

- 优点
	- 定位清晰，工作流程简洁明了
	- 功能支持较为全面
	- 背后有专业的团队进行运维
	- 允许高度的自定义，适合喜欢个性化的创作者

- 缺点(一下信息均来自 2025 年 1 月)
	- 部分 Obsidian 的功能并没有处理，例如展示性的 wiki 链接并没有处理，至少介绍文档中跳过了这部分内容
	- 仔细翻找了网站并没有发现反向链接的说明，但是首页信息显示能够支持反向链接

总结：总体上项目做的还是挺好，但是项目还在进行中，部分细节并不到位，对于 Obsidian 语法支持没有那么高要求的创作者就可以采用这个方案了

#### 官方发布
推荐指数：❤️‍🔥❤️‍🔥❤️‍🔥🩶🩶
##### 介绍
下面是几个样例网站：

- [Obsidian中文教程](https://publish.obsidian.md/chinesehelp/01+2021%E6%96%B0%E6%95%99%E7%A8%8B/2021%E5%B9%B4%E6%96%B0%E6%95%99%E7%A8%8B)一个中文教程网站，使用了 Obsidian 官方的发布服务，里面里面可以体现一些功能，例如对于展示性链接的处理
- [Digital 3D Garden](https://stephanlevin.garden/Welcome) 有深度的前端界面自定义
- [mister chad](https://mister-chad.com/welcome) 非常简洁工整的小站，内容很充实
- [Discrete Structures for Computer Science](https://publish.obsidian.md/discretecs/START+HERE) 神似官方的朴素风格

##### 评述

- 优势
	- 官方发布的网页对于 Obsidian 内部表达的适配效果是一流的，Obsidian 内部所有的功能都能够成功在网页中展示；但是不清楚插件功能如何在网页中展示
	- 有持续的维护服务，能够第一时间适配 Obsidian 的更新
	- 支持高度的个性化设置，如果具有充足的代码经验，可以开发出相当精美的网页；同时还有大量的其他开发者开发的主题
	- 隐私设置，网站能够设置密码，控制访问人员，可能用于企业内部文档管理
	- 有 SEO 加持和移动平台适配，流量可能会更大一些

- 劣势
	- 每个月需要支付 8 美元，由于个人网站的**流量相当小**需要长期持有才会有明显的收益，因此这笔支出不是小数目，这是官方发布服务的**致命缺陷**
	- 停止付费之后网页将不能被访问
	- 国内的服务支持不佳，流量受限

总结：官方发布适合**资金充裕**，并且对于网站的自定义开发没有那么高需求的用户。

#### Digital Garden

推荐指数：❤️‍🔥❤️‍🔥❤️‍🔥🩶🩶

##### 介绍

[Digital Garden](https://github.com/oleeskild/Obsidian-Digital-Garden?tab=readme-ov-file) 是一款 Obsidian 插件，可以将笔记导出为网页并托管在 GitHub 上，然后再使用 [Vercel](https://vercel.com/) 或者 [Netlify](https://www.netlify.com/) 进行网页的发布。具体操作教程见 [Digital Garden教程](https://forum-zh.obsidian.md/t/topic/19256)

这里是几个样例网站：

- [Digital Garden](https://dg-docs.ole.dev/) 官方样例
- [Aaron Youn](https://ajy.co/) 民间自制
- [John's Digital Galaxy](https://notes.johnmavrick.com/Digital+Galaxy/Welcome+%F0%9F%8C%8C) 非常丰富的内容，可以详细展示所有 Digital Garden 涉及的特性，特别是展示性链接循环嵌套

##### 评述

- 优点
	- 功能支持较为全面
	- 支持 Obsidian 的主题迁移

- 缺点
	- 对于中文路径不友好
	- 网页界面自定义需要直接处理网页源码，即 `HTML` `JavsScript` `CSS`，并且默认配套的界面不太好看

总结：工作流程非常的简单，功能支持相当的全面，作为插件嵌入 Obsidian 更加轻便。虽然界面美化需要花些功夫，但如果对于界面美观与否并不在意，可以直接上手。

#### Perlite

推荐指数：❤️‍🔥❤️‍🔥🩶🩶🩶

##### 介绍

[Perlite](https://github.com/secure-77/Perlite) 是一款网页版 Obsidian 文件阅读器，是 Obsidian 官方发布服务的开源平替。[Obsidian Publish的开源替代品Perlite](https://mp.weixin.qq.com/s/th_kStNvJYZvT0F5iTC2Xg) 这是微信公众号上的一个教程文本。

这款开源平替最大的特点就是：其网页 UI 几乎和 Obsidian 的界面完全相同，提供近乎原生的浏览服务。
##### 评述

- 优点
	- 很好地支持几乎所有的 Obsidian 的功能
	- 原生经典界面，给用户提供熟悉感

- 缺点
	- 不算是一个博客页面，官方对于项目的定位也确实不是博客网页而是一个“文件阅读器”
	- 需要使用 Docker，启动缓慢，不如插件的那样的轻巧简洁的体验感

总结：Perlite 的定位决定了其并不适合**直接用作**展示性的博客页面，其界面实在是有点单调，对于访客的视觉吸引力其实并不强。更适合于作为一款网页版的 Obsidian，在上面进行文件的编辑可以更加专注，效果也更好

#### jekyll+Netlify+GitHub Pages

推荐指数：❤️‍🔥❤️‍🔥🩶🩶🩶

##### 介绍

方法流程来源于 [obsidian 目前最完美的免费发布方案 渐进式教程](https://publish.obsidian.md/chinesehelp/01+2021%E6%96%B0%E6%95%99%E7%A8%8B/obsidian+%E7%9B%AE%E5%89%8D%E6%9C%80%E5%AE%8C%E7%BE%8E%E7%9A%84%E5%85%8D%E8%B4%B9%E5%8F%91%E5%B8%83%E6%96%B9%E6%A1%88+%E6%B8%90%E8%BF%9B%E5%BC%8F%E6%95%99%E7%A8%8B+by+oldwinter)，教程内容很详细，既有基本的对比评价，也有详细的指导教程

这是作者构建的例子网站：[oldwinterの数字花园](https://notes.oldwinter.top/)

##### 评述

- 优点
	- 配置简单
	- 允许高度自定义

- 缺点
	- 部分 Obsidian 特色语法不支持，比如 callout 语法
	- 不支持暗色模式
	- 不支持搜索

总结：整体上是一个非常好的 Obsidian 转换为博客网页的实践，但是因为部分核心功能缺失因此并不适合想要完整的网页体验的创作者

#### TiddlyWiki

推荐指数：❤️‍🔥🩶🩶🩶🩶

##### 介绍

[TiddlyWiki](https://tiddlywiki.com/) 是一个历史悠久的笔记框架，至今依然有很强的生命力，许多开发者活跃在这个领域中。国内也有相关站点可以供访问：[太微舞](https://bramchen.github.io/tw5-docs/zh-Hans/)，以及配套教程：[太微中文教程]([https://tw-cn.netlify.app](https://sspai.com/link?target=https%3A%2F%2Ftw-cn.netlify.app%2F) /)；国内的开发者在近年推出的衍生版本 [TidGi（太记）](https://github.com/tiddly-gittly/TidGi-Desktop/releases);

[利用Tiddlywiki发布Obsidian库](https://publish.obsidian.md/chinesehelp/%E5%88%A9%E7%94%A8Tiddlywiki%E5%8F%91%E5%B8%83Obsidian%E5%BA%93%EF%BC%88tidgi-obsidian-manager%E6%8F%92%E4%BB%B6%E4%BB%8B%E7%BB%8D%EF%BC%89)这是一个将 Obsidian 发布到 TiddlyWiki 的流程性教程

此外还有一些散落在互联网上的介绍和样例：

- [了不起的“活笔记”系统：TiddlyWiki（太微笔记） - 少数派](https://sspai.com/post/88083)非常好的一片笔记文章，真正意义上的一文秒懂 TiddlyWiki；文章作者自己的笔记网站：[MRIWiki.cn — 磁共振百科知识太微笔记](https://mriwiki.cn/)
- [太微中文教程]([https://tw-cn.netlify.app](https://sspai.com/link?target=https%3A%2F%2Ftw-cn.netlify.app%2F) /)教程本身就是利用 TiddlyWiki 编写的，可以查看 TiddlyWiki 的使用效果

##### 评述

- 优势
	- **极致的**简洁性与轻量化，可以说没有**任何**其他的个人网页比它更简单！
	- 经过历史的筛选，拥有广泛的用户群体
	- 成熟的国内服务，不需要科学上网就能够访问

- 劣势
	- 由于极致的简洁，导致网页看上去可能有点**原始**
	- 不算完全的个人博客，其完全不迎合主流搜索引擎，直接靠搜索根本无法访问，难以获得流量（也是我撰写这篇文章之前从未听闻的原因😢）

总结：TiddlyWiki 的确是一个非常简洁和轻量化的笔记框架，这也吸引了很多的用户；但也正是因为其定位并非博客网页，所以导致用户的内容封闭在 TiddlyWiki 社区内部，甚至是封闭在创作者自己手中，无法转化为流量，并不适合有**流量**需求的创作者，而更加适合作为一个简单纯粹的笔记存储库

## 结论

在经过完整的分析之前，我并不了解**笔记转博客**这个领域内部的实际情况，因此萌生了想做一个简化插件项目的想法💡

但是经过系统性的调查研究，我必须承认 Quartz 确实是这个领域内出类拔萃的项目，无论从 Obsidian 语法适配性、配置流程便捷程度、前端配套界面美观程度、前端界面自定义便捷程度、后端撰写博客便捷程度等等各种方面，几乎都没有上升空间了。

因此也就不需要我再去启动一个项目来做重复的事情了。在这里向所有**笔记转博客**领域内的相关项目的开发团队致敬🫡不管是否在文章中提及。

**项目没有优劣之分，只有合适与否**，这篇文章中涉及的评价标准都是围绕**是否符合博客网页要求**进行的，因此可能有些项目并不适合做博客网页，但是这丝毫不影响项目本身的价值。

向开源先锋致敬🫡🫡🫡
