---
title: Qt创建C++动态库DLL
date: 2023-08-06 22:31:38
tags: Qt
---

## 1.创建C++动态库DLL

为了提高代码复用性，实现模块化开发，我们通常会对一些常用函数进行封装，通过调用动态链接库的方法实现，Qt自生便能构建共享库。

## 1.1新建共享库工程

* 新建工程，选择动态库，命名为DynamicLibrary，实现一个简单的方法(method) : int test()

DynamicLibrary_global.h

```C++
#ifndef DYNAMICLIBRARY_GLOBAL_H
#define DYNAMICLIBRARY_GLOBAL_H

#include <QtCore/qglobal.h>

#if defined(DYNAMICLIBRARY_LIBRARY)
#  define DYNAMICLIBRARY_EXPORT Q_DECL_EXPORT
#else
#  define DYNAMICLIBRARY_EXPORT Q_DECL_IMPORT
#endif

#endif // DYNAMICLIBRARY_GLOBAL_H
```



dynamiclibrary.h

```c++
#ifndef DYNAMICLIBRARY_H
#define DYNAMICLIBRARY_H

#include "DynamicLibrary_global.h"

class DYNAMICLIBRARY_EXPORT DynamicLibrary
{
public:
    DynamicLibrary();
    int test();
};

#endif // DYNAMICLIBRARY_H
```

dynamiclibrary.cpp

```C++
#include "dynamiclibrary.h"
#include <QDebug>

DynamicLibrary::DynamicLibrary()
{
}

int DynamicLibrary::test()
{
    qDebug() << "Hello Qt5";
    int sum = 0;
    for (int i=1; i<=100; i++) {
        sum = sum + i;
    }
    qDebug() << sum;
    return sum;
}
```

* Build(Debug) 得到动态库`DynamicLibrary.dll`

## 1.2调用动态库

* 新建一个控制台工程LoadDynamic，用于测试调用前面构建的动态库dll
* 导入相关头文件，先复制头文件至工程路径下(DynamicLibrary_global.h  dynamiclibrary.h)，然后项目单击右键添加现有文件。
* 链接库，配置pro文件，增加动态链接库的路径`LIBS += xxx\debug\DynamicLibrary.dll`
* Build(Debug) 得到可执行文件
* 运行可执行文件时需要系统找到对应的动态链接库，可以将原来生成的动态链接库拷贝至可执行文件同级目录下
* 其他依赖文件：开始-Qt-Qt命令行，输入windeployqt xxx\debug\LoadDynamic.exe
* 可执行文件路径处输入cmd，打开命令行输入LoadDynamic.exe运行
* ldd或depends检查依赖库情况