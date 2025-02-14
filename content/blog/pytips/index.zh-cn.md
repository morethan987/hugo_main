---
title: Python小技巧
weight: -15
draft: false
description: Python中的一些小技巧
slug: pytips
tags:
  - Python
series:
  - 技术杂项
series_order: 2
date: 2024-08-10
lastmod: 2025-02-13
authors:
  - Morethan
---

## 创建虚拟环境
### 常规 Python 操作
#### 创建
一些常规的代码例子如下👇

```sh
# 创建虚拟环境
python -m venv your_env_name

# 指定python版本创建虚拟环境，如果你的python是默认安装路径
python -m venv your_env_name --python=python3.11

# python是自定义的安装路径
D:\Python\Python311\python.exe -m venv your_env_name
```

下面有一些可选参数用于创建自定义的虚拟环境：

| 参数名                      | 含义                                      |
| ------------------------ | --------------------------------------- |
| `--system-site-packages` | 创建的虚拟环境将包含全局Python环境中的包，这可以避免重复安装一些常用的包 |
| `--clear`                | 如果指定的虚拟环境目录已经存在，这会清除目录中的所有内容，然后重新创建虚拟环境 |
| `--version`              | 用于确认虚拟环境中 Python 的版本                    |


{{< alert icon="circle-info" cardColor="#b0c4de" textColor="#333333" >}}
所有的参数说明都可以通过运行 `python -m venv -h` 来获得；不用到处查文档了~😆
{{< /alert >}}
#### 激活
默认情况下，虚拟环境处于非激活状态。在“your_env_name/Scripts/”目录下将有一个名为“activate”的文件，用命令行运行即可。

```sh
# 激活虚拟环境
your_env_name/Scripts/activate
```

