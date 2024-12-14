---
title: CPP学习笔记
date: 2024-09-06 20:56:14
tags: CPP
---

视频教程： [youtube 侯捷](https://www.youtube.com/watch?v=2S-tJaPKFdQ&list=PL-X74YXt4LVZ137kKM5dNfCIC4tsScerb)

## P1

类分两种，思考方式完全不同：

**无指针**:多半不用写析构函数
class without pointer members (Complex 复数)

**有指针**:要写析构函数
class with pointer members (String 字符串)


类之间的关系：

**继承**inheritance、**复合**composition、**委托**delegation

单一class:     Object Based：   基于对象

class间有关联：Object Oriented: 面向对象

c++(C++语言 C++标准库) 推荐书籍

## P2

c 变量为全局

c++ 变量和对象绑定(c++中的class和struct区别很小)

c   无指针 数据多个地方 函数一份

c++ 有指针 数据是一个指针 函数一份 具体数据多个地方

c   标准库

`include <stdio.h> == <cstdio>`

c++ 标准库 (include 可以不带.h后缀)

`include <iostream.h> == <iostream>`

标准库 `std::cout <<` 接收基本类型，其他复杂类型需要自己写符号重载函数

```cpp
.h 防卫式声明
#ifndef __COMPLEX__
#define __COMPLEX__
...
#endif
```

一个类满足多种数据类型 使用模板 在使用的时候来确定

`template<typename T>`

类里面使用T来代替未确定类型的变量，以后使用：

```cpp
complex<double> c1(2.5, 1.5);
complex<int> c2(2, 6);
```

## P3

inline 函数 快 放置在class 的 {body} 中，或者函数前加 inline。 复杂的函数即使写inline也没用 全部写inline也没关系

类使用的三种方式:
```cpp
complex c1(1,2);
complex c2;
complex *c3 = new complex(1, 2);
```

```cpp
new delete
array new delete array //需要调用多次析构函数 不然只会调用一次析构函数
```

字符数组

```cpp
class complex
{
public:
    complex (double r=0, double i=0)
        : re (r), im (i)
    {}
    comple& operator += (const complex&);
    double real () const {return re;}
    double imag () const {return im;}
private:
    double re im;
    
    friend complex& __doapl (comple*, const complex&);
};
```

数据全部放到private中，函数主要放在public，如果不想让别人调用就放到private中。

成员函数 非成员函数

重载 函数的参数一定不能一样 编译器看到的函数实际为（函数名+参数） 注意参数默认值的问题

`complex (double r = 0, double i = 0) : re (r), im(i){}`

构造函数：1默认值 2初始化列表（仅构造函数有此功能） 3多态
构造函数一定返回的是一个对象，故不需要写返回值

## P4

把构造函数写在private中，表示不能被外部调用，即不能实现类的声明和定义。

有种设计模式Singleton，利用将构造函数写到private中来实现仅会创建一个对象的功能。

`double real () const {return re;}`
若函数不会去改变类中的数据，需要在函数()和{}中间加 `const` 防止 用户定义一个 `const complex c1(1,2)` 在使用时 `c1.real();` 会出现错误。

函数参数及返回值尽量传引用，若参数值较长，传参数会将参数值复制到栈，而引用仅会传一个地址：

```cpp
pass by value
pass by reference(to const)
retnrn by value
return by reference
```

`comple& operator += (const complex&);`

若参数值不会改变则加入const标识，引用和指针效果类似，函数中改变引用也会改变变量的值。

相同class的各个objects互为friends友元（可以使用private私有数据）
不是新的局部变量，所以可以传引用。

返回值：如返回值不是新变量(局部变量)的则返回引用 如i++

返回值：如返回值需要产生新的变量，则传变量不然函数结束临时变量的引用也就丢失了 如c = a+b 不可传c的引用

返回值：引用可方便用户实现 cout << real () << real() 等运算方式， 因为执行顺序特殊是从左到右，第一次输出后含需要接收，产生第二次输出；

## P5

操作符重载 所有操作符都是为了作用到左边

```cpp
inline complex&       //单独提出来也许其他地方也在用
__doapl(complex* ths, const const complex& r)
{
    ths->re += r.re;
    ths->im += r.rm;
    return *ths;
}

inline complex&
complex::operator += (const complex& r)
{
    return __doapl(this, r);
}
```

返回reference 是为了连续使用 如  c3+=c2+=c1; 为了应对如下:

```cpp
{
    complex c1(2,1);
    complex c2;
    
    c2 = c1 + c2;
    c2 = c1 + 5;
    c2 = 7  + c1;
}
```

所以要写非成员函数的操作符重载 因为返回的是产生了新的数据，所以只能返回value，不可以是reference

```cpp
inline complex
operator + (const compled& x, const complex& y)
{
    return complex( real(x) + real(y),
                    imag(x) + imag(y));
}

inline complex
operator + (const compled& x, double y)
{
    return complex( real(x) + y, imag(x));
}

inline complex
operator + (double x, const complex& y)
{
    return complex( x + real(y), imag(y));
}
```

complex ();  ==>  typename();  用来创建临时对象，但不给其名称 类似 int(xxx);
```cpp
{
    int(7);
    complex c1(2,1);
    complex c2;
    complex ();
    complex (4,5);
    
    cout <<complex(2);
}
```

类::函数名  是  成员函数 有this

函数名      是  全域函数 无this

对于 `<<` 符号的重载只能用全域函数

cout 不认识complex类，所以需要自己写操作符重载函数，并且需要将 << 重载运算写成全域函数，如果是类函数，默认会带this指针，使用方式会反成 a << cout 而不是 cout << a;
所有的成员函数，默认包含一个参数*this，this指针谁调用，指向谁。

``` cpp
#include <iostream.h>
ostream&
operator << (ostream& os, const complex& x)
{
    return os << '(', << real(x) << ',' << imag(x) << ')';
}
```

构造函数不带指针的类，编译器会自己增加拷贝构造和拷贝复制函数

## P6 复习

## P7

构造函数的参数带指针  需要写析构函数

big three: **拷贝构造** **拷贝复制** **析构**

```cpp
{
    String s1("hello");
    String s2(s1);    //拷贝构造
    String s2 = s1;   //拷贝赋值
}
```

```cpp
class String
{
public:
    String(const char* cstr = 0);
    String(const String& str);
    String& operator = (const String& str);
    ~String();
    char* get_c_str() const {return m_data;}
private:
    char* m_data;
};
inline String::String(const char* cstr = 0)
{
    if(cstr) {
        m_data = new char[strlen(cstr)+1];              //一定要 +1 用来存储结束符 '\0'
        strcpy(m_data, cstr);
    }
    else {
        m_data = new char[1];
        *m_data = '\0';
    }
}
inline String::~String()
{
    delete[] m_data;
}
```

使用：

```cp
{
    String s1();
    String s2("hello");
    
    String* p = new String("hello");
    delete p;
}
```

**浅拷贝**(别名)  问题：两个变量控制同一块内存  一个变量原来的数据没有指针

**深拷贝**(数据相同的两个变量)

编译器默认给出的是浅拷贝，需要自己写拷贝构造函数

经典写法三步走 **删除** **新建** **拷贝**

```cpp
inlin
String& String::operator=(const String& str)
{
    if(this == &str)   //检测自我赋值
        return *this;
    delete[] m_data;
    m_data = new char[strlen(str.m_data)+1];
    strcpy(m_data, str.m_data);
    return *this;
}
```

## P8 堆 栈 内存管理

本章只针对 new 这种动态分配的用法
```cpp
Complex c1(1,2);                   //全局对象 直到程序结束
{
    Complex c1(1,2);               //离开作用域 c1 调析构 然后自动清除
    Complex* p = new Complex(3);   //离开作用域 p所指内存 还存在  需要手动释放
    static Complex c2(1,3);        //离开作用域 c2还存在，直到程序结束
    
    delete p;                      //手动释放
}

{
    Complex* p = new COmplex(3);   //离开作用域 p所指内存还存在，但p已消失，造成内存无法回收
}
```

`new`   ：分配内存 数据类型转换 调用构造函数

`delete`：调用析构函数 释放内存

动态内存分配详解：  16的倍数 不够填0

单个对象的情况：
上下Cooke 4字节*2  内容是分配长度，最后一位代表 空闲 or 使用
调试模式会多出
固定字符 上4字节*8 下4字节
实际内容

数组的情况：
debug 和 release 会再实际内容之前加数组长度

`new []`   //array

`delete []`

单一或数组形式都会有Cooke记录分配内存的长度，但数组形式的delete如果没有搭配[] 会造成对象的析构函数仅调用一次，无法将对象构造时申请的内存删除，进而造成内存泄露。
所以若构造函数没有内存分配，使用delete不会出问题。但不可以这样用。

## P9 复习String类的实现过程

## P10 类模板 函数模板

在变量或函数前加 static 它就是静态变量或静态函数

在对象实例化的时候，产生的时不包含静态部分的数据；

静态函数没有this指针，不能传递调用来源，所以只能操作静态变量；

```cpp
class Account {
public:
    static double m_rate;
    static void set_rate(const double& x) {m_rate = x;}
};
double Account::m_rate = 8.0;  //必须如此赋初值，类中仅有对象有声明，还必须在此定义，初值可有可无；
```

调用静态函数的两种方式
```cpp
int main()
{
    Account::set_rate(5.0);  //通过类名调用 class name
    
    Account a;
    a.set_rate(7.0);         //通过对象调用 object   特殊点在于没有this
}
```

Singleton 设计模式  唯一

```cpp
class A {
public:
    static A& getInstance();
    setup() {...}
private;
    A();;
    A(const A& rhs);
    ...
};
A& A::getInstance()
{
    static A a;
    return a;
}
```

优点只有在使用个体Instance() 函数时才会创建a
调用: `A::getInstace().setup();`
    

类模板 `class template`  使用一个符号标识一种数据类型

```cpp
template<typename T>
class complex
{
public:
    complex (T r = 0, T i = 0)
        : re (t), im (i)
    {}
    complex& operator += (const complex);
    T real () const {return re;}
    T imag () const {return im;}
private:
    T re, im;
    
    friend complex& __doapl (complex*, const const complex&);
}
{
    complex<double> c1(2.5, 1.5);
    complex<int> c2(2, 6);
    ...
}
```

函数模板 `function template`
```cpp
template <class T>
inline
const T& min(const T& a, const T& b)
{
    return b < a ? b : a;
}

stone r1(2, 3), r2(3,3), r3;
r3 = min(r1, r2);
```

编译器不知道stone如何比大小，所以再增加操作符重载
```cpp
class stone
{
public:
    stone(int w, int h, int we)
        : _w(w), _h(h), _weight(we)
            {    }
    bool operator< (const stone& rhs) const
        {    return _weight < rhs._weight;  }
private:
    int _w, _h, _weight;
};
```

如何建立自己的命名空间 可以分段写
```cpp
namespace std
{
   ...
}
```

如何使用:

用法1 using directive  全部打开
```cpp
#include <iostream.h>
using namespace std;

int main()
{
    cin << ...;
    cout << ...;
    return 0;
}
```

用法2 using declaration  打开一条 如下打开cout 而cin没打开
```cpp
#include <iostream.h>
using std::cout;

int main()
{
    std::cin << ...;
    cout << ...;
    return 0;
}
```

用法3   完整书写

```cpp
#include <iostream.h>

int main()
{
    std::cin << ...;
    std::cout << ...;
    return 0;
}
```

## 面向对象

## P11 组合与继承

复合 Composition

class中含有另一个class  **(has a)**  让Sequence帮queue实现功能
设计模式 Adapter 变压器、转换器
```cpp
template <class T, class Sequence = deque<T>>
class queue {
    ...
protected:
    Sequence c;    //重点 queue中有个Sequence(底层容器)
public:
    bool empty() const {return c.empty();}
    
    void push (const value_type& x) {c.push_back(x);}
    void pop() {c.pop_front();}
};
```
生命周期(一起出现或消失，仅有先后顺序)

创建过程 先内部 后外部  (基础扎实)

外部构造函数会帮我们调用内部的构造函数，但仅会调用默认的构造函数，如果不适用需要自己写清楚
```cpp
Container::Container(...) : Component() {...};     //Component()是编译器帮我们加的 不适用就自己主动写
```

析构过程 先外部 后内部  (层层剥开)
```cpp
Container::~Container(...){... ~Component()};   //~Component() 是编译器帮我们加的
```

委托 Delegation (Composition by reference)

从来不讲 by point，因为学术界只讲 by reference 传指针也归类为 by reference  **class中含有另一个class的指针**

至于什么时候真的有另一个class不确定
生命周期 不同步，没有先后，需要谁创建谁。
可以实现类似接口的功能，对外的class不变，内部函数的变化不影响外部。 也叫编译防火墙   Handle/Body(头和身体)

继承 Inheritance

class中含有另一个class, 表示 **is-a**  子类中有父类的成分part 包含数据和函数的调用权

创建过程 先内部父类 后外部子类  (基础扎实)         子类构造函数会帮我们调用父类的构造函数，但仅会调用默认的构造函数，如果不适用需要自己写清楚
```cpp
Derived::Derived(...) : Base() {...};  //Base()是编译器帮我们加的 不适用就自己主动写
析构过程 先外部子类 后内部父类  (层层剥开)
Derived::~Derived(...){... ~Base()};   //~Base() 是编译器帮我们加的
```

如果类会成为父类，析构函数必须是virtual，否则会出现undefined behavior


## P12 虚函数 virtual functions

non-virtual **非虚函数**：不希望子类(derived class)重新定义(override, 覆写)

virtual       **虚函数**：  希望子类(derived class)重新定义(override, 覆写)，即使它已有默认定义。

pure virtual  **纯虚函数**：  希望子类(derived class)重新定义(override, 覆写)，它一定没有默认定义

子函数使用override重写定义 
```cpp  
class Shape{
public:
    virtual void draw() const = 0;                 //纯虚函数  子函数一定要重写
    virtual void error(const std::string& msg);    //虚函数
    int objectID() const;                          //非虚函数
    ...
};
class Rectangle: public Shape{...};
class Ellipse.public Shape {...};
```

设计模式：Template Method
较为有名的应用MFC

```cpp
#include <iostream>
using namespace std;

class CDocument
{
public:
    void OnFileOpen()
    {
        cout << "open file..." << endl;
        Serialize();
        cout << "close file..." << endl;
    }
    virtual void Serialize() {};
};
class CMyDoc : public CDocument
{
public:
    //只有应用程序本身知道如何读取自己的文件（格式）
    virtual void Serialize()
    {
        cout << "CMyDoc::Serial()" << endl;
    }
}
int main()
{
    CMyDoc myDoc;
    myDoc.OnFileOpen();
}
```

委托 + 继承

设计模式Observer
列表存放需要被更新的目标，提供订阅 退订 和 遍历功能
父类提供更新虚函数，由子类继承实现需要被更新的操作
多用于UI更新

委托 + 继承

设计模式Composite
文件系统 既有文件 也有目录 目录里面还有文件和目录

委托 + 继承

设计模式Prototype 原型
创建一个未来别人定义的类


## P14

C++技术主线

面向对象编程 和 泛型编程(模板)

STL标准库基本都是模板编程，泛型思维，继承关系很少
书籍推荐

## P15

转换函数 conversion function
```cpp
class Fraction   //分数
{
public:
    Fraction(int num, int den = 1);
     : m_numeratior(num), m_denominator(den) {}
    operator double() const {
      return (double)(m_numerator / m_denmoinator);
    }
private:
    int m_numerator;    //分子
    int m_denominator;  //分母
};
```

应用：
```cpp
Fraction f(3, 5);
double d = 4+f;  //调用operator double() 将f 转为0.6
```