---
title: Analysis Report on the Note-to-Blog Project
weight: -40
draft: false
description: An analytical report on the note-to-blog project  
slug: note-to-blog-report
tags:
  - Hugo
  - 博客
  - 报告
series:
  - Operation
series_order: 4
date: 2025-01-10
lastmod: 2025-01-12
authors:
  - Morethan
---

{{< lead >}}  
Familiarizing oneself with the project analysis process, this is a practical analysis report.  
{{< /lead >}}

After I had completed my [small plugin]({{< ref "/blog/plugin-writing-experience/" >}}), I realized that there doesn't seem to be a one-click solution in the **note-to-blog** field. So, I wrote this article to analyze whether it would make sense to create a new project to fill this gap.

**There is no such thing as a "better" or "worse" project; there is only whether it is suitable or not.** The evaluation criteria in this article are based on **whether they meet the requirements of a blog website**. Therefore, some projects may not be suitable for creating a blog site, but this does not diminish their value.

## Field Definition

Before we begin the analysis, it's important to have a clear understanding of the **note-to-blog** field.

- **Notes:** In this context, "notes" specifically refers to the notes in Obsidian, which use Obsidian Flavored Markdown. This adds certain complexities to the process of converting notes to a webpage.
    
- **Blog:** A blog, by definition, is a means to gain traffic. Creators spend time writing notes, and then using conversion software, generate beautifully formatted, feature-rich blog websites.
    

Evaluation criteria are as follows:

- **Privacy:**
    
    - Does it run locally?
    - Is it open-source?
- **Usability:**
    
    - How well does it adapt to Obsidian's syntax?
    - How complex is the service deployment?
    - How detailed is the documentation?
    - How easy is it to customize settings?
- **Web Functionality:**
    
    - Does the default web template include all essential functions (search, day/night mode, etc.)?
    - How visually appealing is the default webpage?
    - Does it support SEO?
    - How smoothly does it convert Obsidian's native syntax (for example, are there untranslatable code blocks in internal links, or does it discard some Obsidian syntax features)?

## Project Overview

In my vision, this project would be an Obsidian plugin that seamlessly exports Obsidian notes into Hugo blog webpages, supporting all of Obsidian's basic core features.

Result: It greatly reduces the cost of creating a blog webpage, allowing anyone who can use Obsidian to have their own blog.

## Market and User Feasibility Analysis

### Market Demand Analysis

#### Overview

- **Basic Needs:** The demand to build a personal website and continuously produce content, including for self-improvement, self-expression, and creating a unique and comprehensive personal skill showcase (for **corporate recruitment**), etc.
    
- **Target User Group:** Heavy Obsidian users who want to share notes; knowledge creators who want to build a personal blog but have abandoned the idea due to technical difficulty.
    

#### Relevant Data

- **Flowershow:** As of October 2024, the plugin had been downloaded 3,355 times; by January 2025, this number had risen to 4,594, while the most downloaded plugin had reached 3,211,992 downloads.
- **Quartz:** As of January 2025, Quartz had accumulated 7.7k stars on GitHub.

### Existing Solutions

#### Quartz (Hugo)

Recommendation: ❤️‍🔥❤️‍🔥❤️‍🔥❤️‍🔥❤️‍🔥

##### Introduction

