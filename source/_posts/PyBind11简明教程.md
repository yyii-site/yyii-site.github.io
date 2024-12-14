---
title: PyBind11简明教程
date: 2024-04-21 21:40:59
tags: Python
---

python通过PyBind11调用cpp

## 1.概述

原文链接: [How to Call C++ from Python](https://www.matecdev.com/posts/cpp-call-from-python.html)

翻译整理: [BimAnt](http://www.bimant.com/blog/pybind11-crash-tutorial/)

从 Python 调用 C++ 基本上有两种方法：使用 PyBind11 C++ 库生成 Python 模块，或使用 cytpes Python 包访问已编译的共享库。 使用 PyBind11 我们可以更轻松地共享许多数据类型，而使用 ctypes 是一种低级 C 风格的解决方案。

就我而言，我希望能够利用 C++ 的性能和可移植性，但我不想放弃解释语言的交互性以进行快速探索和调试。

幸运的是，从 Python 调用 C++ 并不像乍看起来那么困难。 这样，我们就可以在开发 C++ 代码的同时掌握 Python 的一些交互性。

就我而言，我想使用 Python 来：

* 将一些问题参数传递给C++
* 调用 C++ 代码来运行计算密集型例程
* 检索最终结果以及一些用于调试的中间计算。
* 以交互方式探索结果，并生成图表和报告。

使用 ctypes 的问题是共享许多数据类型需要大量的低级解决方法。 例如，虽然 ctypes 不支持复数等基本内容，但 PyBind11 使 Numpy 能够与 Eigen 完全互操作，并且需要最少的代码。

不过，我也发现了 PyBind11 的一个小问题。 事实证明，重新编译 C++ 代码并尝试重新加载 PyBind 生成的 Python 模块后，什么也没发生。 重新加载已编译模块的唯一方法是重新启动我的 Python 会话。 无论如何，这不是什么大问题，因为 Python 的启动时间可以忽略不计。 也许，这个步骤可以在 IDE 级别自动化。

因此，现在的问题是如何充分利用 PyBind11。

## 2.与 PyBind11 共享 C++ 类

PyBind11 的官方文档非常好，我能够毫无问题地开始使用它。 然而，我想分享这个库的超级快速入门指南，以及我打算如何使用它。

Pybind11 是一个仅包含头文件的库，你可以通过以下方式获取它：

`pip install pybind11`

虽然没有必要将所有 C++ 代码构造为类，但如果你有一个要在 C++ 和 Python 之间共享的类，Pybind11 将使事情变得非常容易。 实际上，我更像是一个struct向量类型的人，我总是想在给定的项目中引入最少数量的类。

然而，在这种情况下，我发现使用外观设计模式（参见 wiki）可以同时带来非常简单的 Python/C++ 互操作性和良好的 API。

所以，我设计了一个简单的类。 它基本上包含：

* 读取问题参数的构造函数。
* 执行计算的 run() 函数。
* 一些 Eigen 数组作为公共变量来存储结果。

这是我的最小示例：

```h
// mylib.h
#include <Eigen/Dense>
#include <cmath>

using Eigen::Matrix, Eigen::Dynamic;
typedef Matrix<std::complex<double>, Eigen::Dynamic, Eigen::Dynamic> myMatrix;

class MyClass {

    int N;
    double a;
    double b;

public:

    Eigen::VectorXd v_data;
    Eigen::VectorXd v_gamma;

    MyClass(){}
    MyClass( double a_in, double b_in, int N_in) 
    {
        N = N_in;
        a = a_in;
        b = b_in;
    }

    void run() 
    { 
        v_data = Eigen::VectorXd::LinSpaced(N, a, b); 

        auto gammafunc = [](double it) { return std::tgamma(it); };
        v_gamma = v_data.unaryExpr(gammafunc);
    }
};
```

为了共享这个类，我们需要添加一些 C++ 代码。 我倾向于在一个单独的文件中执行此操作，其中包含创建 python 包装器所需的所有内容：

```cpp
// pywrap.cpp
#include <pybind11/pybind11.h>
#include <pybind11/eigen.h>
#include "mylib.h"

namespace py = pybind11;
constexpr auto byref = py::return_value_policy::reference_internal;

PYBIND11_MODULE(MyLib, m) {
    m.doc() = "optional module docstring";

    py::class_<MyClass>(m, "MyClass")
    .def(py::init<double, double, int>())  
    .def("run", &MyClass::run, py::call_guard<py::gil_scoped_release>())
    .def_readonly("v_data", &MyClass::v_data, byref)
    .def_readonly("v_gamma", &MyClass::v_gamma, byref)
    ;
}
```

有几点需要强调：

* 类构造函数签名由 .def(py::init<int, double, double>())  指定
* 对于 run() 函数，我们要求释放 GIL（全局解释器锁），这将阻止我们的函数使用多个线程。
* 最后，可以使用以下 CMakeLists.txt 文件进行编译：

```CMakeLists
cmake_minimum_required(VERSION 3.10)

project(MyLib)
set(CMAKE_CXX_STANDARD 20)
set(PYBIND11_PYTHON_VERSION 3.6)
set(CMAKE_CXX_FLAGS "-Wall -Wextra -fPIC")

find_package(pybind11 REQUIRED)
find_package(Eigen3 REQUIRED)

pybind11_add_module(${PROJECT_NAME} pywrap.cpp)

target_compile_definitions(${PROJECT_NAME} PRIVATE VERSION_INFO=${EXAMPLE_VERSION_INFO})
target_include_directories(${PROJECT_NAME} PRIVATE ${PYBIND11_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE Eigen3::Eigen)
```
现在已准备好了。 如果你使用 VS Code，配置 CMake 扩展后，只需按 F7 即可编译 C++ 库。

## 3.从 Python 调用 C++ 库

这个过程非常简单，并且应该开箱即用。 然而，有一些步骤可以优化交互式工作流程，这些步骤稍微棘手，但也值得实施。

例如，如果正在执行 Python 环境并且编译的库进入构建目录，可以执行以下操作：

import sys
sys.path.append("build/")
from MyLib import MyClass

import matplotlib.pyplot as plt

Simulation = MyClass(-4,4,1000)
Simulation.run()

plt.plot(Simulation.v_data, Simulation.v_gamma, \
"--", linewidth = 3, color=(1,0,0,0.6),label="Function Value")
plt.ylim(-10,10)
plt.xlabel("x")
plt.ylabel("($f(x) = \gamma(x)$)")
plt.title("(Gamma Function: $\gamma(z) = \int_0^\infty x^{z-1} e^{-x} dx$)",fontsize = 18);
plt.show()

结果如下：

![Gamma](image-137.png)

请注意，特征向量会自动转换为 Python 数组。

修改 myLib.hpp 后，只需在 pywrap.cpp 中为我们要公开的每个新函数或变量添加一行代码。

不幸的是，这不会导致完全交互式的工作流程。 当你在更改后重新编译 C++ 代码时，Python 端不会发生任何事情。 即使尝试使用 importtools 重新加载 Python 模块：

```sh
import importlib
importlib.reload(MyLib)
```

什么都没发生。 原因是编译后的代码无法在Python中重新加载。

因此，使用 PyBind11 时，每次重新编译 C++ 代码时都需要重新启动 Python 会话，这对于开发目的来说有点烦人。 然而，这是一个很小的代价，因为 Python 的启动时间可以忽略不计，并且可能有一种方法可以使用一些 IDE 热键或其他工具来自动化该过程。

## 4.结束语

好了，这就是可以轻松地从 Python 调用 C++ 库的方法。教程的示例代码可以从github获取。

特别是，这个两步过程可以产生非常交互式的开发工作流程。 尽管我们有一个编辑-编译-运行工作流程，但我们在最后添加了一个解释器，所以现在我们的工作流程看起来像编辑-编译-运行-探索。

将来，我计划将两个功能合并到此工作流程中：

* 第一个是 C++20 模块，它应该可以加快大型 C++ 项目的编译时间。 不幸的是，CMake 仍然与模块不兼容（请参阅此问题以获取更新），并且显然必须依赖像 Ninja-Build 这样的构建系统才能立即使用此功能。
* 另一件事是修复重新编译 C++ 代码后（手动）重新启动 Python 会话的需要。 为此，我希望也许可以在 VSCode 级别对此采取一些措施。 到目前为止，VS Code 中的最佳选项似乎是终止 Python 会话，然后使用 Shift+Enter 执行 Python 代码，如果尚未打开会话，则会创建一个新会话。