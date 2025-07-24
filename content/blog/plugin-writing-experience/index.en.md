---
title: An experience of writing plugin
weight: -30
draft: false
description: An experience of writing plugin
slug: plugin-writing-experience
language: en
tags:
  - Experience
series:
  - Casual essay
series_order: 1
date: 2025-01-06
lastmod: 2025-06-21
authors:
  - Morethan
---

{{< lead >}}   A Reflection on Writing a Plugin and What I Learned from the Experience.   {{< /lead >}}

### The Beginning

It all started with my blog website. I stumbled upon an article on WeChat about building a blog with `Hugo`, and since I wanted to revamp my old, simple site, I decided to give it a try. My original site was extremely rudimentary, and the whole writing process involved jumping between `HTML`, `JS`, and `CSS` in a rather awkward manner. On top of that, I had always admired the [blog](https://lilianweng.github.io/) of a great tech guru, `Lilian Weng`, which was also built with `Hugo`. This further strengthened my resolve to change my site's underlying platform.

So, I quickly started diving into `Hugo`.

To my surprise, the results were extraordinary! My old webpage took me nearly a month to build, but with `Hugo`, I was able to finish everything in less than half a day. What shocked me even more was that `Hugo`, a program written in `Go`, didn’t require users to set up a `Go` environment! 😮

At the same time, I discovered an incredibly well-documented `Hugo` theme—[Blowfish](https://blowfish.page/). This was by far the most detailed documentation I had ever seen for any project, bar none (๑•̀ㅂ•́)و✧.

With `Hugo` and `Blowfish` working in tandem, my small site quickly took shape. Of course, I’m not great at designing, so I just used the default layout from `Blowfish`, as I felt any changes would ruin the beauty of the page.

To be honest, after all this work, I didn’t have any strong emotional reactions, except for deep respect for the coding skills of the authors of `Hugo` and `Blowfish`.

That was until I wanted to upload the massive amount of notes I had in `Obsidian` to my new blog.

### The Bitter Taste of Originality

I soon realized that there wasn’t a plugin available to directly convert the format of my `Obsidian` notes to fit the `Blowfish` theme. Fueled by the earlier "pleasant experience," I decided to write a [plugin](https://github.com/morethan987/Hugo-Blowfish-Exporter) myself! (😄 Although, I would soon stop laughing 😢)

The rest of the experience wasn’t anything particularly exciting—just endless switching between webpages, searching through API documentation, and never-ending conversations with AI bots. After countless revisions, I finally ended up with something exceedingly simple: a plugin that identifies specific patterns in documents and performs content replacement.

It was quite laughable. Compared to the few hours it took to set up the website, the nearly forty hours I spent writing that plugin felt almost negligible. At one point, I seriously considered just deleting my few hundred lines of code.

Yes, such a simple plugin drained me mentally and physically. I truly tasted the **bitterness of originality**.

Now, looking back at `Hugo` and `Blowfish`, I feel deeply **shocked** by their complexity and the effort required to implement all of those features. If they were getting paid for this work, I could at least understand the level of effort involved. But they were both open-source, relying entirely on user goodwill and appreciation.

I saw the last update of the `Blowfish` author’s blog, which was in March 2024, and I fell into deep thought.

### Sentiments and Idealism

I imagine that the author of `Blowfish` must have paused the development of the theme for some reason—perhaps due to life circumstances. After all, this project didn’t bring in much real income.

Suddenly, I remembered the changes I had noticed before—those GitHub profiles, once full of green squares, gradually becoming sparse, and eventually disappearing. Beneath this peaceful change, there might be a shift in someone's life. Whether it's because of busy work or the gradual fading of motivation, the original passionate drive eventually drowns in silence. I can't stop this from happening, but I understand the reasons behind it.

**Open-source is driven by passion, but passion doesn’t pay the bills. People need to live in the present.**

I recalled a tech YouTuber, `Ma Nong Gao Tian`, a core Python developer who humorously complained about the harsh realities of open-source life. His prematurely graying hair made me feel a pang of empathy—he had spent most of his life writing code and yet found himself out of work, surviving on a few extra bucks from his videos.

### In Conclusion

As the saying goes, "Eight or nine times out of ten, things don't go as planned in life." I looked at the results of my more than 40 hours of work and shook my head with a smile.

After writing this, I’m off to bed. It’s now 1:48 AM on January 6, 2025, and I still haven’t reviewed for my English final exam tomorrow.

Looking at this blog again, I laughed and shook my head.

Such is life.

---

### Continuation One

It's now June 21, 2025, in the early morning. After more than five months, I've reopened this article. Reading these strange words that no one sees gives me a subtle feeling.

In these five months, I started developing an open-source project for GitHub-based calendar management. Since I have the habit of writing blogs and frequently use my own plugins, I’ve iterated on my little plugin multiple times: from initially only being able to perform some formatting conversions, it now integrates git operations, translation APIs, dual-system support for Windows and Linux, and fully implements the entire Obsidian blog publishing process.

Now, all my blog publishing operations can be done within Obsidian: exporting text, differential translation, automatic export, git diff content verification, and git push.

Looking back now, I'm quite moved. What initially seemed like tiring work eventually became manageable. Proper code segmentation and isolation in the early stages made adding new features easier and facilitated AI-assisted code development.

Although I've invested a lot of effort into this plugin, there are still some benefits: it makes my blog writing process very easy and helps me maintain the habit of regular writing.

To be honest, I don't really know what purpose these words serve🤔 But based on the experiences of others, it seems beneficial. And I myself need these words to organize my thoughts, so I enjoy doing it.

A few days ago, I read a WeChat public account article: [Alibaba Employee Posts Ten-Thousand-Word Resignation Post on the Internal Network](https://mp.weixin.qq.com/s/IvPKglDR-KCE6guntrPTmw)

After reading it, besides lamenting the restlessness of the times, there wasn't much else to say. After all, this kind of issue isn't something I should be pondering solutions for, nor do I dare to think about any solutions. All I can do now is invest a bit in long-term goals during the breaks while pursuing short-term ones.

Under the heavy pressure of life, I try to make a little room so that sunlight can shine through.
