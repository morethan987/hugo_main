---
title: CUMCM 2024 Summary
weight: -20
draft: false
description: A summary and reflection on CUMCM 2024
slug: cumcm2024
language: en
tags:
  - CUMCM
  - math
series:
  - MathModel
series_order: 2
date: 2024-09-12
lastmod: 2025-01-15
authors:
  - Morethan
---
{{< katex >}}

## Preface

This article is primarily a review and summary of the entire process of CUMCM 2024.

Our team was formed in the winter of 2023, and CUMCM 2024 was our first participation in the "Mathematical Modeling" competition. After numerous mock contests, we finally made it to the national competition. After submitting the final paper, we won the first prize at the provincial level and were recommended for the first prize at the national level, ultimately receiving the second prize at the national level.

There were moments of excitement and surprise, as well as disappointment; we must have done some things right in the competition, which is why we won a national award in our first attempt; but there are definitely shortcomings, after all, there must be a reason for going from "recommended for the first prize at the national level" to "second prize at the national level".

In short, this experience is truly unforgettable, and it is even more worth summarizing and learning from the experience to prepare for next year's competition.


{{< alert icon="circle-info" cardColor="#b0c4de" textColor="#333333" >}}
CUMCM stands for Chinese Undergraduate Mathematical Contest in Modeling; it is commonly referred to as the "National Mathematical Modeling Competition".
{{< /alert >}}

## Terminology Explanation

| Term       | Explanation                                        |
| --------- | -------------------------------------------------- |
| Computational System | The traditional modeling process, encapsulating a large function |
| Optimization System | A system used to optimize the adjustable parameters in the computational system to generate the best parameter configuration |
| Computational Flow | The process of handling input data in the computational system |
| Computational Flow Node | A key intermediate step in the workflow |
| Optimization Flow | The main logic of the optimization system |
| Main Body of the Paper | Includes the abstract, restatement, descriptions of computational and optimization flows, results presentation and analysis, that is, all content before the conclusion of the paper |
| Conclusion of the Paper | Includes sensitivity analysis and model extension |

## Objective Conditions

### Task Division

Although there were many topics to choose from for the competition, our group chose to **focus** on optimization problems, which is Topic A.

- Me: Modeling + Coding + Part of Paper Writing
- CL: Modeling + Paper Writing + Part of Coding
- HWJ: Paper Beautification

### Workflow

The coding part of the entire Topic A can be roughly divided into two systems:
- Computational System:
	- Function: Accept input data and parameters, return the required results
	- Nature: Directly determined by the problem, different topics have different computational systems, which need to be constructed temporarily
- Optimization System:
	- Function: Accept the computational system as the target function to be optimized, execute its own optimization logic, and finally return the computational results
	- Nature: The method system is relatively mature and can be prepared in **advance of the competition** with various optimization systems

The paper writing part is divided into:
- Overall Framework: Determined by the LaTeX template
- Main Content Filling: Clear description of the workflow and optimization flow
- Typesetting and Beautification: Adjust the details of each part, with illustrative images (flowcharts, schematics)
- Concluding Content

#### Pre-Modeling

- Objective: Under the premise of accurately understanding the problem, quickly carry out preliminary modeling, basically determine the direction of modeling and calculation methods;

- Estimated Time: 3 hours

- Work: All team members conduct a web search to see if there are any literature materials that **basically hit the topic**.
	- Hit Successful: The most ideal situation, at this time, you can directly study the papers and collect ideas;
	- Hit Unsuccessful: Although there are no ready-made materials for reference, some ideas have been accumulated in the process of literature review.

#### Early Modeling

- Overall Objective: Construct a precise and **optimization method adaptable** computational system
	- Modeling: Clarify the operations between input data and each computational flow node
	- Coding: Implement the computational flow with code and achieve data visualization
	- Paper: Fill in the content of the first question and initially typeset

- Estimated Time: 30 hours

- Work:
	1. All team members model together, first clarify the modeling ideas, and provide a complete mathematical derivation process
	2. Me and CL: Code implementation and paper content filling are carried out simultaneously
	3. HWJ: Draw more vivid schematic diagrams that cannot be generated by code

#### Mid-Modeling

- Overall Objective: Construct a **suitable** optimization system
	- Modeling: According to the particularity of the computational system, choose the most matching optimization system
	- Coding: Make minor changes in the implementation of the optimization system to match the computational system
	- Paper: Complete the main part of the paper and start local detail fine-tuning

- Estimated Time: 20 hours

- Work: Similar to the previous, but the focus of work has shifted from code writing to paper writing
	- Simplify the paper, at this time, the paper is very bloated
	- Fine-tune the logic of the paper to make the context more closely related
	- Beautify the typesetting, reduce text, increase images

