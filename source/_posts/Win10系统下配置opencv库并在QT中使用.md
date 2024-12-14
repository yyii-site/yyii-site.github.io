---
title: Win10系统下配置opencv库并在QT中使用
date: 2023-08-06 22:51:34
tags: OpenCV
---

OpenCV Qt Windows10

1. 官网组装[https://opencv.org/releases/](https://opencv.org/releases/)下载Windows环境所使用的压缩包（包含源文件和编译好的库文件）

2. 将压缩包解压到本地文件夹，如G:\
   
   ![opencv_file_list](opencv_file_list.png)
   
3. 编辑WIndow系统环境变量，增加（G:\opencv\build\x64\vc15\bin）

4. qt新建界面应用工程，使用MSCV编译器。

5. qt项目文件夹处右击，选择添加库。

   ![qt_add_library](qt_add_library.png)

6. 外部库，下一步；

7. 库文件：G:/opencv/build/x64/vc15/lib/opencv_world345.lib；包含路径：G:/opencv/build/include；勾选debug版添加'd'；下一步，完成；软件会将配置信息添加到.Pro工程文件中；

   ![qt_set_project](qt_set_project.png)

8. 准备一张图片，注意图片名及路径仅可使用英文字符。如G:\opencv\demo.png

9. 修改代码mainwindow.cpp，注意文件路径

```c++
#include "mainwindow.h"
#include "ui_mainwindow.h"

#include <opencv2/highgui/highgui.hpp>
#include <opencv2/core/core.hpp>

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);

    cv::Mat image = cv::imread("G:/opencv/demo.png");
    cv::namedWindow("My Image");
    cv::imshow("window", image);
//    cv::waitKey(-1);
}

MainWindow::~MainWindow()
{
    delete ui;
}
```

10. 断点调试

点击Debug按钮开始调试，会有图片及界面弹出。增加断点验证调试功能是否正常。
