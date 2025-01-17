---
title: 代码协同方案
weight: -50
draft: false
description: 针对数学建模国赛和MATLAB改进的代码协同方案
slug: code-collaboration-scheme
tags:
  - MATLAB
  - CUMCM
series:
  - 数学建模
series_order: 3
date: 2025-01-16
lastmod: 2025-01-17
authors:
  - Morethan
---
{{< lead >}}
总结 CUMCM2024 比赛经验，针对数学建模国赛 A 题和MATLAB改进的代码协同方案
{{< /lead >}}

## 流程概览
{{< mermaid >}}
graph LR;
id1("工作分配")-->id2("VS Code协同")-->id3("MATLAB代码执行");
{{< /mermaid >}}

## 工作分配
主要有两个要点：

1. 划分为两个部分，一个主要部分和次要部分；
2. 划分的任务之间不要有依赖关系

## 代码协同
### 软件工具
1. VS Code 编辑器，以及 Live Server 插件，MATLAB 插件，matlab-formatter 格式化插件

2. MATLAB

### 命名规范
1. 主函数统一命名为 `main`：能够直接计算出最终答案的叫主函数，其他的都叫辅助函数
3. 数据加工代码：数据加工代码指的是没有任何返回值，仅仅产生数据表格的代码，用 `data` 开头，例如：将太阳高度角 {{< katex >}}\\(\\phi\\) 转化为余弦值，`dataCosPhi`
4. 内部数据转换代码，用 `to` 连接，例如将自然坐标系中点的坐标转化为直角坐标系中点的坐标 `StoXY`
5. 绘图代码：绘制图形的代码，用 `fig` 开头
6. 测试代码：`test` 开头
7. 其余需要特殊定义的代码：根据功能，返回布尔值的用 `is` 开头，如判断碰撞函数 `isCollided`；返回其他数据的用 `get` 开头

### 文件说明注释
除了上面命名规范里面提及的一些常见类型的文件不用添加参数外，即 `data`，`test`，`fig`，`main` 这四个，其余的都应该在文件内添加说明性注释；

简明而清晰的文件说明注释：文件说明一般在文件开头，包含函数功能概述、函数接受参数、函数输出参数三个主要部分

实在不会写，用 AI 帮助就行；

### 内部变量命名
#### 统一的固定参数
所有**固定的参数**全部都放置到根目录下的一个 `config.m` 文件中，进行统一管理，每个参数后面请添加注释说明，例如：

```MATLAB
learningRate = 0.001; % 学习率
batchSize = 32; % 每批输入的数据量
numEpochs = 100; % 迭代次数
```

在其他代码中引用这个配置文件只需要加入代码：

```MATLAB
run('config.m');
```

{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
尽管参数被转移到了其他地方但是 VS Code 仍然能够进行自动检测并给出补全提示！
{{< /alert >}}
#### 函数内部使用的参数
函数内部所有使用的变量都应该在主程序开始之前明确定义，并在变量后添加简要的中文注释；

{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
如果有一些表达相同含义的参数在多个文件中使用，请使用统一的命名，尤其是 AI 生成的代码，请在 VS Code 中使用 `F2` 进行重命名
{{< /alert >}}
### 代码格式化
代码的格式化主要通过 matlab-formatter 格式化插件来实现：在启动插件之后，在 VS Code 中使用快捷键 `Ctrl+Shift+P` 呼出命令面板，搜索 `格式化`，然后回车选中 `使用...来格式化代码`，然后选择 `matlab-formatter` 回车即可

### Git 版本控制
这里需要新建一个 GitHub 代码仓库来存放整个项目文件。尽管使用了 Live Server 插件能够更加快速地执行代码协同，但是仍然需要在一些重要的开发结点进行提交保存，保留代码的回滚能力。

所有的 Git 版本控制操作都在代码负责人的电脑上操作，其他辅助编程人员使用 Live Server 插件进行更实时的代码协同。

- 本地 `commit` 保存小改进
- `push` 操作以小题为单位进行，保存重大进展

## 代码执行
在 VS Code 里面直接运行 MATLAB 代码会出现一些奇怪的问题，因此运行代码请在 MATLAB 原生环境中运行

## 其他
### AI 指令集
由于生成式 AI 的代码规范可能与项目不同，因此如果**有生成一整个文件的需求**，请在你的指令之前添加如下的代码规范指令：

```text
你是一个成熟而规范的MATLAB程序员，在正确实现用户目标的前提下，遵守以下代码规范：
1. 简明而清晰的文件说明注释：文件说明一般在文件开头，包含函数功能概述、函数接受参数、函数输出参数三个主要部分
2. 函数内部的变量命名应当简明易读
3. 函数内部所有使用的变量都应该在主程序开始之前明确定义，并在变量后添加简要的中文注释

用户指令：
```

### 配套样例项目
-  [样例项目](https://github.com/morethan987/morethan987/tree/main/MathModelExampleProject)

### 代码技巧
- **并行运行**
MATLAB 能够支持多线程计算，仅需将一般的 `for` 循环改写为 `parfor` 即可

{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
parfor 函数的执行要求十分严格，具体参阅[官方说明](https://ww2.mathworks.cn/help/parallel-computing/parfor.html)
{{< /alert >}}
### 大型表格处理
模型计算的输出往往是一个超大型数据表格，并且存储在 `.mat` 文件中，难以进行数据的快速提取。

直接编写了一个简单的 Python 脚本来进行大型表格数据的提取操作：[数据提取器](https://github.com/morethan987/morethan987/tree/main/%E6%95%B0%E6%8D%AE%E6%8F%90%E5%8F%96%E5%99%A8)

同时配合 LaTeX 在线表格编辑[网站](https://tableconvert.com/zh-cn/latex-generator)，能够实现在论文中快速插入表格数据。