---
title: Code Collaboration Scheme
weight: -50
draft: false
description: Code Collaboration Scheme especial for CUMCM and MATLAB
slug: code-collaboration-scheme
tags:
  - MATLAB
  - CUMCM
series:
  - MathModel
series_order: 3
date: 2025-01-16
lastmod: 2025-01-20
authors:
  - Morethan
---
{{< katex >}}
{{< lead >}}  
Summary of CUMCM 2024 competition experience, focusing on the A problem and the collaborative approach with MATLAB code improvement  
{{< /lead >}}

## Process Overview

{{< mermaid >}}
graph LR;
id1("Task Allocation")-->id2("VS Code Collaboration")-->id3("MATLAB Code Execution");
{{< /mermaid >}}

## Task Allocation

There are two key points:

1. Tasks are divided into two parts: a main part and a secondary part.
2. Tasks should be independent of each other, meaning no dependencies between them.

## Code Collaboration

### Software Tools

1. VS Code Editor, along with the Live Server plugin, MATLAB plugin
2. MATLAB.

### Naming Conventions

1. Main function should be uniformly named `main`: The main function is the one that directly computes the final result, while others should be called auxiliary functions.
2. Data processing code: This refers to code that does not return any value but generates data tables. It should start with `data`, e.g., converting solar altitude angle \(\phi\) to cosine value, `dataCosPhi`.
3. Internal data conversion code: This should start with `to`, e.g., converting coordinates from a natural coordinate system to a Cartesian coordinate system `StoXY`.
4. Plotting code: Code related to plotting graphs should start with `fig`.
5. Testing code: Should start with `test`.
6. For other special types of functions, they should be named based on their functionality. Functions that return boolean values should start with `is`, such as a collision detection function `isCollided`; functions that return other data should start with `get`.

### File Documentation Comments

Except for the common types of files mentioned in the naming conventions (`data`, `test`, `fig`, `main`), all other files should include documentation comments.

The documentation should be concise yet clear. It generally includes a brief description of the function, the input parameters, and the output parameters.

If you're unsure how to write documentation, you can use AI assistance.

### Internal Variable Naming

#### Fixed Parameters

All **fixed parameters** should be placed in a `config.m` file located in the root directory for centralized management. Each parameter should be followed by a comment explaining its purpose, for example:

```MATLAB
learningRate = 0.001; % Learning rate
batchSize = 32; % Batch size
numEpochs = 100; % Number of epochs
```

To use this configuration file in other code files, simply include the following line:

```MATLAB
run('config.m');
```



{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
Even the parameters are transformed into another file, the VS Code can also recognizes it and provide complement suggestion.
{{< /alert >}}


#### Function-Internal Parameters

All variables used within a function should be explicitly defined at the beginning of the function and should include brief Chinese comments.



{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
If similar parameters are used across multiple files, make sure to use consistent naming, especially in AI-generated code. Use `F2` in VS Code to rename them.
{{< /alert >}}


### Code Formatting

Code formatting is mainly handled using the official `MATLAB` plugin. After you activate the plugin, press `Shift+Alt+F` to format your code.

### Git Version Control

A GitHub repository needs to be created to store the entire project files. Although the Live Server plugin enables faster real-time code collaboration, it is still necessary to commit and save at key development milestones to maintain the ability to roll back code.

All Git version control operations are performed on the lead developer's computer, while other supporting developers use the Live Server plugin for more immediate code collaboration.

- Local `commit` to save small improvements
- `push` operations are done in major increments

## Code Execution

Thanks to the latest function of the plugin `MATLAB`, we can now directly **run and debug** in the VS Code IDE. Though there are some [limitations](https://github.com/mathworks/MATLAB-extension-for-vscode?tab=readme-ov-file#limitations), it deserves a standing ovation.

## Other

### AI Instruction Set

Since generative AI's code style might differ from the project's standards, if you need to **generate an entire file**, please include the following code standard instructions at the beginning of your request:

```text
You are a mature and standardized MATLAB programmer. While correctly implementing the user's goal, you should follow these code standards:
1. Clear and concise file documentation comments: The documentation should include a brief description of the function, input parameters, and output parameters.
2. Function-internal variable names should be concise and easy to read.
3. All variables used within a function should be explicitly defined at the beginning of the function, with brief Chinese comments added next to them.

User's instruction:
```

### Example Project

In order to develop a project as fast and well-defined as possible, I have created an [Example Project](https://github.com/morethan987/morethan987/tree/main/MathModelExampleProject) which contains all the regulations mentioned before. You can directly change the files in the example project without mnemonic cost. 😄

### Code Tips

- **Parallel Execution**  
    MATLAB supports multi-threaded computing. You can convert a typical `for` loop to a `parfor` loop for parallel execution.



{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
The execution of `parfor` functions is subject to strict requirements. Please refer to the [official documentation](https://ww2.mathworks.cn/help/parallel-computing/parfor.html) for more details.
{{< /alert >}}


### Huge Table Process

The comment output of the Mathematical Model is a **Huge** table in `.mat` file, which is usually hard to abstract the target data.

Here I directly write a simply Python program, [Data extractor](https://github.com/morethan987/morethan987/tree/main/%E6%95%B0%E6%8D%AE%E6%8F%90%E5%8F%96%E5%99%A8), to automatically operate the huge table data. With tiny modification, you can get any data you want from the `.mat` file.

Utilizing the online LaTeX table [editor](https://tableconvert.com/zh-cn/latex-generator), we can get the capacity to quickly insert the table data into your thesis.
