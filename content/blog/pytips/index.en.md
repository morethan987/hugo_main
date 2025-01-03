---
title: Python Tpis
weight: 15
draft: false
description: A set of Python tips
slug: pythontips
tags:
  - Python
series:
  - Operation
series_order: 2
date: 2024-08-10
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
D:\Python\Python311\python.exe -m venv your_env_name
```

More parameters you may need for a customized virtual env. 🤔

| Params                   | Meaning                                                                                             |
| ------------------------ | --------------------------------------------------------------------------------------------------- |
| `--system-site-packages` | Give the virtual environment access to the system site-packages dir.                                |
| `--clear`                | Delete the contents of the environment directory if it already exists, before environment creation. |
| `--version`              | print the python version of the env                                                                 |

{{< alert >}}
All the detailed expaination of the parameters can be got by the code `python -m venv -h`. No need to search everywhere~😆
{{< /alert >}}

### Activate
The virtual env is defaulted not active. In the direction `your_env_name/Scripts/` will be a file named `activate`. Run it with your cmd.

```sh
# activate virtual env
your_env_name/Scripts/activate
```
