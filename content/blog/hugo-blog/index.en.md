---
title: Hugo Blog Setup
weight: -35
draft: false
description: Hugo Blog Setup
slug: hugo-blog
tags:
  - Hugo
  - Blog
series:
  - Operation
series_order: 3
date: 2025-01-07
authors:
  - Morethan
---

{{< lead >}}  
A blog website journey, from hand-coding to Hugo, a story of twists and turns.  
{{< /lead >}}

## Why Hugo?
It all started with hearing that Hugo could generate webpages and that it was incredibly efficient at compiling static pages. I decided to dive into the research—Hugo is said to be the **fastest static site generator** in the world, as claimed on its official website.

Of course, words are just words, so here’s the output I got when I compiled and ran Hugo locally without the `public` folder at the beginning:

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

In total, compiling 104 pages (both Chinese and English) took just 0.872 seconds, including the time to build the local server. That speed is hard to criticize. And the local server can listen for changes to the source code in real time and do **incremental refactoring**, depending on the size of the change, usually around 0.03 seconds.


{{< alert icon="pencil" >}}
I haven't used other page generators for setting up blogs, so I can't compare Hugo's speed with others.
{{< /alert >}}

## References
Here are all the resources I used during the blog setup process:

- [莱特雷-letere](https://letere-gzj.github.io/hugo-stack/) This is a blogger’s site also built with Hugo. It contains a lot of tutorials on other web tools as well. The series is also available on [Bilibili video tutorial](https://www.bilibili.com/video/BV1bovfeaEtQ/?spm_id_from=333.337.search-card.all.click&vd_source=38d0addc11facdcdfe9d401e43b75680).
- [Blowfish](https://blowfish.page/zh-cn/) This is the Hugo theme I used, and the documentation is excellent. I’ve never seen such a patient author.
- [Official Hugo Website](https://gohugo.io/)
- [Hugo Themes](https://themes.gohugo.io/)

## Full Deployment Process
### Setting Up Hugo
This part is covered in detail in the [webpage](https://letere-gzj.github.io/hugo-stack/) and [video tutorial](https://www.bilibili.com/video/BV1bovfeaEtQ/?spm_id_from=333.337.search-card.all.click&vd_source=38d0addc11facdcdfe9d401e43b75680) by the blogger. If you don’t like reading text, you can follow the video tutorial. 😝

Honestly, setting up Hugo is one of the easiest setups I've ever seen, no exaggeration. You simply download Hugo from the [official website](https://gohugo.io/), place it in a folder, and unzip it. You’ll find just one file, `hugo.exe`—it’s that simple.


{{< alert icon="pencil" >}}
Hugo is really convenient. I tried Hexo before, but the Node.js setup turned me away. Even now, I have no idea why it failed to compile. 😢
{{< /alert >}}

The only slight difficulty is adding the directory containing `hugo.exe` to your environment variables.

### Creating a Template System
Open the terminal in the folder where `hugo.exe` is located and run the command `hugo new site your-site-name`. You’ll see a new folder appear in the current directory.

The **template system** sounds advanced, but it’s just a special folder structure created in the same directory as `hugo.exe`. You can’t arbitrarily modify its contents because each folder has a specific purpose.

| Name          | Purpose                      |
| ------------- | ---------------------------- |
| `asset`       | Stores images, icons, and other assets used by the website |
| `config`      | Website configuration folder (may not exist initially; some themes require it) |
| `hugo.toml`   | One of the website configuration files |
| `content`     | All content goes here         |
| `public`      | The folder containing the fully compiled website (empty initially) |
| `themes`      | Stores your website’s themes   |

### Basic Theme Configuration
Hugo has a lot of themes, and you can browse them on [Hugo Themes](https://themes.gohugo.io/). You can download the theme you like and place it in the `themes` folder. The process might sound abstract, but you can check out the [video tutorial](https://www.bilibili.com/video/BV1bovfeaEtQ/?spm_id_from=333.337.search-card.all.click&vd_source=38d0addc11facdcdfe9d401e43b75680) for more guidance.

Here’s one **important tip**: most themes come with a **sample site** located in the `exampleSite` folder. If you don’t want to configure everything from scratch, you can just use the sample configuration.

After configuring the theme, you’ll need to customize it. I highly recommend the [Blowfish](https://blowfish.page/zh-cn/) theme, which is fantastic, and I truly respect the author 🫡.

### Blowfish Theme
The [Blowfish](https://blowfish.page/zh-cn/) official documentation is incredibly detailed, so I won’t repeat it here. Any additional words would be disrespectful to such a comprehensive guide 🫡.

However, there are some issues you might encounter, and I’ll briefly mention them below. You should carefully read the official documentation to fully understand these points 🤔.

- **Why does the "Recent Articles" section still show up even if `params.homepage.showRecent = false` is set?**

If you face this issue, it’s likely because, like me, you lazily used the `exampleSite` configuration. This is because the homepage layout is controlled by more than one interface, and another one is located in the `layouts\partials\home\custom.html` file.

If you don’t mind, just ignore it. But if you care (like I did 🤪), you can comment out the following code in the file:

```html
<section>
  {{ partial "recent-articles-demo.html" . }}
</section>
```

- **Why doesn’t the logo change between day and night modes when I use an `svg` logo?**

This is a bug I discovered, and I’ve already submitted an improvement to the theme author. See [code improvement](https://github.com/nunocoracao/blowfish/pull/1902).

- **Why does the small icon in the browser window still show the `blowfish` logo even after I change the site’s logo?**

This is mentioned in the official documentation, but it’s buried deep. You can find it under [Partials Documentation for Blowfish](https://blowfish.page/zh-cn/docs/partials/#%E7%BD%91%E7%AB%99%E5%9B%BE%E6%A0%87favicons).

To be honest, the official documentation is excellent. 👍 After going through the entire process, I only encountered a few minor issues that were not easy to understand 😋.

## Plugins I Use
I prefer using [Obsidian](https://obsidian.md/) for writing articles. However, the format used by Obsidian and the one used by Blowfish is quite different, so converting between the two can be a hassle 🤔.

After searching around, I found that there weren’t any suitable plugins! So, I developed my own plugin: [Hugo-Blowfish-Exporter](https://github.com/morethan987/Hugo-Blowfish-Exporter).

While the plugin is simple, it covers almost all of my needs, including:

  - **Callouts** (supports all official callout names, with [additional icons](https://github.com/morethan987/hugo_main/tree/main/assets/icons))
  - **Inline math formulas** (Blowfish supports block-level formulas)

  - **Mermaid** (supports Mermaid diagrams)

  - **Image embedding** (automatically exports images)

  - **Wiki-style links**, with a **file interlinking system** that simulates Obsidian’s functionality. I think it works pretty well 😁.

1. The none-displayed link is simply exported as normal hyperlink in the HTML file;

2. The displayed link is more complex: change(override) the source code of Blowfish to support the file injection through the `shortcode`, `mdimporter`; every Obsidian file should includes a meta data `slug` to tag the folder that contains the target markdown file in your website repository.
3. The overriding of the theme's source code can be found in the [mdimporter](https://github.com/morethan987/hugo_main/blob/main/layouts/shortcodes/mdimporter.html) and the [stripFrontMatter](https://github.com/morethan987/hugo_main/blob/main/layouts/partials/stripFrontMatter.html) used to remove metadata from the injected files' headers. For overriding the directory, refer to the configuration on GitHub.

I put a lot of effort into this plugin, even though it only took a few days 🤔. But those few days were quite exhausting 😵‍💫.

If this plugin helps you, feel free to share it! If you’re not happy with the functionality, you can submit an issue on GitHub 🫡. If you're familiar with the code, you can modify it directly; the code is well-commented and quite standard 🤗.

And if you modify and upgrade the code, I’d be very grateful if you share your changes with me (via a pull request on GitHub)! ☺️

## Final Thoughts

**Setting up a blog site is just the first step in a long journey; the real challenge is filling it with content.**

As I mentioned in [An experience of writing plugins]({{< ref "/blog/plugin-writing-experience" >}}), many personal blogs fade into obscurity in as little as a year, from the initial burst of excitement to the eventual silence.

In this fast-paced world, most meaningless and inefficient things are eventually replaced by **efficiency**, and the original enthusiasm and dreams often compromise with reality. I too no longer have the passion I once had, and my actions have become more like those of a **real adult**.

But there's still a bit of **unwillingness** in me. This website is a form of resistance, and I’ll do my best to maintain it. That’s also why I developed the plugin—to make updating the blog easier.

I hope this tutorial helps anyone planning to set up their own blog. Let’s keep moving forward, together 🫡.
