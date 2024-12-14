---
title: PyBind11应用
date: 2024-11-01 21:43:51
tags: Python C/C++
---

## 问题

网上思岚激光雷达的应用教程一上来就是基于ROS2系统的。这种方法可以通过GUI方式查看测试数据，直观又强大。

但是如果想环境和依赖简单一点，硬件资源占用少一点，可以用思岚提供的SDK。[SDK](https://github.com/slamtec/rplidar_sdk)是以持C++源代码的方式提供的，自带例程。

项目需要python读取的雷达数据。一开始是由docker中的ROS2作为数据处理和通讯的中转站。但发现占用资源多，操作有点复杂。

转而考虑python直接调用sdk, 由于头文件非常多ctypes方式没怎么尝试就放弃了。中途用qt6调sdk然后通过UDP传给python，虽然数据通了，但部署得交叉编译还得依赖qt6. 性能也有损失。

最后就用到主角 pybind11 (blog中搜索 **PyBind11简明教程**) 和 [外观设计模式](https://en.wikipedia.org/wiki/Facade_pattern) *实现 python 调用 cpp 的 sdk*

## 工程目录

```bash
├── build
├── CMakeLists.txt
├── main.cpp
├── mylib.h
├── pywrap.cpp
├── readme.md
├── sdk
│   ├── CMakeLists.txt
│   ├── include
│   └── src
└── test.py
```

## 封装SDK

外观设计模式简单来说就是把零散复杂的SDK根据使用流程做一个抽象和封装。

让别人调用的时候不再关注流程和操作上的技术细节。

比如此处的mylib.h (如果将类的实现分开写道单独的如mylib.cpp文件中，python调用的时候会报错，好像是CPP编译器给函数名增加了一些信息。)

注意：
* 本项目仅用的串口，网络通讯方式已被省略，如有需要请参考SDK例程
* 由于udev已经将雷达串口映射为 "/dev/rplidar" 波特率100000 请自行修改。

```hpp
#ifndef __MY_RADAR
#define __MY_RADAR

#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <string.h>

#include <vector>
#include <pybind11/pybind11.h>
#include <pybind11/stl.h> // 包含 STL 支持
namespace py = pybind11;

#include "sl_lidar.h"
#include "sl_lidar_driver.h"
#ifndef _countof
#define _countof(_Array) (int)(sizeof(_Array) / sizeof(_Array[0]))
#endif

#ifdef _WIN32
#include <Windows.h>
#define delay(x)   ::Sleep(x)
#else
#include <unistd.h>
static inline void delay(sl_word_size_t ms) {
    while (ms >= 1000) {
        usleep(1000 * 1000);
        ms -= 1000;
    };
    if (ms != 0)
        usleep(ms * 1000);
}
#endif

using namespace sl;


class MyClass {

public:    
    MyClass(int com)
    {
        IChannel* _channel;
        sl_result     op_result;

        std::cout << "init" << std::endl;

        drv = *createLidarDriver();
        if (!drv) {
            fprintf(stderr, "insufficent memory, exit\n");
            exit(-2);
        }
        sl_lidar_response_device_info_t devinfo;
        bool connectSuccess = false;

        _channel = (*createSerialPortChannel("/dev/rplidar", 1000000));
        if (SL_IS_OK((drv)->connect(_channel))) {
            op_result = drv->getDeviceInfo(devinfo);

            if (SL_IS_OK(op_result))
            {
                connectSuccess = true;
            }
            else {
                delete drv;
                drv = NULL;
            }
        }

        if (!connectSuccess) {
            fprintf(stderr, "Error, cannot bind to the specified serial port.\n");

            goto on_finished;
        }

        printf("SLAMTEC LIDAR S/N: ");
        for (int pos = 0; pos < 16; ++pos) {
            printf("%02X", devinfo.serialnum[pos]);
        }

        printf("\n"
            "Firmware Ver: %d.%02d\n"
            "Hardware Rev: %d\n"
            , devinfo.firmware_version >> 8
            , devinfo.firmware_version & 0xFF
            , (int)devinfo.hardware_version);

        // check health...
        if (!checkSLAMTECLIDARHealth(drv)) {
            goto on_finished;
        }

        drv->setMotorSpeed();
        drv->startScan(0, 1);

    on_finished:
        return;
    };

    ~MyClass() {
        std::cout << "uninit" << std::endl;
        drv->stop();
        delay(200);
        if (drv) {
            drv->setMotorSpeed(0);
            delete drv;
            drv = NULL;
        }
    };

    void run(void) {
        std::cout << "run" << std::endl;
        if (drv) {
            sl_lidar_response_measurement_node_hq_t nodes[8192];
            size_t   count = _countof(nodes);

            sl_result op_result = drv->grabScanDataHq(nodes, count);

            if (SL_IS_OK(op_result)) {
                drv->ascendScanData(nodes, count);
                if (data_angle.size() != count)
                {
                    data_angle.resize(count);
                    data_dist.resize(count);
                    data_quality.resize(count);
                }

                for (int pos = 0; pos < (int)count; ++pos) {
                    // printf("%s theta: %03.2f Dist: %08.2f Q: %d \n",
                    //     (nodes[pos].flag & SL_LIDAR_RESP_HQ_FLAG_SYNCBIT) ? "S " : "  ",
                    //     (nodes[pos].angle_z_q14 * 90.f) / 16384.f,
                    //     nodes[pos].dist_mm_q2 / 4.0f,
                    //     nodes[pos].quality >> SL_LIDAR_RESP_MEASUREMENT_QUALITY_SHIFT);
                    data_angle.at(pos) = (nodes[pos].angle_z_q14 * 90.f) / 16384.f;
                    data_dist.at(pos) = nodes[pos].dist_mm_q2 / 4.0f;
                    data_quality.at(pos) = nodes[pos].quality >> SL_LIDAR_RESP_MEASUREMENT_QUALITY_SHIFT;
                }
            }
        }
    };

    const std::vector<double>& get_angle() const { return data_angle;}
    const std::vector<double>& get_dist() const { return data_dist;}
    const std::vector<int>& get_quality() const { return data_quality;}

private:
    std::vector<double> data_angle;
    std::vector<double> data_dist;
    std::vector<int>    data_quality;

    ILidarDriver* drv;

    bool checkSLAMTECLIDARHealth(ILidarDriver* drv)
    {
        sl_result     op_result;
        sl_lidar_response_device_health_t healthinfo;

        op_result = drv->getHealth(healthinfo);
        if (SL_IS_OK(op_result)) { // the macro IS_OK is the preperred way to judge whether the operation is succeed.
            printf("SLAMTEC Lidar health status : %d\n", healthinfo.status);
            if (healthinfo.status == SL_LIDAR_STATUS_ERROR) {
                fprintf(stderr, "Error, slamtec lidar internal error detected. Please reboot the device to retry.\n");
                // enable the following code if you want slamtec lidar to be reboot by software
                // drv->reset();
                return false;
            }
            else {
                return true;
            }

        }
        else {
            fprintf(stderr, "Error, cannot retrieve the lidar health code: %x\n", op_result);
            return false;
        }
    }
};


#endif
```

虽然内容很多，但对于 python 上层调用方来讲。就只有

* 构造函数：雷达初始化，完成准备工作
* run函数：串口将雷达数据保存到私有变量
* get_angle/get_dist/data_quality等数据传递函数：从私有变量将数据传递给python
* 析构函数：关闭雷达释放资源

这也是上面提到的 **外观设计模式**

## 创建 python 包装器所需的内容

照猫画虎实现的 pywrap.cpp 文件

```cpp
#include <pybind11/pybind11.h>
#include "mylib.h"

namespace py = pybind11;
constexpr auto byref = py::return_value_policy::reference_internal;

PYBIND11_MODULE(MyLib, m) {
    m.doc() = "optional module docstring";

    py::class_<MyClass>(m, "MyClass")
    .def(py::init<int>())
    .def("run", &MyClass::run, py::call_guard<py::gil_scoped_release>())
    .def("get_angle", &MyClass::get_angle)
    .def("get_dist", &MyClass::get_dist)
    .def("get_quality", &MyClass::get_quality)
    ;
}
```

## 前期调试

此文件仅用于前期验证库文件功能是否正常

main.cpp

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <string.h>

#include "sl_lidar.h" 
#include "sl_lidar_driver.h"

#include "mylib.h"


int main(int argc, const char* argv[]) {
    MyClass radar(1);
    radar.run();
    return 0;
}
```

## 编译生成库文件

安装 pybind11。安装方式除了下面，还有 pip 和 conda 等

```bash
sudo apt install pybind11-dev
```

将 sdk 和 pybing11 教程中的 cmake 文件结合到一起（注意pybind11_DIR 文件目录根据自己的实际情况修改）

```cmake
cmake_minimum_required(VERSION 3.10)

project(MyLib)

set(CMAKE_CXX_STANDARD 11)
set(PYBIND11_PYTHON_VERSION 3.6)
set(CMAKE_CXX_FLAGS "-Wall -Wextra -fPIC")

set(pybind11_DIR "/usr/local/lib/python3.6/dist-packages/pybind11/share/cmake/pybind11")
find_package(pybind11 REQUIRED)
find_package(Threads REQUIRED)

# 静态库生成的路径
set(LIB_PATH ${CMAKE_CURRENT_SOURCE_DIR}/lib)
# 第三方库头文件路径
list(APPEND HEAD_PATH ${CMAKE_CURRENT_SOURCE_DIR}/sdk/include)
list(APPEND HEAD_PATH ${CMAKE_CURRENT_SOURCE_DIR}/sdk/src)
# 静态库的名字
set(RADRA_LIB "rplidar_driver")

# 添加子目录
add_subdirectory(sdk)

include_directories(${HEAD_PATH})

link_directories(${LIB_PATH})
# link_libraries(${RPLIDAR_LIB})

pybind11_add_module(${PROJECT_NAME} pywrap.cpp)

# exe 前期编写main.cpp调库调试功能，后面生成库供python调用（exe 和 so 不能同时启用）
# file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# add_executable(app ${SRC_LIST})
# target_link_libraries(app ${RADRA_LIB} Threads::Threads)

# so
target_compile_definitions(${PROJECT_NAME} PRIVATE VERSION_INFO=${EXAMPLE_VERSION_INFO})
target_include_directories(${PROJECT_NAME} PRIVATE ${PYBIND11_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE ${RADRA_LIB} Threads::Threads)
```

编译库

```bash
mkdir build
cd build
cmake ..
make
```

顺利的话会生成类似 `MyLib.cpython-36m-aarch64-linux-gnu.so` 文件

## python 调用

test.py
```python
import sys
import numpy as np
sys.path.append("build/")
from MyLib import MyClass

radar = MyClass(1)

while True:
    radar.run()
    l_angle = radar.get_angle()
    l_dist = radar.get_dist()
    l_quality = radar.get_quality()
    print(l_angle)
```

测试

```bash
python3 test.py
```
可以按 `Ctrl+C` 停止