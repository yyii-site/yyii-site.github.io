---
title: Ubuntu安装QT交叉编译环境
date: 2024-02-25 21:24:00
tags: Qt
---

## 1.Ubuntu 安装QT交叉编译环境

[参考](https://blog.csdn.net/guohuaqu/article/details/109519744)

需求实现再x86机器下完成对ARM64平台的程序开发

## 2.安装ARM64编译器

```bash
sudo apt-get install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu
```
查看

![shell aarch64-linux-gnu-](1667790599228.png)

## 3.安装qt开发环境

qt安装包：qt-opensource-linux-x64-5.12.12.run

sudo chmod +x，然后运行，就会出现图形界面，按步骤一步步安装即可。

## 4.安装qt工具

ARM64交叉编译器，例如qmake、库等

下载qt源码包：[qt-everywhere-src-5.12.12.tar.xz](http://download.qt.io/archive/qt/5.14/5.12.12/single/qt-everywhere-src-5.12.12.tar.xz) 

解压 xz -d qt-everywhere-src-5.12.12.tar.xz

修改编译配置文件

```bash
yi@linux:~/Downloads/qt-everywhere-src-5.12.12/qtbase/mkspecs$ ls
aix-g++                macx-xcode
aix-g++-64             modules
android-clang          modules-inst
android-g++            netbsd-g++
common                 openbsd-g++
cygwin-g++             qconfig.pri
darwin-g++             qdevice.pri
devices                qmodule.pri
dummy                  qnx-aarch64le-qcc
features               qnx-armle-v7-qcc
freebsd-clang          qnx-x86-64-qcc
freebsd-g++            qnx-x86-qcc
haiku-g++              solaris-cc
hpuxi-g++-64           solaris-cc-64
hurd-g++               solaris-cc-64-stlport
integrity-armv7        solaris-cc-stlport
integrity-armv7-imx6   solaris-g++
integrity-armv8-rcar   solaris-g++-64
integrity-x86          unsupported
linux-aarch64-gnu-g++  wasm-emscripten
linux-arm-gnueabi-g++  win32-arm64-msvc2017
linux-clang            win32-clang-g++
linux-clang-libc++     win32-clang-msvc
linux-g++              win32-g++
linux-g++-32           win32-icc
linux-g++-64           win32-icc-k1om
linux-icc              win32-msvc
linux-icc-32           winrt-arm64-msvc2017
linux-icc-64           winrt-arm64-msvc2019
linux-icc-k1om         winrt-arm-msvc2015
linux-llvm             winrt-arm-msvc2017
linux-lsb-g++          winrt-arm-msvc2019
lynxos-g++             winrt-x64-msvc2015
macx-clang             winrt-x64-msvc2017
macx-g++               winrt-x64-msvc2019
macx-icc               winrt-x86-msvc2015
macx-ios-clang         winrt-x86-msvc2017
macx-tvos-clang        winrt-x86-msvc2019
```

查看修改配置文件 vim linux-aarch64-gnu-g++/qmake.conf

```bash
#
# qmake configuration for building with aarch64-linux-gnu-g++
#

MAKEFILE_GENERATOR      = UNIX
CONFIG                 += incremental
QMAKE_INCREMENTAL_STYLE = sublib

include(../common/linux.conf)
include(../common/gcc-base-unix.conf)
include(../common/g++-unix.conf)

# modifications to g++.conf
QMAKE_CC                = aarch64-linux-gnu-gcc
QMAKE_CXX               = aarch64-linux-gnu-g++
QMAKE_LINK              = aarch64-linux-gnu-g++
QMAKE_LINK_SHLIB        = aarch64-linux-gnu-g++

# modifications to linux.conf
QMAKE_AR                = aarch64-linux-gnu-ar cqs
QMAKE_OBJCOPY           = aarch64-linux-gnu-objcopy
QMAKE_NM                = aarch64-linux-gnu-nm -P
QMAKE_STRIP             = aarch64-linux-gnu-strip
load(qt_config)
```

开始编译：

 进入根目录cd qt-everywhere-src-5.12.12 

```bash
./configure -prefix /home/yi/Qt5.12.12/5.12.12/aarch64 -make libs -xplatform linux-aarch64-gnu-g++ -no-opengl -skip qtdeclarative
```

-prefix 代表你的安装文件夹

-xplatform 代表你所制定的编译器

no-opengl 跳过编译openGL(因为我已经安装了Qt，所以不需要界面)

-skip qtdeclarative 跳过 qtdeclarative（不跳过，编译出错，出错的原因不清楚）

线程开到了最大，并把log存在了本地。编译具体时间比较依赖自己PC的配置。

```bash
make -j16 2>&1 | tee build.log
```
创建安装文件夹

```bash
Mkdir -p Qt5.12.12/5.12.12/aarch64
```
安装

```bash
make install
```
查看结果

```bash
yi@linux:~/Qt5.12.12/5.12.12/aarch64$ ls
bin  doc  include  lib  mkspecs  plugins  translations
```
## 5.配置Qt，添加交叉编译环境

4.配置Qt，添加交叉编译环境

a.打开Qt

b.tools → options,打开面板

c.设置compilers,手动添加ARM64交叉编译器

add c

![commpilers](commpilers.png)


d.手动add qt version

![qt kits-Versions](1667797779321.png)

e.手动add Kits

![qt kits-Manual](1667797929755.png)

![qt Debugger](1667797958269.png)

## 6.新建工程&编译

选择刚刚手动添加的kits

