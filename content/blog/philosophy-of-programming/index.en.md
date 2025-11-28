---
title: The Philosophy of Programming
weight: -115
draft: true
description: Reflections on programming philosophy drawn from real-world development experience
slug: philosophy-of-programming
language: en
tags:
  - Experience
  - EngineeringPractice
series:
  - Casual essay
series_order: 2
date: 2025-11-28
lastmod: 2025-11-28
authors:
  - Morethan
---

{{< lead >}}   Programming is the serialization of a local world.   {{< /lead >}}

## Preface

I have been working with computers for about two years now and have gone through a fair amount of hands-on programming practice. In the beginning, programming felt mostly like a tool—much like using a smartphone in daily life: ordinary, utilitarian, and taken for granted. But programming is fundamentally different from using a phone. It is intellectually demanding labor, and over time, the way you think begins to change. Gradually, I found myself being influenced by certain philosophies embedded in programming languages. I began to understand the subtle elegance behind different paradigms, and this is exactly where programming becomes fascinating: _how can I describe something so complex?_ Or more precisely, _how can I describe it in such an elegant way?_

This article is not meant to define "philosophy" through abstract language. Words alone are not philosophy. Instead, I want to record those moments when a simple idea suddenly makes you think, “Oh, so that’s how it works.” These are the moments where you can feel the intentions and craftsmanship of language designers flowing through what otherwise seems like ordinary keystrokes.

## React

React is not a programming language, but rather a programming paradigm. Much like object-oriented programming, it represents a way of thinking about code rather than a specific language. Its concrete implementation still relies on the TSX language. Personally, I think React defines a method for describing reactive user interfaces. I chose it as the first example because it was the spark that led me to start thinking seriously about programming philosophy.

Why is that? Let's imagine an ideal modern website. A truly modern interface is not just static text—it is rich in elements, carefully designed visuals, and an overall layout that feels comfortable and natural. It also includes smooth interactions: backgrounds that subtly react to cursor movement, buttons that ripple like water when clicked, and animations that feel almost alive.

The vision is beautiful. The problem is expressing it to a computer. Describing such behavior in precise, logical steps is surprisingly difficult. This is where React offers a powerful abstraction. By using TSX and following a simple set of principles, you can express highly complex UI behavior in a relatively concise and structured way.

The official documentation includes a section called [Thinking in React](https://zh-hans.react.dev/learn/thinking-in-react), which walks through a concrete example and explains how complex React applications are actually constructed.

To quote the documentation: React changes the way you think about visual design and application structure. When building a user interface with React, you first break it down into individual **components**, and then connect them so that data flows through the system.

After approaching a large React codebase with the wrong way of thinking, revisiting this short guide may finally make everything _click_. And perhaps that is the point of philosophy in programming after all.
