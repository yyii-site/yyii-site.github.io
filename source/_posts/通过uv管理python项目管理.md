---
title: 通过uv管理python项目管理
date: 2025-12-22 22:30:11
tags: Python
---

## 纯手工管理项目
在项目文件夹中

创建python运行环境
```python
python -m venv .venv
```

激活环境
```python
source .venv/bin/activate
```

编辑项目管理文件，其中包含依赖声明
```python
edit pyproject.toml
```

安装依赖 并把自身安装到依赖中，并创建源码的链接
```python
pip install -e .
```

如果使用`pip install .`会把项目打包成一个标准软件包并安装这个软件包，但这样会造成 .venv 中和项目中都有一份项目源代码，所以要加 -e 选项。

传统方式运行代码

```python
source .venv/bin/activate
python main.py
```

导出项目中安装的包（会导出直接和间接依赖包，很难管理和维护）

```bash
pip frzzeen > requirements.txt
```



## 使用uv指令

编辑pyproject.toml 基本信息（仅需包含项目名称和版本号）
```toml
[project]
name = "uv_test"
version = "0.1.0"
```

下面的代码会将包信息加入 dependencies 列表，检查并自动创建 `.venv`虚拟环境，并将软件包和依赖安装到虚拟环境
```python
uv add flask
```

如果是别人的项目，只需要执行下面的代码即可自动读取 `pyproject.toml` 并配置好环境
```python
uv sync
```

uv 运行代码（在虚拟环境上下文中执行代码）
```python
uv run main.py
```



## 专门学习使用uv

https://youtu.be/aVXs8lb7i9U?si=zoLFH2cVCGGLH8II

要让一段程序运行需要解决两个问题：

* 确定Python的版本 
* 解决依赖

### python版本呢

查看uv 支持的python版本

```bash
uv python list
```

安装需要的版本

```bash
uv python install cpython-3.12
```

使用指定python版本运行脚本

```bash
uv run -p 3.12 main.py
```

如果使用交互版本

```bash
uv run -p 3.12 python
```



### 依赖

直接创建示例工程

```bash
uv init -p 3.13
```

可以直接在.python-version 文件中修改python的版本，也可在 pyproject.toml 中定义 `requires-python = ">=3.10,<3.13"`

安装指定的依赖包，此时uv不仅会下载库也会为工程创建虚拟环境 .venv

```bash
uv add flask
```

可以在vscode右下角选择我们的虚拟环境，vscode中的错误也消失了。

运行脚本

```
uv run main.py
```

打印依赖树

```
uv tree
```

### 检查工具

使用 ruff 检查代码是否符合规范

安装方式1：

```
uv add ruff --dev
```

`--dev` 是指在开发过程中使用，不要打包到软件中

使用：

```
ruff check
```

删除依赖或软件包

```
uv remove ruff --dev
```



方式2：

```
uv tool install ruff
```

将 ruff 安装到系统中 （查看路径 which ruff）

查看已经安装的工具

```
uv tool list
```



### 打包

将项目打包成一个可以运行的脚本

在 `pyproject.toml` 中加入一个  `project.scripts` 定义 `脚本名称 = 脚本:函数名`

```
[project.scripts]
ai = “ai:main”
```

运行打包命令，把整个工程打包成 .whl 文件

```
uv build
```

whl 文件就可以发布到软件仓库让别人使用

```
uv tool install xxx
或者
uv add xxx
```

本地安装

```
uv tool install dist/test-0.1.0-py3-none-any.whl
```

现在即可直接在终端使用脚本名称运行软件了

```
ai xxx
```



### 其他工具

使用 uv tool 安装的脚本都拥有自己的虚拟环境，不依赖开发环境或者系统环境

uv tool run ruff 也可以使用 uvx ruff

单元测试 pytest 

mock 库


