---
title: Reflections on Moravec's Paradox  
weight: -25  
draft: true  
description: Reflections on Moravec's Paradox  
slug: moravecs-paradox  
tags:  
  - AI  
  - Musings  
series:  
  - AI Musings  
series_order: 1  
date: 2025-01-03  
lastmod: 2025-01-20  
authors:  
  - Morethan  
---  

## Moravec's Paradox  

> High-level reasoning requires relatively little computational power, whereas low-level perception and motor skills demand enormous computational resources.  

This phenomenon implies that computers and robots find it relatively easy to handle complex logic and mathematical problems but struggle with basic perceptual and motor tasks such as walking, grasping, and visual recognition.  

### Analysis  

Low-level perception and motor skills feel subjectively simple but are computationally complex from a *computational theory* perspective.  

**High-level tasks**: The term itself is highly subjective—"high-level" essentially equates to "difficult for humans." However, these so-called difficult tasks are mostly **polynomial-time solvable**, meaning that, by their very nature, high-level tasks do not involve inherently complex problems.  

**Low-level tasks**: The problems they encompass are mostly NP-hard or even PSPACE-hard, indicating that these problems are fundamentally complex.  

Humans effortlessly solve low-level tasks due to **evolutionary optimization over time**: it took eons for humans to develop the ability to efficiently and accurately perform low-level tasks, whereas acquiring high-level reasoning abilities took comparatively little time.  

This paradoxical phenomenon seems to stem from the generalizability of knowledge. Some knowledge is highly compressible, while other forms are not. A more precise concept here is *computational reducibility*.  

A simple example: exams. Logically, all exam questions are derived from basic knowledge through reasoning, right? In theory, exams should be entirely solvable through logical deduction. Yet, anyone who has taken an exam knows that the process from reading a question to forming a solution is not particularly "logical." In fact, it often feels like **there’s no clear technique**—just an intuitive sense of how to approach the problem.  

#### Two Types of Knowledge  

Knowledge that can be derived from existing information through finite symbolic logic expressions. Its characteristics include precision, high generalizability, well-defined problem boundaries, and the ability to clearly specify given conditions, methods, and outcomes. Under explicit conditions, it can accurately predict results (with no room for "probability" within the problem's scope).  

For problems where outcomes cannot be derived from given conditions through precise logical reasoning, we rely on statistical experimentation to extract valuable patterns. These patterns are characterized by extensive trial-and-error, poorly defined problem boundaries, and difficulty in obtaining necessary conditions. Despite these challenges, they forcibly correlate conditions with outcomes, producing rules that are volatile, uncertain, and locally correct.  

#### Proportion of the Two Types  

**Clearly generalizable knowledge is far less abundant than non-generalizable knowledge.** In a sense, generalizable knowledge is a special case of non-generalizable knowledge.  

#### The Trend of Non-Generalizable Knowledge Transforming into Generalizable Knowledge

The nature of non-generalizable knowledge determines the difficulty of acquiring it (massive experimentation consumes vast energy, an unavoidable step). Operating non-generalizable knowledge is highly energy-intensive (ungeneralizable knowledge requires significant resources to maintain). Moreover, non-generalizable knowledge struggles to transcend the boundaries of individual human lifespans (it tends to vanish with the individual, as it cannot be easily recorded or transmitted—though machine intelligence may differ fundamentally here).  

Human individuals have limited energy and cannot rely solely on non-generalizable knowledge to interact with the world. Thus, there is a tendency for non-generalizable knowledge to transform into generalizable knowledge. Although this process is arduous and energy-intensive for individuals (being itself a form of non-generalizable knowledge), it saves enormous energy at the collective human level.  

#### Acceptable Power Input Determines the Upper Limit of Intelligence

Here, we might propose another criterion for classifying intelligence levels: the **power input an individual can accept**. The higher the level, the more non-generalizable knowledge an individual can master. Since energy is always finite (humans may be limited to Earth's resources, whereas machine intelligence could harness stellar-scale energy), there will always be some degree of knowledge generalization. However, due to the inherent nature of non-generalizable knowledge, the generalizable knowledge of higher-level intelligence may remain non-generalizable to lower-level intelligence. **The generalizability of knowledge is relative.**  

From this perspective, **acceptable power input is crucial for an intelligent system**. Alternatively, **we might artificially limit the power consumption of machine intelligence to observe the transformation of non-generalizable knowledge into generalizable knowledge.**