[Quartz](https://github.com/jackyzha0/quartz) is a toolset that converts Obsidian notes into web pages. The latest version, `v4`, has undergone a complete rewrite compared to `v3`, removing its dependency on `Hugo` and optimizing the user customization experience. The `v4` version is now primarily built with `TypeScript`, and the original `Hugo` templates have been replaced with `JSX`.

As a result, Quartz in its current form is almost entirely disconnected from `Hugo`. However, much of the information available in Internet still advertises Quartz as being built on top of `Hugo`.

Official example website: [Welcome to Quartz 4](https://quartz.jzhao.xyz/)

##### Review

- **Pros:**
    
    - Extremely complete functionality
    - The only toolset that successfully handles display wiki links
    - Detailed documentation
- **Cons:**
    
    - Virtually no drawbacks, but one notable limitation is the lack of Chinese documentation.

Summary: An excellent project, where the styles displayed in Obsidian are the same as those displayed on the webpage. It has garnered the most stars on GitHub among all available solutions.

#### Flowershow

Recommendation: ❤️‍🔥❤️‍🔥❤️‍🔥❤️‍🔥🩶

##### Introduction

[Flowershow](https://flowershow.app/) is an overall publishing service based on Obsidian, which can convert your Obsidian notes into an online digital garden website with directory structure. [Vercel](https://vercel.com/) is a cloud service for front-end deployment, enabling serverless front-end deployment via GitHub. Each content submission triggers automatic deployment. For domestic users, [Netlify](https://www.netlify.com/) is an alternative.

Subjectively, the development team behind Flowershow is very passionate and mission-driven. Their core philosophies are detailed in their [About page](https://flowershow.app/about).

Objectively, Flowershow's positioning as a blog webpage generation platform based on Obsidian is spot-on, and the final result is very good both from a front-end and back-end perspective.

##### Review

- **Pros:**
    
    - Clear positioning and a straightforward workflow
    - Comprehensive feature support
    - Professional team behind maintenance and operations
    - Highly customizable, suitable for creators who enjoy personalizing their setup
- **Cons (as of January 2025):**
    
    - Some Obsidian features are not handled, such as display wiki links. At least this section is omitted in the documentation.
    - A reverse link feature is mentioned on the homepage, but it’s unclear in the site’s details.

Summary: Overall, the project is well done, but some details still need improvement. This solution is suitable for creators who don't require high support for Obsidian syntax.

#### Official Publish

Recommendation: ❤️‍🔥❤️‍🔥❤️‍🔥🩶🩶

##### Introduction

Examples of websites using Obsidian's official publishing service:

- [Obsidian Chinese Tutorial](https://publish.obsidian.md/chinesehelp/01+2021%E6%96%B0%E6%95%99%E7%A8%8B/2021%E5%B9%B4%E6%96%B0%E6%95%99%E7%A8%8B) - A Chinese tutorial website, using the official Obsidian publishing service.
- [Digital 3D Garden](https://stephanlevin.garden/Welcome) - Deep front-end customizations.
- [Mister Chad](https://mister-chad.com/welcome) - A simple, neat site with rich content.
- [Discrete Structures for Computer Science](https://publish.obsidian.md/discretecs/START+HERE) - Official simple style.

##### Review

- **Pros:**
    
    - The official publish service offers top-notch support for Obsidian’s internal representations, ensuring all Obsidian features are correctly displayed on the webpage.
    - Continuous maintenance ensures quick adaptation to updates from Obsidian.
    - Highly customizable settings for users with coding experience, and a wide range of themes from other developers.
    - Privacy settings, password protection, and access control for internal document management.
    - SEO support and mobile platform adaptation for greater potential traffic.
- **Cons:**
    
    - Costs $8 per month. Since personal websites typically have very little traffic initially, this can become a significant expense over time. This is the **major drawback** of the official service.
    - If you stop paying, the website becomes inaccessible.
    - Limited support in certain regions, with traffic constraints in China.

Summary: The official service is suitable for users with **sufficient funds** and moderate customization needs.

#### Digital Garden

Recommendation: ❤️‍🔥❤️‍🔥❤️‍🔥🩶🩶

##### Introduction

[Digital Garden](https://github.com/oleeskild/Obsidian-Digital-Garden?tab=readme-ov-file) is an Obsidian plugin that exports notes as webpages and hosts them on GitHub, with deployment via [Vercel](https://vercel.com/) or [Netlify](https://www.netlify.com/).

Example sites:

- [Digital Garden](https://dg-docs.ole.dev/) - Official example
- [Aaron Youn](https://ajy.co/) - Created by an individual user
- [John's Digital Galaxy](https://notes.johnmavrick.com/Digital+Galaxy/Welcome+%F0%9F%8C%8C) - Rich content showcasing Digital Garden’s features, including display links.

##### Review

- **Pros:**
    
    - Comprehensive feature support
    - Supports Obsidian theme migration
- **Cons:**
    
    - Not friendly with Chinese paths
    - Web interface customization requires direct handling of source code (`HTML`, `JavaScript`, `CSS`), and the default interface is not very visually appealing.

Summary: The workflow is simple, and the feature support is extensive. However, the interface requires effort to improve, and creators who care less about aesthetics can jump straight into using it.

#### Perlite

Recommendation: ❤️‍🔥❤️‍🔥🩶🩶🩶

##### Introduction

[Perlite](https://github.com/secure-77/Perlite) is an open-source alternative to Obsidian's official publishing service, providing a browser

-based file reader with an interface nearly identical to Obsidian’s.

##### Review

- **Pros:**
    
    - Supports almost all Obsidian features.
    - Classic native interface, offering a familiar experience for users.
- **Cons:**
    
    - It is not a blog page but a "file reader" instead.
    - Requires Docker, which can result in slower startup times compared to the simplicity of a plugin experience.

Summary: Perlite is best suited for those who need a browser-based Obsidian experience, rather than as a public-facing blog.

#### Jekyll + Netlify + GitHub Pages

Recommendation: ❤️‍🔥❤️‍🔥🩶🩶🩶

##### Introduction

This method is derived from [obsidian's most perfect free publishing solution](https://publish.obsidian.md/chinesehelp/01+2021%E6%96%B0%E6%95%99%E7%A8%8B/obsidian+%E7%9B%AE%E5%89%8D%E6%9C%80%E5%AE%8C%E7%BE%8E%E7%9A%84%E5%85%8D%E8%B4%B9%E5%8F%91%E5%B8%83%E6%96%B9%E6%A1%88+%E6%B8%90%E8%BF%9B%E5%BC%8F%E6%95%99%E7%A8%8B+by+oldwinter).

Example website by the author: [oldwinter’s Digital Garden](https://notes.oldwinter.top/)

##### Review

- **Pros:**
    
    - Simple configuration
    - Highly customizable
- **Cons:**
    
    - Does not support certain Obsidian features like callout syntax
    - No dark mode support
    - No search functionality

Summary: A good solution for converting Obsidian to a blog, but missing some core features, making it unsuitable for creators seeking a complete web experience.

#### TiddlyWiki

Recommendation: ❤️‍🔥🩶🩶🩶🩶

##### Introduction

[TiddlyWiki](https://tiddlywiki.com/) is an older note-taking framework that remains very active today, with developers continuously enhancing it.

##### Review

- **Pros:**
    
    - Extremely simple and lightweight
    - Widely used with a strong user base
    - Domestic services available with no need for VPNs
- **Cons:**
    
    - The simplicity might result in a somewhat **primitive** interface.
    - Not a full-fledged personal blog; lacks SEO and is difficult to access via search engines, limiting traffic potential.

Summary: TiddlyWiki is ideal for personal note storage but not for creators seeking a blog that attracts traffic.

## Conclusion

Before conducting a thorough analysis, I was unaware of the actual landscape in the **note-to-blog** field, which led me to consider creating a simplified plugin. 💡

However, after systematic research, I must admit that Quartz stands out as the best project in this space. Whether it's the adaptation to Obsidian's syntax, ease of configuration, front-end aesthetics, customization options, or backend blog creation, there is very little room for improvement.

Thus, there is no need for me to initiate a project to duplicate what’s already been done. I salute all the teams involved in the **note-to-blog** field, whether mentioned in this article or not. 🫡

**There are no "better" or "worse" projects, only those that are suitable or not.** The evaluation criteria in this article focus on **whether the solution meets the requirements for a blog webpage**, and thus, some projects may not be ideal for blogging but still offer great value.

Saluting open-source pioneers! 🫡🫡🫡