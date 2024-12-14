---
title: Windows系统中使用CMake配置OpenCV的CPP环境
date: 2024-04-18 20:27:32
tags: OpenCV
---

Windows CMake OpenCV C++

## 1.概述

Windows系统中使用Cmake配置OpenCV的C++环境

参考[Link](https://www.youtube.com/watch?v=CnXUTG9XYGI)

## 2.下载并解压 [opencv](https://opencv.org/releases/)

选择Windows，不要选择Sources

## 3.下载并安装cmake

省略...

## 4.编辑系统环境变量

在Path 增加以下三项(根据安装位置修改路径)

```
C:\Program Files\CMake\bin
D:\opencv\build\x64\vc15\bin
D:\opencv\build\x64\vc15\lib
```

## 5.新建文件夹及内部的文件
```
HelloOpenCV
└─src
    └─main.cpp
CMakeLists.txt
1.jpg
```

```cpp
//main.cpp
#include <opencv2/opencv.hpp>
#include <iostream>

int main()
{
    std::string imagePath = "1.jpg";
    cv::Mat image = cv::imread(imagePath, cv::IMREAD_COLOR);
    cv::imshow("Display window", image);
    cv::waitKey(0);
    return 0;
}
```

根据安装位置修改 `OpenCV_DIR` 的值
```CMake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.0)
project(HelloOpenCV)

set(OpenCV_DIR "D:/opencv/build/x64/vc15/lib")

find_package(OpenCV REQUIRED)

include_directories(${OpenCV_INCLUDE_DIRS})

add_executable(HelloOpenCV src/main.cpp)

target_link_libraries(HelloOpenCV ${OpenCV_LIBS})
```

## 6.Terminal 中编译

```PowerShell
mkdir build
cmake -B .\build\
```

Debug
```PowerShell
cmake --build .\build\
.\build\Debug\HelloOpenCV.exe
```

Release
```PowerShell
cmake --build .\build\ --config Release
.\build\Release\HelloOpenCV.exe
```