### Poetry
Poetry 是一个 Python 的包管理工具，从[更新日志](https://python-poetry.org/history/)来看，它从 2018 年 2 月就已经开始填充代码了，不算一个新的工具，但是最近几年非常流行💫

其官方宣传的亮点如下：

- 更佳详尽全面的第三方依赖包分析
- **自动化**的虚拟环境创建
- 更佳符合直觉的命令🤔

#### 安装
非常直接：`pip install poetry`，进行全局安装

#### 全局配置
poetry 有一些非常特殊的机制，可能和 pip 等有些不太一样。你可能会需要去设置一下某些配置，具体如下：

```sh
# 列举出所有配置项
poetry config --list

# poetry默认将虚拟环境创建在一个专属的文件夹中，而不是项目目录中
# 使用下面的命令将环境文件夹创建在项目目录中
poetry config virtualenvs.in-project true

# 当项目中没有虚拟环境的时候，poetry默认能够自动创建
# 不放心可以再运行一下这个命令
poetry config virtualenvs.create true

# 其他的几乎不用关注，默认即可
```

#### 初始化项目
如果你需要从头创建一个新的项目，那么你需要：创建一个项目文件夹，并把命令行的目录切换到那里，然后简单地：

```sh
poetry init
```

交互式的命令行会指引你创建一个新的项目，并创建两个 poetry 特色文件：`pyproject.toml` 管理依赖包；`poetry.lock` 来锁定包版本，类似于 `pip` 的 `requirements.txt`。用过 Node 的同志应该比较熟悉😝

如果你使用的是别人的项目，那么一般都会自带那两个特色文件。

如果你需要一个简单的目录框架，则可以使用：

```sh
poetry new my-package
```

它会自动创建一个适合大多数项目的目录结构：

```sh
my-package
├── pyproject.toml
├── README.md
├── my_package
│   └── __init__.py
└── tests
    └── __init__.py
```

如果你有更加复杂的需求：[new Command](https://python-poetry.org/docs/cli#new)

#### 创建虚拟环境
这里就有几种情况可以说说。

1. 你是从头开始构建自己的项目

```sh
poetry env use python # 还记得你怎么安装poetry的吗？这里的Python默认那个版本

poetry env use python3.7 # 使用你系统中的Python3.7版本

poetry env use 3.7 # 懒癌专属

poetry env use /full/path/to/python # 如果上面的简写形式没法识别，用这个
```

2. 你是使用别人的项目

有两个非常类似的命令，从下面的注释就可以看出二者的区别。官方更推荐 sync 命令，因为这样可以避免安装一些没有被 `poetry.lock` 追踪的包，这些包可能是原开发者不小心加进去的；

**但是**实际测试下来更建议使用 install，因为 sync 有时会出现一些奇怪的错误，比如它把自己给移除了，导致命令行崩溃🤔

```sh
poetry install # 自动创建虚拟环境并安装pyprojecy.toml中的所有依赖包

# 确保你的 virtualenvs.create 设置为 true
# 自动创建虚拟环境并安装poetry.lock中的所有依赖包，移除pyproject.toml中多余的包
poetry sync
```

然后使用如下命令进入虚拟环境

```sh
poetry shell
```

#### 添加包
poetry 允许开发者区分哪些包是生产环境需要的，哪些包只是开发环境需要的

```sh
# 默认添加到生产环境组
poetry add your-package

# 添加到开发环境组
poetry add your-package -D
```

#### 更新包
```sh
poetry update # 自动分析依赖并更新可能的所有包

poetry update your-package1 your-package2 # 只更新你列举的包
```

#### 移除包
这个确实是 poetry 值得圈点的功能：pip 在安装时会安装你给出的包及其第三方依赖，但是在移除包的时候只会移除你列出的包，而无法移除其对应的第三方依赖。

poetry 却能够安全完整地移除你的第三方依赖包，并且不影响别的依赖包。

```sh
# 移除生产环境的包
poetry remove your-package

# 移除开发环境的包
poetry remove your-package -D
```

#### 显示依赖
```sh
poetry show --tree # 展示pyproject.toml中的依赖树

poetry show your-package --tree # 展示特定包的依赖树
```

## 程序打包
我们经常需要把自己写的 Python 程序分享出去。然而只分享源代码会使得不太懂代码的用户非常苦恼，因为源代码的运行还需要搭建本地运行环境，因此打包程序应运而生。

### pyinstaller
安装非常简单，就像别的 Python 包一样 `pip install pyinstaller` 即可；使用时在目标文件所在的目录启动终端，然后运行命令即可。

**特点：打包速度较快；打包后的程序较大**

常见的命令代码：

```shell
# 将main.py文件打包为一个独立的可执行文件；可执行文件运行后禁用命令行窗口
pyinstaller -F -w main.py

# 将main.py文件打包为一个工程文件夹；可执行文件运行后启用命令行窗口
pyinstaller -D main.py
```

| 参数                                    | 说明                                                         |
| ------------------------------------- | ---------------------------------------------------------- |
| `-h, --help`                          | 显示帮助信息并退出                                                  |
| `-v, --version`                       | 显示程序版本信息并退出                                                |
| `-F, --onefile`                       | 将所有内容打包为一个独立的可执行文件                                         |
| `-D, --onedir`                        | 将所有内容打包为一个目录（默认选项）                                         |
| `-w, --windowed, --noconsole`         | 禁用命令行窗口（仅对 Windows 有效）                                     |
| `-c, --console, --nowindowed`         | 使用命令行窗口运行程序（默认选项，仅对 Windows 有效）                            |
| `-a, --ascii`                         | 不包含 Unicode 字符集支持（默认包含）                                    |
| `-d, --debug`                         | 产生 debug 版本的可执行文件                                          |
| `-n NAME, --name=NAME`                | 指定生成的可执行文件或目录的名称（默认为脚本名称）                                  |
| `-o DIR, --out=DIR`                   | 指定 spec 文件的生成目录（默认为当前目录）                                   |
| `-p DIR, --path=DIR`                  | 设置 Python 导入模块的路径（类似设置 PYTHONPATH）                         |
| `-i <FILE>, --icon <FILE>`            | 设置可执行文件的图标（支持 `.ico` 或 `.icns` 格式）                         |
| `--distpath DIR`                      | 指定生成的可执行文件的输出目录（默认为 `./dist`）                              |
| `--workpath WORKPATH`                 | 指定临时工作文件的目录（默认为 `./build`）                                 |
| `--add-data <SRC;DEST or SRC:DEST>`   | 添加额外的数据文件或目录到可执行文件中（Windows 使用分号，Linux/OSX 使用冒号分隔源路径和目标路径） |
| `--add-binary <SRC;DEST or SRC:DEST>` | 添加额外的二进制文件到可执行文件中                                          |
| `--hidden-import MODULENAME`          | 添加未自动检测到的模块                                                |
| `--exclude-module EXCLUDES`           | 排除指定的模块                                                    |
| `--clean`                             | 清理 PyInstaller 缓存和临时文件                                     |
| `--log-level LEVEL`                   | 设置构建时控制台消息的详细程度（可选值：TRACE、DEBUG、INFO、WARN、ERROR、FATAL）     |

### Nuitka
将 Python 代码打包为 exe 可执行文件，转换原理是先将 Python 代码转换为 C 代码，然后再编译 C 代码。

**特点：打包速度相当慢；需要额外安装 C 编译器，尽管可以自动完成但是对于内存空间管理非常严格的用户并不适用；打包后的程序体积很小（实测是 pyinstaller 的十分之一）**

安装命令：
```shell
pip install -U nuitka
```

常见使用命令：
```shell
# 将main.py文件打包为一个exe文件，使用链式优化，完成打包后清理临时文件
python -m nuitka --lto=yes --remove-output --onefile main.py
```

| 参数                                                                                              | 说明                                                        |
| ----------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| `--standalone`                                                                                  | 创建一个包含所有依赖的独立可执行文件夹。                                      |
| `--onefile`                                                                                     | 将所有内容打包为一个单独的 `.exe` 文件。                                  |
| `--optimize=N`                                                                                  | 设置优化级别（`0`、`1` 或 `2`），数字越大，优化越多。                          |
| `--lto`                                                                                         | 启用链接时优化（Link Time Optimization），可选值为 `no`、`yes` 或 `thin`。 |
| `--enable-plugin=<plugin_name>`                                                                 | 启用指定插件，如 `tk-inter`、`numpy`、`anti-bloat` 等。               |
| `--output-dir=<dir>`                                                                            | 指定编译输出目录。                                                 |
| `--remove-output`                                                                               | 编译完成后删除中间生成的 `.c` 文件和其他临时文件。                              |
| `--nofollow-imports`                                                                            | 不递归处理任何导入模块。                                              |
| `--include-package=<package_name>`                                                              | 显式包含整个包及其子模块。                                             |
| `--include-module=<module_name>`                                                                | 显式包含单个模块。                                                 |
| `--follow-import-to=<module/package>`                                                           | 指定递归处理的模块或包。                                              |
| `--nofollow-import-to=<module/package>`                                                         | 指定不递归处理的模块或包。                                             |
| `--include-data-files=<source>=<dest>`                                                          | 包含指定的数据文件。                                                |
| `--include-data-dir=<directory>`                                                                | 包含整个目录的数据文件。                                              |
| `--noinclude-data-files=<pattern>`                                                              | 排除匹配模式的数据文件。                                              |
| `--windows-icon-from-ico=<path>`                                                                | 设置 Windows 可执行文件的图标。                                      |
| `--company-name`, `--product-name`, `--file-version`, `--product-version`, `--file-description` | 设置 Windows 可执行文件的属性。                                      |

## 参考资料
- Poetry 相关：
	- [poetry 入门完全指南_poetry使用-CSDN博客](https://blog.csdn.net/weixin_42871919/article/details/137125544)非常详细的资料
	- [Poetry](https://python-poetry.org/)

