---
title: Ubuntu-OpenCV-CPP-Without-IDE
date: 2024-04-13 20:55:30
tags: OpenCV
---

Ubuntu OpenCV CMake

## 1.Building packages

1. update packages list

```sh
sudo apt update
```

2. Install building packages

```sh
sudo apt -y install build-essential
```

## 2.Install OpenCV from Ubuntu Repositories

[link](https://phoenixnap.com/kb/installing-opencv-on-ubuntu)

1. Install OpenCV

```sh
sudo apt -y install libopencv-dev
```

2. Check the OpenCV version

```sh
dpkg -l libopencv-dev
```

## 3.Install CMake

```sh
sudo apt -y install cmake
cmake --version
```

## 4.Create CPP Projetct

[link](https://docs.opencv.org/4.x/db/df5/tutorial_linux_gcc_cmake.html)

```sh
DisplayImage
├── build
│   └── 
├── CMakeLists.txt
├── cmake-modules
│   └── OpenCVConfig.cmake
├── img
│   └── lenna.jpeg
└── src
    └── DisplayImage.cpp
```

```cpp
// DisplayImage.cpp

#include <stdio.h>
#include <opencv2/opencv.hpp>

using namespace cv;

int main(int argc, char **argv)
{
    if (argc != 2)
    {
        printf("usage: DisplayImage.out <Image_Path>\n");
        return -1;
    }

    Mat image;
    image = imread(argv[1], IMREAD_COLOR);

    if (!image.data)
    {
        printf("No image data \n");
        return -1;
    }
    namedWindow("Display Image", WINDOW_AUTOSIZE);
    imshow("Display Image", image);

    waitKey(0);

    return 0;
}
```


```
# CMakeLists.txt

# CMake 最低版本号要求
cmake_minimum_required(VERSION 3.6)

# 项目信息
project(DisplayImage)

# 将cmake-modules文件夹中的文件加入 CMAKE_MODULE_PATH
list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake-modules")

# 查找并从外部项目加载设置
# REQUIRED如果找不到软件包，该选项将停止处理并显示一条错误消息
find_package(OpenCV REQUIRED)

# 将给定目录添加到编译器用来搜索包含文件的目录中。相对路径被解释为相对于当前源目录
include_directories(${OpenCV_INCLUDE_DIRS})

# 利用源码文件生成目标可执行程序
add_executable(DisplayImage src/DisplayImage.cpp)

# 添加链接库
target_link_libraries(DisplayImage ${OpenCV_LIBS})
```

## 5.Find OpenCV Config

[link](https://wiki.hanzheteng.com/development/cmake/cmake-find_package)

```sh
sudo find / -name "OpenCVConfig.cmake"
```

copy to cmake-modules folder

## 6.Build and Test

```sh
cd build
cmake ..
make
./DisplayImage ../img/lenna.jpeg
```

maybe need `chmod +x DisplayImage`

![test](Screenshot_test.png)


## 7.VSCode Debug

* Install Extensions

![install_Extensions](Screenshot_VSCode_Extensions.png)

* restart and set cmake

![set_cmake](Screenshot_set_cmake.png)

* add breakpoint and click Debug button

![breakpoint](Screenshot_set_breakpoint.png)

* Debugging

![run_debug](Screenshot_Debug.png)

* set pic path for main args

please tell me...


