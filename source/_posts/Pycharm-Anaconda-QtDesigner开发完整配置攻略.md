---
title: Pycharm+Anaconda+QtDesigner开发完整配置攻略
date: 2024-02-12 22:38:31
tags: PyQt
---

搭建QyQt开发环境

PyCharm + Anaconda(或miniconda) + Qt Designer

## 1.PyCharm下载安装
pycharm 社区版

## 2.Anaconda下载安装

[anaconda](https://repo.anaconda.com/archive/Anaconda3-2021.05-Windows-x86_64.exe)

anaconda mirror

## 3.设置pypi源

pycharm project

```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

## 4.安装软件包

```
pip install -r requirements.txt
```

## 5.查看python版本

```
C:\Users\替换为用户名>pip -V
pip 21.0.1 from E:\ProgramData\Anaconda3\lib\site-packages\pip (python 3.8)
```

## 6.conda 添加中科大源
[https://blog.csdn.net/R18830287035/article/details/90633942](https://blog.csdn.net/R18830287035/article/details/90633942)

```
C:\Users\替换为用户名>
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/
conda config --set show_channel_urls yes
```

```
C:\Users\替换为用户名>
conda config --show channels
channels:
  - https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
  - defaults
```

## 7.PyCharm+Qt Designer+PyUIC安装配置教程

[https://www.cnblogs.com/lsdb/p/9121903.html](https://www.cnblogs.com/lsdb/p/9121903.html)


## 8.PyCharm Setting

```
Tools - External Tools
  Qt-QtDesigner 
    Name: QtDesigner
    Group: Qt
    Program: E:\ProgramData\Anaconda3\envs\py37ccd\Lib\site-packages\PySide2\designer.exe
    Working dir: $FileDir$
  Qt-PyUIC
    Name: PyUIC
    Group: Qt
    Program: E:\ProgramData\Anaconda3\envs\py37ccd\python.exe
    Arguments: -m PyQt5.uic.pyuic $FileName$ -o $FileNameWithoutExtension$.py
    Working dir: $FileDir$
```

## 9.PyCharm+QTDesigner+PyUIC使用教程
https://www.cnblogs.com/lsdb/p/9122425.html
