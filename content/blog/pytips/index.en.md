---
title: Python Tips and Tricks
weight: -15
draft: false
description: Some handy tips for Python
slug: pytips
language: en
tags:
  - Python
series:
  - Technical Miscellany
series_order: 2
date: 2024-08-10
lastmod: 2025-05-17
authors:
  - Morethan
---

## Creating Virtual Environments

### Standard Python Operations

#### Creation

Here are some common code examples 👇

```sh
# Create a virtual environment
python -m venv your_env_name

# Create a virtual environment with a specific Python version (if Python is installed in the default location)
python -m venv your_env_name --python=python3.11

# If Python is installed in a custom location
D:\Python\Python311\python.exe -m venv your_env_name
```

Below are some optional parameters for creating customized virtual environments:

| Parameter                     | Description                                                                 |
| ----------------------------- | -------------------------------------------------------------------------- |
| `--system-site-packages`      | Includes packages from the global Python environment to avoid redundant installations |
| `--clear`                     | Clears the target directory if it already exists before creating the virtual environment |
| `--version`                   | Confirms the Python version in the virtual environment                     |


{{< alert icon="circle-info" cardColor="#b0c4de" textColor="#333333" >}}
All parameter descriptions can be obtained by running `python -m venv -h`—no need to search elsewhere for documentation~😆
{{< /alert >}}

#### Activation

By default, the virtual environment is inactive. In the "your_env_name/Scripts/" directory, there will be a file named "activate," which can be executed via the command line.

```sh
# Activate the virtual environment
your_env_name/Scripts/activate
```

### Poetry

