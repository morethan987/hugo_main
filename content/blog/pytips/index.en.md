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
lastmod: 2025-01-20
authors:
  - Morethan
---

## Virtual Env
### Creat
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

### Activate
The virtual env is defaulted not active. In the direction `your_env_name/Scripts/` will be a file named `activate`. Run it with your cmd.

```sh
# activate virtual env
your_env_name/Scripts/activate
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
