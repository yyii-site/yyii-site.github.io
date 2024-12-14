---
title: opencv交叉编译
date: 2024-02-25 21:48:14
tags: OpenCV
---

## 1.概述

Ubuntu x86 交叉编译 aarm64 OpenCV 3.4.5

[参考](https://www.codenong.com/cs105274123/)

## 2.环境

Ubuntu 18.04 aarm64    OpenCV3.4.5  目标arm64

源码

 OpenCV源码下载地址: https://opencv.org/releases/ 

安装交叉编译器

```bash
sudo apt-get install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```

安装cmake

```bash
sudo apt install cmake cmake-gui
```

解压OpenCV

## 3.使用cmake-gui生成Makefile

![img](20200402174348844.png) 

![img](20200402174507776.png) 

![img](20200402174708819.png) 

![img](20200402174747432.png) 

![img](20200402175311654.png) 

查看是否生成Makefile文件

## 4.编译

```bash
make && make install
```

用file指令查看编译出的可执行文件

```bash
file bin/opencv_version
```

将文件加install中生成的文件移动到aarm64文件夹，方面查找和使用。

## 5.编译例程

 在这个opencv-3.4.5/samples/cpp/example_cmake目录里官方已经给出了一个example可以拿来测试下，使用编译器编译，编译时加上OpenCV相关的库和头文件。 

```bash
g++ example.cpp -o aarm64 -I /home/yi/opencv/aarm64/include/ -L /home/yi/opencv/aarm64/lib/ -lopencv_calib3d -lopencv_objdetect -lopencv_core -lopencv_photo -lopencv_dnn -lopencv_shape -lopencv_features2d  -lopencv_stitching -lopencv_flann -lopencv_superres -lopencv_highgui -lopencv_videoio -lopencv_imgcodecs  -lopencv_video -lopencv_imgproc -lopencv_videostab -lopencv_ml
```

库、头文件和可执行文件移动到目标板（若cmake-gui选择编译平台为gnu，即可在本地测试无需拷贝文件）

## 6.设置动态库路径

```bash
export LD_LIBRARY_PATH=/home/yi/opencv/aarm64/lib/:$LD_LIBRARY_PATH
```

## 7.运行


```bash
./aarm64
```

软件运行界面

![1668051815828](1668051815828.png)