#### Late Modeling

- The basic modeling is completed, and all members check for loopholes:
	- Conventional checks such as typos, inaccurate expressions, formula spelling errors, etc.
	- Optimize code comments to make them more readable
	- Focus on checking **personal information**



{{< alert icon="fire" cardColor="#e63946" iconColor="#ffffff" textColor="#ffffff" >}}
Personal information must not be retained in the competition paper, including file paths in the code, such as `C:\Users\Morethan`; retaining personal information is a very serious mistake!
{{< /alert >}}


### Actual Combat Effectiveness

When we applied the above strategies to the actual combat process, that is, the formal competition of CUMCM 2024, the results were as follows:

- Effective Time:
	- The total duration of the competition is three days, a total of 72 hours
	- The team works from seven in the morning to eight in the evening, excluding meal times, with an effective time of 12 hours a day
	- Time utilization rate is \(50\%\) (quite low in comparison🤔)
- Completed Work:
	- The main body of the paper is 28 A4 pages
	- The code part is 35 A4 pages, excluding the reused code between each sub-question, there should be about 20 pages
	- A total of 25 illustrations in the paper
	- The above data is after the paper has been streamlined, with the initial draft of the paper being nearly 50 pages
- Uncompleted Work:
	- The final result calculation, due to the large amount of calculation (the code efficiency is not high), the code was finished two hours in advance, but there was not enough time to calculate the results😭😭
	- The calculation accuracy of the model is not enough, the accuracy is `1s` which does not meet the standard answer's precision
	- The conclusion part of the paper was not actually completed

## Strengths

### Topic Selection

- Focused on Topic A, accumulated sufficient experience in mock contests, and polished a set of efficient workflows

- The methodology for Topic A is relatively well-constructed

### Workflow

- The workflow is relatively clear, and the efficiency is high

- Guided by the final paper, modeling, paper, and code are carried out simultaneously, ensuring sufficient content in the paper

### Division of Labor

- Adopted a blurred division of labor, each team member has a main job and a secondary job, can work independently on their main job, and can also complete some work on the secondary job, greatly improving time utilization

- The team members are very capable, as handling two division tasks means more learning costs

## Weaknesses

### Workflow

- The plan is perfect, but some necessary links were not well done in practice

- Effective time ratio: finishing work at eight in the evening is too early! More time should be taken to model trial and error to ensure the correctness and accuracy of the model

### Division of Labor

- The code writing, code debugging, code visualization, result calculation, and result visualization involve too much code, which is difficult for one person to handle;

- Task overlap caused by blurred division of labor increases collaboration costs

### Modeling

- Topic understanding accuracy: This time, there was a significant deviation in our understanding of the topic, which led to wasting a lot of time on model correction;

### Code

- Code efficiency: Due to no time limit before, there was insufficient preparation for "very long" code, no experience with code parallelism;

- Result precision: The initial modeling was too rough, and a bad characteristic was used: setting the time step to 1, and using it **as an array index**, which made it difficult to reduce the time step later, resulting in insufficient precision of the final results

## Improvement Plans

- Carefully select the venue, **increasing effective time**✨is the most important✨

### Division of Labor

- Slightly change the division of labor, increase the investment of human resources in coding

- Increase learning input in each main and secondary division to increase work efficiency

### Modeling

- Focus more on understanding the topic, don't rush; correcting modeling errors is not worth the loss

### Code

- Build a set of effective code collaboration plans to enhance code writing speed

- Start building code writing standards:
	- Variable naming
	- Documentation at the beginning of the file
	- **Code writing process standards**

- Code parallelization: Add some parallelizable code to the code to increase running speed


{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
All code improvements must be implemented in a document! Not just slogans!
{{< /alert >}}

- The final output: [Code Collaboration Scheme]({{< ref "/blog/code-collaboration-scheme/" >}})

### Paper

- Study excellent papers
	- Pay attention to its paper framework
	- Pay attention to its language style, text readability, detail, illustration logic, and image readability

- Improve ourselves
	- Optimize the paper's main logic framework, refine the content of each section
	- Improvements in language style, text readability, detail, illustration logic, and image readability, etc.


{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
The results are fixed in the form of comments in the LaTeX template!
{{< /alert >}}

## Summary

> A test paper without full marks is more rewarding than one with full marks!

Accumulating knowledge of applied mathematics, enhancing paper writing skills, and improving the ability to discover problems are more meaningful than the competition itself. 🫡

CUMCM, every MathModeler can benefit from it. 🤗