Poetry is a Python package management tool. According to its [changelog](https://python-poetry.org/history/), it has been in development since February 2018, making it not a new tool but one that has gained significant popularity in recent years 💫.

Its advertised highlights include:

- More comprehensive third-party dependency analysis
- **Automated** virtual environment creation
- More intuitive commands 🤔

#### Installation

Straightforward: `pip install poetry` for a global installation.

#### Global Configuration

Poetry has some unique mechanisms that may differ from tools like pip. You might need to configure certain settings as follows:

```sh
# List all configuration items
poetry config --list

# By default, Poetry creates virtual environments in a dedicated folder rather than the project directory
# Use the following command to create the environment folder in the project directory
poetry config virtualenvs.in-project true

# Poetry can automatically create virtual environments when none exist in the project
# For reassurance, you can run this command
poetry config virtualenvs.create true

# Switch to a domestic mirror
poetry config repositories.tencentyun https://mirrors.tencentyun.com/pypi/simple

# Other settings can generally be left as defaults
```

#### Initializing a Project

If you're starting a new project from scratch, you'll need to: create a project folder, navigate to it in the command line, and simply run:

```sh
poetry init
```

An interactive command-line interface will guide you through creating a new project, generating a `pyproject.toml` file to record dependencies, and creating Poetry's signature `poetry.lock` file to lock package versions. Those familiar with Node will find this familiar 😝.

If you're using someone else's project, you'll typically only find the `pyproject.toml` file, which is compatible across different package management tools. Some authors may also include the `poetry.lock` file. In such cases, you can choose to: 1) delete the lock file and use another package manager, or 2) keep the lock file and use Poetry to manage dependencies.

If you need a simple directory structure, you can use:

```sh
poetry new my-package
```

This will automatically create a directory structure suitable for most projects:

```sh
my-package
├── pyproject.toml
├── README.md
├── my_package
│   └── __init__.py
└── tests
    └── __init__.py
```

For more complex needs: [new Command](https://python-poetry.org/docs/cli#new).

#### Creating a Virtual Environment

Here are a few scenarios to consider.

1. You're building your project from scratch:

```sh
poetry env use python # Remember how you installed Poetry? This uses the default Python version.

poetry env use python3.7 # Uses Python 3.7 from your system.

poetry env use 3.7 # For the lazy.

poetry env use /full/path/to/python # If the shorthand doesn't work, use this.
```

2. You're using someone else's project:

There are two very similar commands, and the comments below highlight their differences. The official recommendation is the `sync` command, as it avoids installing packages not tracked by `poetry.lock`, which might have been accidentally added by the original developer.

**However**, in practice, `install` is often more reliable, as `sync` can sometimes produce strange errors, like removing itself and crashing the command line 🤔.

```sh
poetry install # Automatically creates a virtual environment and installs all dependencies listed in pyproject.toml.

# Ensure virtualenvs.create is set to true.
# Automatically creates a virtual environment and installs all dependencies listed in poetry.lock, removing any extra packages from pyproject.toml.
poetry sync
```

Then, enter the virtual environment with:

```sh
poetry shell
```

#### Adding Packages

Poetry allows developers to distinguish between production and development dependencies.

```sh
# Defaults to the production group.
poetry add your-package

# Adds to the development group.
poetry add your-package -D
```

#### Updating Packages

```sh
poetry update # Automatically analyzes dependencies and updates all possible packages.

poetry update your-package1 your-package2 # Updates only the listed packages.
```

#### Removing Packages

This is one of Poetry's standout features: pip installs a package and its dependencies but only removes the specified package, leaving its dependencies behind.

Poetry, however, can safely and completely remove a package and its dependencies without affecting other packages.

```sh
# Removes a production dependency.
poetry remove your-package

# Removes a development dependency.
poetry remove your-package -D
```

#### Displaying Dependencies

```sh
poetry show --tree # Displays the dependency tree from pyproject.toml.

poetry show your-package --tree # Displays the dependency tree for a specific package.
```

## UV
A package management tool written in `Rust`, with a command structure very similar to `poetry`.

### Installation

First, configure environment variables:

```bash
cd ~ # Ensure you're in the default directory.
ls -a # Check if the .bashrc file exists.
vim .bashrc # Edit the file with vim.

# Append these three lines to the end of the file.
# Domestic mirror for the installer.
export UV_INSTALLER_GHE_BASE_URL=https://ghproxy.cn/https://github.com
# Domestic mirror for Python installation.
export UV_PYTHON_INSTALL_MIRROR=https://ghproxy.cn/https://github.com/indygreg/python-build-standalone/releases/download
# Domestic mirror for Python packages.
export UV_DEFAULT_INDEX=https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

- **Installation via curl (recommended)**:

```bash
# Replace "/custom/path" with your desired installation location.
curl -LsSf https://astral.sh/uv/install.sh | env UV_INSTALL_DIR="/custom/path" sh
```

- **Installation via pip/pipx (recommended)**:

```bash
pipx install uv # Install uv using pipx.

pip install uv # Install uv using pip.
```

### Usage

Usage is very similar to `poetry`.

## Conda  

I personally don't use Conda much for environment management, but since many lab servers default to using Conda, I’ll document the basics here for reference.  

### Usage  

The following commands are fundamental for checking environment information. They’re typically used when accessing a new server environment or troubleshooting environment-related issues.  

```bash
########## Checking Environment Information ##########

arch # View the system architecture

conda --version # Get the Conda version

python --version # Get the Python version

conda env list # List all available environments

conda activate xxx # Activate the virtual environment named "xxx"

conda deactivate # Exit the current virtual environment

whereis python # Check the location of Python in the currently activated environment

nvidia-smi # Check CUDA status (not a Conda command but frequently used)

conda config --show channels # View Conda's download sources

conda remove -n xxx # Delete the virtual environment named "xxx"

conda create -n env_name # Create a virtual environment with the specified name, using the default Python version
```

Adding domestic (Chinese) download sources:  

```bash  
conda config --show channels # View Conda's download sources  

# Add Tsinghua's mirror source  
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2  
```

Managing Python packages:

```bash  
# Install a specific package
conda install package_name

# Remove a specific package  
conda remove package_name  

# Update a specific package  
conda update package_name  

# Search for a specific package  
conda search package_name  

# List all installed packages and their versions  
conda list  

# Clean unused packages and cache to save disk space  
conda clean --all  

# Install packages from a requirements.txt file  
conda install -f requirements.txt  
```  

Manual installation of dependency packages for handling extreme offline situations:

```bash
# Directly use pip download command to download all related whl files
pip download scikit-image --dest ./scikit_image_files --only-binary :all: --python-version 3.13 --platform manylinux_2_17_x86_64 --implementation cp --abi cp313

# Manually transfer to the server, switch to the target folder, and run (note the whl files)
# --find-links specifies the folder containing the whl files
pip install scikit_image-0.25.2-cp313-cp313-manylinux_2_17_x86_64.manylinux2014_x86_64.whl --no-index --find-links=./
```


{{< alert icon="triangle-exclamation" cardColor="#ffcc00" textColor="#333333" iconColor="#8B6914" >}}
Do not rename the whl files 😢
{{< /alert >}}

## Passwordless Login via VSCode  

For students without a local GPU, editing and running files directly on a server is very convenient. The VSCode extension `remote-ssh` can also display a VSCode interface directly on the server. To avoid the hassle of repeatedly entering passwords, you can set up passwordless login as follows 😄.  

For a local Windows machine, SSH-related configuration files are usually stored in the `C:\Users\<User_name>\.ssh` folder. Passwordless login can be achieved by modifying files in this directory.  

1. Configure the `config` file. A sample configuration is as follows:  

```txt  
# Replace <User_name> with your local machine's username  
Host 3090  
    HostName xx.xxx.xx.xx  
    User morethan  
    IdentityFile "C:\Users\<User_name>\.ssh\id_rsa"  
```  

2. Generate the authentication file `id_rsa.pub`.  

3. Locally create an `authorized_keys` file and copy the contents of `id_rsa.pub` into it.  

4. On the server, create a `.ssh` folder in the default directory and copy the `authorized_keys` file from your local machine into it.  

Now, when logging into the server via VSCode, you’ll find that passwordless login is enabled.  


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
The steps above only need to be performed once initially. When configuring passwordless login for another server, you only need to modify the `config` file and copy the `authorized_keys` file to the server—no further file modifications are required 😄.
{{< /alert >}}

## Packaging Applications

We often need to share Python programs we've written. However, sharing only the source code can be frustrating for users unfamiliar with coding, as running the source requires setting up a local environment. Hence, packaging tools come into play.

### pyinstaller

Installation is straightforward, just like any other Python package: `pip install pyinstaller`. To use it, navigate to the directory containing the target file in the terminal and run the command.

**Features: Fast packaging speed; larger resulting executable size.**

Common commands:

```shell
# Package main.py into a standalone executable; disable the console window when running.
pyinstaller -F -w main.py

# Package main.py into a project folder; enable the console window when running.
pyinstaller -D main.py
```

| Parameter                                   | Description                                                                 |
| ------------------------------------------ | -------------------------------------------------------------------------- |
| `-h, --help`                               | Show help message and exit.                                               |
| `-v, --version`                            | Show program version and exit.                                            |
| `-F, --onefile`                            | Package everything into a single executable.                              |
| `-D, --onedir`                             | Package everything into a directory (default).                            |
| `-w, --windowed, --noconsole`              | Disable the console window (Windows only).                                |
| `-c, --console, --nowindowed`              | Run the program with a console window (default, Windows only).            |
| `-a, --ascii`                              | Exclude Unicode character set support (included by default).              |
| `-d, --debug`                              | Generate a debug version of the executable.                               |
| `-n NAME, --name=NAME`                     | Specify the name of the generated executable or directory (default: script name). |
| `-o DIR, --out=DIR`                        | Specify the directory for the spec file (default: current directory).     |
| `-p DIR, --path=DIR`                       | Set the Python import path (similar to PYTHONPATH).                       |
| `-i <FILE>, --icon <FILE>`                 | Set the executable's icon (supports `.ico` or `.icns` formats).           |
| `--distpath DIR`                           | Specify the output directory for the executable (default: `./dist`).      |
| `--workpath WORKPATH`                      | Specify the directory for temporary files (default: `./build`).           |
| `--add-data <SRC;DEST or SRC:DEST>`        | Add additional data files or directories (Windows uses semicolons; Linux/OSX uses colons). |
| `--add-binary <SRC;DEST or SRC:DEST>`      | Add additional binary files.                                              |
| `--hidden-import MODULENAME`               | Add modules not automatically detected.                                   |
| `--exclude-module EXCLUDES`                | Exclude specified modules.                                                |
| `--clean`                                  | Clean PyInstaller cache and temporary files.                              |
| `--log-level LEVEL`                        | Set verbosity for console messages (options: TRACE, DEBUG, INFO, WARN, ERROR, FATAL). |

### Nuitka

Converts Python code into an exe executable by first translating Python to C and then compiling the C code.

**Features: Very slow packaging speed; requires a C compiler (though installation can be automated, it may not suit users with strict memory constraints); resulting executable is very small (about one-tenth the size of pyinstaller's output).**

Installation command:

```shell
pip install -U nuitka
```

Common usage commands:

```shell
# Package main.py into a single exe file with link-time optimization and clean up temporary files afterward.
python -m nuitka --lto=yes --remove-output --onefile main.py
```

| Parameter                                                                                       | Description                                                                        |
| ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `--standalone`                                                                                  | Creates a standalone executable with all dependencies.                             |
| `--onefile`                                                                                     | Packages everything into a single `.exe` file.                                     |
| `--optimize=N`                                                                                  | Sets optimization level (`0`, `1`, or `2`); higher numbers mean more optimization. |
| `--lto`                                                                                         | Enables Link Time Optimization (options: `no`, `yes`, or `thin`).                  |
| `--enable-plugin=<plugin_name>`                                                                 | Enables specified plugins (e.g., `tk-inter`, `numpy`, `anti-bloat`).               |
| `--output-dir=<dir>`                                                                            | Specifies the output directory for compilation.                                    |
| `--remove-output`                                                                               | Deletes intermediate `.c` files and other temporary files after compilation.       |
| `--nofollow-imports`                                                                            | Does not recursively process any imported modules.                                 |
| `--include-package=<package_name>`                                                              | Explicitly includes an entire package and its submodules.                          |
| `--include-module=<module_name>`                                                                | Explicitly includes a single module.                                               |
| `--follow-import-to=<module/package>`                                                           | Specifies modules or packages to process recursively.                              |
| `--nofollow-import-to=<module/package>`                                                         | Specifies modules or packages to exclude from recursive processing.                |
| `--include-data-files=<source>=<dest>`                                                          | Includes specified data files.                                                     |
| `--include-data-dir=<directory>`                                                                | Includes all data files in a directory.                                            |
| `--noinclude-data-files=<pattern>`                                                              | Excludes data files matching a pattern.                                            |
| `--windows-icon-from-ico=<path>`                                                                | Sets the icon for the Windows executable.                                          |
| `--company-name`, `--product-name`, `--file-version`, `--product-version`, `--file-description` | Sets properties for the Windows executable.                                        |

## References
- Poetry-related:
	- [Poetry Beginner's Guide_Using Poetry-CSDN Blog](https://blog.csdn.net/weixin_42871919/article/details/137125544) Very detailed resource.
	- [Poetry](https://python-poetry.org/)
	- [How to Change Poetry's Domestic Mirror - Data Science SourceResearch](https://www.resourch.com/archives/66.html)

- UV-related:
	- [Installing and Using Python Project and Package Manager UV - Deep Sea Xiao Tao](https://blog.xtao.de/380) One of the few blogs that includes the domestic mirror address for `uv python install`.

- Conda-related：
	- [Summary of Commonly Used Conda Commands - Zhihu](https://zhuanlan.zhihu.com/p/1902762958083298021)
