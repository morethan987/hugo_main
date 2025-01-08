---
title: Hugo博客搭建
weight: -35
draft: false
description: Hugo博客搭建
slug: hugo-blog
tags:
  - Hugo
  - 博客
series:
  - 技术流程
series_order: 3
date: 2025-01-07
authors:
  - Morethan
---

{{< lead >}}
博客网站折腾日志，从手搓到 Hugo 的曲折经历。
{{< /lead >}}

## 为什么 Hugo
最初也只是因为偶然间听说了 Hugo 可以做网页，并且听说编译静态页面非常高效，于是才去搜集的相关资料——据说 Hugo 是**世界上最快**的静态页面生成器，官方网站也是这么写的。

当然，口说无凭，下面就是我初次本地编译运行 Hugo 得到的输出，即在完全没有 `public` 文件夹的情况下的输出：

|                  | ZH-CN | EN  |
| ---------------- | ----- | --- |
| Pages            | 53    | 51  |
| Paginator pages  | 0     | 0   |
| Non-page files   | 13    | 13  |
| Static files     | 7     | 7   |
| Processed images | 3     | 0   |
| Aliases          | 18    | 17  |
| Cleaned          | 0     | 0   |

Built in 872 ms

中英文总共 104 个网页页面花费时间 0.872 秒，并且包含构建本地服务的时间，这个速度确实没什么好挑剔的了。并且本地的服务能够实时监听源代码的改动并进行**增量重构**，这个增量重构的时间取决于改动的多少，一般在 0.03 秒左右。

{{< alert icon="pencil" >}}
我并没有用别的网页生成工具搭建过博客，因此无法给出其他生成器的实际生成速度
{{< /alert >}}
## 参考引用
下面是我在搭建博客的过程中所用到的所有资源链接：

