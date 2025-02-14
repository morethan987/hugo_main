---
title: Python Tpis
weight: -15
draft: false
description: A set of Python tips
slug: pytips
tags:
  - Python
series:
  - Technical Miscellany
series_order: 2
date: 2024-08-10
lastmod: 2025-02-13
authors:
  - Morethan
---

## Virtual Env
### Plain Python Method
#### Creat
Some tipical code 👇

```sh
# creat a virtual env named "your_env_name"
python -m venv your_env_name

# assign the version of python, make sure your python in default direction
python -m venv your_env_name --python=python3.11

# simply list the absolute direction of python, simple and efficient
D:/PythonPython311/python.exe -m venv your_env_name
```

More parameters you may need for a customized virtual env. 🤔

| Params                   | Meaning                                                                                             |
| ------------------------ | --------------------------------------------------------------------------------------------------- |
| `--system-site-packages` | Give the virtual environment access to the system site-packages dir.                                |
| `--clear`                | Delete the contents of the environment directory if it already exists, before environment creation. |
| `--version`              | print the python version of the env                                                                 |

{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
All the detailed expaination of the parameters can be got by the code `python -m venv -h`. No need to search everywhere~😆
{{< /alert >}}
#### Activate
The virtual env is defaulted not active. In the direction `your_env_name/Scripts/` will be a file named `activate`. Run it with your cmd.

```sh
# activate virtual env
your_env_name/Scripts/activate
```

### Poetry

Poetry is a Python package management tool that started development in February 2018, according to its [changelog](https://python-poetry.org/history/). It's not exactly a new tool, but it has become quite popular in recent years 💫.

The key highlights promoted by its official documentation include:

- More detailed and comprehensive analysis of third-party dependencies
- **Automated** virtual environment creation
- More intuitive and user-friendly commands 🤔

#### Installation

It's straightforward: `pip install poetry` for global installation.

#### Global Configuration

Poetry has some unique mechanisms that differ from tools like `pip`. You might need to set up some configurations, which can be done as follows:

```sh
# List all configurations
poetry config --list

# By default, Poetry creates virtual environments in a dedicated folder, not within the project directory
# Use the following command to create the virtual environment in the project directory
poetry config virtualenvs.in-project true

# Poetry will automatically create a virtual environment when one doesn't exist in the project
# To be sure, you can run this command
poetry config virtualenvs.create true

# Other settings can usually be left as default
```

#### Initialize a Project

If you need to create a new project from scratch, you'll want to create a project folder, navigate to it in your terminal, and then run:

```sh
poetry init
```

An interactive CLI will guide you through creating a new project, which will generate two key files for Poetry: `pyproject.toml` for managing dependencies, and `poetry.lock` to lock package versions, similar to `pip`'s `requirements.txt`. Those familiar with Node.js should find this concept quite familiar 😝.

If you're working with someone else's project, it will usually include those two files.

If you want a simple directory structure, you can use:

```sh
poetry new my-package
```

This will create a basic project structure like this:

```sh
my-package
├── pyproject.toml
├── README.md
├── my_package
│   └── __init__.py
└── tests
    └── __init__.py
```

For more complex needs, refer to the [new Command documentation](https://python-poetry.org/docs/cli#new).

#### Create a Virtual Environment

There are a few situations to consider here:

1. If you're building your project from scratch:

```sh
poetry env use python # Default Python version you installed Poetry with

poetry env use python3.7 # Use the Python 3.7 version on your system

poetry env use 3.7 # A shortcut for the above

poetry env use /full/path/to/python # Use this if the above shortcuts don't work
```

2. If you're working with someone else's project:

There are two very similar commands. As shown in the comments, the official recommendation is to use `sync`, as it avoids installing any packages that aren't tracked by `poetry.lock`, which may include unintended packages that the original developer added:

```sh
poetry install # Automatically creates a virtual environment and installs all dependencies from pyproject.toml

# Make sure virtualenvs.create is set to true
poetry sync # Automatically creates a virtual environment and installs dependencies from poetry.lock
```

To activate the virtual environment, use:

```sh
poetry shell
```

#### Add Packages

Poetry lets you differentiate between packages needed in the production environment and those only required for development:

```sh
# Add a package to the production environment
poetry add your-package

# Add a package to the development environment
poetry add your-package -D
```

#### Update Packages

```sh
poetry update # Automatically analyzes dependencies and updates all packages if necessary

poetry update your-package1 your-package2 # Only updates the specific packages you list
```

#### Remove Packages

This is one of Poetry's standout features: While `pip` installs the specified package and its third-party dependencies, it can’t remove these dependencies when you uninstall a package. Poetry, on the other hand, safely and completely removes the dependencies tied to a package without affecting others.

```sh
# Remove a package from the production environment
poetry remove your-package

# Remove a package from the development environment
poetry remove your-package -D
```

#### Show Dependencies

```sh
poetry show --tree # Show the dependency tree of the packages in pyproject.toml

poetry show your-package --tree # Show the dependency tree for a specific package
```

## Program Packaging

We often need to share our Python programs. However, sharing the source code alone can be frustrating for users who aren't familiar with coding, as it requires setting up a local environment to run the code. Therefore, program packaging comes in handy.

### PyInstaller

Installation is very simple, just like any other Python package: `pip install pyinstaller`. To use it, open the terminal in the target directory and run the command.

**Features: Fast packaging speed; relatively large packaged files**

Common command codes:

```shell
# Package the main.py file as a standalone executable; the command window is disabled when the executable runs
pyinstaller -F -w main.py

# Package the main.py file into a project folder; the command window is enabled when the executable runs
pyinstaller -D main.py
```

|Parameter|Description|
|---|---|
|`-h, --help`|Show help information and exit|
|`-v, --version`|Show program version information and exit|
|`-F, --onefile`|Package everything into a single standalone executable|
|`-D, --onedir`|Package everything into a directory (default option)|
|`-w, --windowed, --noconsole`|Disable the command window (only works on Windows)|
|`-c, --console, --nowindowed`|Use the command window to run the program (default option, only on Windows)|
|`-a, --ascii`|Exclude Unicode character set support (included by default)|
|`-d, --debug`|Generate a debug version of the executable|
|`-n NAME, --name=NAME`|Specify the name of the generated executable or directory (default is script name)|
|`-o DIR, --out=DIR`|Specify the directory where the spec file will be generated (default is the current directory)|
|`-p DIR, --path=DIR`|Set the Python module import path (similar to setting PYTHONPATH)|
|`-i <FILE>, --icon <FILE>`|Set the executable file icon (supports `.ico` or `.icns` formats)|
|`--distpath DIR`|Specify the output directory for the executable file (default is `./dist`)|
|`--workpath WORKPATH`|Specify the temporary working file directory (default is `./build`)|
|`--add-data <SRC;DEST or SRC:DEST>`|Add additional data files or directories to the executable (use semicolon for Windows, colon for Linux/OSX to separate source and destination paths)|
|`--add-binary <SRC;DEST or SRC:DEST>`|Add additional binary files to the executable|
|`--hidden-import MODULENAME`|Add modules not automatically detected|
|`--exclude-module EXCLUDES`|Exclude specified modules|
|`--clean`|Clean PyInstaller cache and temporary files|
|`--log-level LEVEL`|Set the verbosity of console messages during the build process (possible values: TRACE, DEBUG, INFO, WARN, ERROR, FATAL)|

### Nuitka

Nuitka packages Python code into an executable (.exe). The underlying process first converts Python code to C code, then compiles the C code.

**Features: Slow packaging speed; requires an additional C compiler (though automatic setup is possible, it can be strict on memory management, so may not be suitable for users with limited space); smaller packaged files (tested to be about one-tenth the size of PyInstaller's output)**

Installation command:

```shell
pip install -U nuitka
```

Common usage command:

```shell
# Package the main.py file into an exe file with link-time optimization, and remove temporary files after packaging
python -m nuitka --lto=yes --remove-output --onefile main.py
```

| Parameter                                                                                       | Description                                                                                      |
| ----------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `--standalone`                                                                                  | Create a standalone executable folder that includes all dependencies.                            |
| `--onefile`                                                                                     | Package everything into a single `.exe` file.                                                    |
| `--optimize=N`                                                                                  | Set optimization level (`0`, `1`, or `2`), the higher the number, the more optimized the result. |
| `--lto`                                                                                         | Enable Link-Time Optimization (possible values: `no`, `yes`, `thin`).                            |
| `--enable-plugin=<plugin_name>`                                                                 | Enable a specific plugin, such as `tk-inter`, `numpy`, `anti-bloat`, etc.                        |
| `--output-dir=<dir>`                                                                            | Specify the output directory for the compiled files.                                             |
| `--remove-output`                                                                               | Delete intermediate `.c` files and other temporary files after compilation.                      |
| `--nofollow-imports`                                                                            | Do not recursively process any imported modules.                                                 |
| `--include-package=<package_name>`                                                              | Explicitly include an entire package and its submodules.                                         |
| `--include-module=<module_name>`                                                                | Explicitly include a specific module.                                                            |
| `--follow-import-to=<module/package>`                                                           | Specify which modules or packages to recursively process.                                        |
| `--nofollow-import-to=<module/package>`                                                         | Specify which modules or packages not to recursively process.                                    |
| `--include-data-files=<source>=<dest>`                                                          | Include specified data files.                                                                    |
| `--include-data-dir=<directory>`                                                                | Include all data files in the specified directory.                                               |
| `--noinclude-data-files=<pattern>`                                                              | Exclude data files matching a specified pattern.                                                 |
| `--windows-icon-from-ico=<path>`                                                                | Set the Windows executable icon.                                                                 |
| `--company-name`, `--product-name`, `--file-version`, `--product-version`, `--file-description` | Set Windows executable properties.                                                               |

## Reference
- poetry related: 
	- [poetry 入门完全指南_poetry使用-CSDN博客](https://blog.csdn.net/weixin_42871919/article/details/137125544), a detailed and first-hand blog.
	- [Poetry](https://python-poetry.org/), the official documents.