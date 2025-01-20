---
title: Python小技巧
weight: -15
draft: false
description: Python中的一些小技巧
slug: pytips
tags:
  - Python
series:
  - 技术流程
series_order: 2
date: 2024-08-10
lastmod: 2025-01-20
authors:
  - Morethan
---

## 创建虚拟环境
### 创建
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

| 参数名                   | 含义                                     |
| ------------------------ | --------------------------------------- |
| `--system-site-packages` | 创建的虚拟环境将包含全局Python环境中的包，这可以避免重复安装一些常用的包|
| `--clear`                | 如果指定的虚拟环境目录已经存在，这会清除目录中的所有内容，然后重新创建虚拟环境 |
| `--version`              | 用于确认虚拟环境中 Python 的版本                    |


{{< alert icon="circle-info" cardColor="#b0c4de" textColor="#333333" >}}
所有的参数说明都可以通过运行 `python -m venv -h` 来获得；不用到处查文档了~😆
{{< /alert >}}
### 激活
默认情况下，虚拟环境处于非激活状态。在“your_env_name/Scripts/”目录下将有一个名为“activate”的文件，用命令行运行即可。

```sh
# 激活虚拟环境
your_env_name/Scripts/activate
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