- [莱特雷-letere](https://letere-gzj.github.io/hugo-stack/) 这是一位博主的网页，也是用 Hugo 搭建的，里面同时也有很多其他的网页工具的教程；这是他在 B 站上发布的系列[视频教程](https://www.bilibili.com/video/BV1bovfeaEtQ/?spm_id_from=333.337.search-card.all.click&vd_source=38d0addc11facdcdfe9d401e43b75680)
- [Blowfish](https://blowfish.page/zh-cn/) 这是我使用的一款 Hugo 主题，文档写的相当的好，真的没见过如此耐心的作者。
- [Hugo官方网页](https://gohugo.io/)
- [Hugo Themes](https://themes.gohugo.io/) 

## 部署全流程
### 搭建 Hugo 环境
这一部分在那位博主的[网页](https://letere-gzj.github.io/hugo-stack/)和[视频教程](https://www.bilibili.com/video/BV1bovfeaEtQ/?spm_id_from=333.337.search-card.all.click&vd_source=38d0addc11facdcdfe9d401e43b75680)中都有非常详细的讲解，不喜欢看文字的可以去看视频😝

说实话，Hugo 的环境搭建真的是我见过的最简单的，没有之一。你只需要到 [Hugo官方网页](https://gohugo.io/)去把 Hugo 下载下来，然后存放到合适的文件夹里面，然后解压就完成了，解压之后的文件夹内也只有一个 `hugo.exe` 文件，简直不要太简单。

{{< alert icon="pencil" >}}
Hugo 真的是太方便了，我曾经尝试过 Hexo 但是 Node.js 的配置就把我拒之门外了，到现在我也不知道为什么会编译报错😢
{{< /alert >}}
稍微需要那么一点点🤏难度的工作其实就是把 `hugo.exe` 所在的目录添加到环境变量里面去。

### 生成模版系统
在 `hugo.exe` 的目录中打开终端，然后输入命令 `hugo new site your-site-name`，然后就可以看到一个新的文件夹📂出现在了当前文件夹里面。

**模板系统**听着很高级，其实就是在你的 `hugo.exe` 的同级目录下面建一个文件夹，但是里面的所有文件夹都有特殊的含义，不能随便改动。

| 名称          | 含义                      |
| ----------- | ----------------------- |
| `asset`     | 存放网站结构用到的图片，图标等资产       |
| `config`    | 网站配置文件夹（初始时可能没有，有些主题需要） |
| `hugo.toml` | 网站配置文件之一                |
| `content`   | 所有内容都在这里面               |
| `public`    | 是编译后生成的完整网页，一开始没有       |
| `themes`    | 存放你的网站主题                |
### 主题配置基本操作
Hugo 的网站主题很多，具体参考 [Hugo Themes](https://themes.gohugo.io/) 你可以选一款你喜欢的主题，然后下载之后就放在 `themes` 文件夹里面就行。这一部分文字描述非常抽象，见[视频教程](https://www.bilibili.com/video/BV1bovfeaEtQ/?spm_id_from=333.337.search-card.all.click&vd_source=38d0addc11facdcdfe9d401e43b75680)

这里有一个**重点**是：基本上每一个主题都会配置一个**样例网站**，一般在文件夹 `exampleSite` 里面，如果实在不想跟着网站文档自己配置，直接用这个样例网站的配置也是可以的。

每一个不同的主题基本配置完成之后都要进行个性化配置。这里我重点推荐重点推荐一下我使用的主题 [Blowfish](https://blowfish.page/zh-cn/) 相当好的一款主题，向作者致敬🫡

### Blowfish 主题
[Blowfish](https://blowfish.page/zh-cn/) 官方文档上面已经有了非常非常详细的指导文档，不再过多赘述。任何一个多余的字都是对于如此详细的指导文档的不尊重🫡

我这里简要说明一些可能出现的问题，下面的内容你可能需要仔细阅读官方文档之后才能明白其中含义🤔

- 在 `params.homepage.showRecent = false` 的情况下，为什么还会显示"最近文章"？

如果遇到这个问题，说明你跟我一样懒惰🤪直接套用了 `exampleSite` 的代码。这是因为控制主页面的接口不止这一个，还有一个在 `layouts\partials\home\custom.html` 文件中。

如果你不介意那么直接忽略就行，如果你介意（跟我一样🤪），那就把文件中的下面的代码注释掉👇

```html
<section>
  {{ partial "recent-articles-demo.html" . }}
</section>
```

- 为什么我使用 `svg` 格式的 `logo` 无法完成(日间/夜间)模式的切换？

这是我发现的一个 `bug`，已经给主题作者推送我改进的代码了，详见[代码改进](https://github.com/nunocoracao/blowfish/pull/1902)

- 为什么浏览器窗口上的小图标一直都是 `blowfish`，即使改换了 `logo` 也不行？

官方文档其实是有写的，但藏的太深了，见[局部模板(Partials) · Blowfish](https://blowfish.page/zh-cn/docs/partials/#%E7%BD%91%E7%AB%99%E5%9B%BE%E6%A0%87favicons)

说实话，官方文档写的真的好👍一套完整流程走下来竟然只有这么点不太容易理解的错误😋

## 配套插件
我日常习惯使用 [Obsidian](https://obsidian.md/) 来写文章，由于 Obsidian 的格式与 Blowfish 的格式还是有较为明显的区别，二者的格式转换非常麻烦🤔

在查询了一圈之后，发现根本就没有适合的插件！于是，我自己开发了一个插件：[Hugo-Blowfish-Exporter](https://github.com/morethan987/Hugo-Blowfish-Exporter)

虽然插件功能很简单，但是也已经覆盖了我自己绝大部分的使用功能，包括：

  - **callout**（支持所有官方的 callout 名称，需要[新增图标](https://github.com/morethan987/hugo_main/tree/main/assets/icons)）

  - **内联数学公式**（Blowfish 支持块级公式）

  - **mermaid**（支持 mermaid 图表）

  - **图片插入**（能够自动导出图片）

  - **Wiki链接导出**，提供了一套模拟 Obsidian 的**文件互联方案**，自认为很成功😁

1. 非展示性链接简单处理为网页超链接形式

2. 展示性链接较为复杂：需要更新 `Blowfish` 主题的源代码，通过 `mdimporter` 这个 `shortcode` 来进行文件注入；同时为了方便链接，每一个文件都需要设置 `slug` 属性来标记网站中存放 `markdown` 文件的文件夹

这个插件也是投入了我巨大的精力，虽然也只有几天时间🤔但是那几天还是挺累的😵‍💫
  
如果这个插件帮助了你，还请转发分享；如果你对于这个插件的功能不满意，你也可以在 GitHub 上向我提交 Issue🫡或者熟悉代码的朋友可以直接把源码拿去修改，注释很完整，代码比较规范🤗

如果你能把你亲自修改升级的代码也分享给我（在 GitHub 上提交 Pull Request）那更是万分感谢！☺️

## 写在最后

**一个博客网站的搭建只是万里长征的第一步，真正困难的还是博客内容的填充。**

正如我在[一次写插件的经历]({{< ref "/blog/plugin-writing-experience" >}})中所写，很多个人博客网站从一开始的火热到最终的沉寂可能只需要短短一年的时间。

在这个生活节奏越来越快的时代，无意义、无效率的事情大多都会向*高效*让步，曾经的初心与梦想往往会向生活妥协。我自己也早已经没有了当初的热情，行为方式上也更加的像一个**真正的成年人**。

但是我仍然是有些**不甘心**，这个网站就是一种抗争吧，我会尽力维护下去，这也是我写插件方便我更新博客的目的所在。

因此希望这篇教程所提供的内容能够帮助到正在准备搭建自己的博客网站的你，你我共勉🫡