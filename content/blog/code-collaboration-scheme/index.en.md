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
lastmod: 2025-01-16
authors:
  - Morethan
---

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

1. VS Code Editor, along with the Live Server plugin, MATLAB plugin, and matlab-formatter for code formatting.
2. MATLAB.

### Naming Conventions

1. Main function should be uniformly named `main`: The main function is the one that directly computes the final result, while others should be called auxiliary functions.
2. Data processing code: This refers to code that does not return any value but generates data tables. It should start with `data`, e.g., converting solar altitude angle {{< katex >}}\\(\\phi\\) to cosine value, `dataCosPhi`.
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

#### Function-Internal Parameters

All variables used within a function should be explicitly defined at the beginning of the function and should include brief Chinese comments.

{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
If similar parameters are used across multiple files, make sure to use consistent naming, especially in AI-generated code. Use `F2` in VS Code to rename them.
{{< /alert >}}
### Code Formatting

Code formatting is mainly handled using the `matlab-formatter` plugin. After activating the plugin, you can format your code in VS Code by pressing `Ctrl+Shift+P` to bring up the command palette, then search for "Format" and select `Use... to format code`, choosing `matlab-formatter` to apply the formatting.

## Code Execution

Running MATLAB code directly in VS Code may cause some unexpected issues, so it's recommended to run the code in the native MATLAB environment.

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

- [Example Project](https://github.com/morethan987/morethan987/tree/main/MathModelExampleProject)

### Code Tips

- **Parallel Execution**  
    MATLAB supports multi-threaded computing. You can convert a typical `for` loop to a `parfor` loop for parallel execution.

{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
The execution of `parfor` functions is subject to strict requirements. Please refer to the [official documentation](https://ww2.mathworks.cn/help/parallel-computing/parfor.html) for more details.
{{< /alert >}}