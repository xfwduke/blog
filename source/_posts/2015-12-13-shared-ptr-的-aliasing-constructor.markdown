---
title: shared_ptr 的 aliasing constructor
comments: false
date: 2015-12-13 15:12:36
tags:
  - C++ Primer
  - C++ 标准库第二版
categories:
  - C/C++
---

共享指针的 _aliasing constructor_ 是其构造函数中比较特殊的一个, 因为它的参数和其他的构造函数都不同

{% codeblock aliasing constructor lang:cpp %}
template <class U> shared_ptr (const shared_ptr<U>& x, element_type* p) noexpect;
{% endcodeblock %}

这个形式的构造函数适用于

{% blockquote Nicolai M.Josuttis 侯捷,C++标准库第二版-5.2.4 %}
某对象拥有另一个对象
{% endblockquote %}

<!--more-->

{% codeblock 具有包含关系的类 lang:cpp %}
struct Inner
{
    int _v;

    Inner (int v) : _v (v)
    { }

    ~Inner ()
    {
      cout << "Destory inner" << endl;
    }
};

struct Outter
{
    Inner _in;

    Outter (int v) : _in (v)
    { }

    ~Outter ()
    {
      cout << "Destroy outter" << endl;
    }
};
{% endcodeblock %}

_Outter_ 和 _Inner_ 类存在包含关系——一个 _Outter_ 类型的对象包含一个 _Inner_ 类型的子对象 ( _subobject_ )

当[^1]

* 通过动态分配( _dynamically allocted_ ) 构造了一个 _Outter_ 类型的共享指针
* 希望把子对象 ( _Inner_ 类型) 以共享指针的形式用于其他地方

需要保证子对象使用结束前, 其外围对象能一直有效—— _aliasing constructor_ 就是为了达到这个目的

## 错误的示例

先来看看不使用 _aliasing constructor_ 会怎样

{% codeblock 类定义沿用前面的 lang:cpp %}
#include <iostream>
#include <memory>

using namespace std;

struct null_deleter
{
    template<typename T>
    void operator () (T *)
    { }
};

shared_ptr<Inner> fun ()
{
  shared_ptr<Outter> spo (new Outter (5));
  shared_ptr<Inner> spi (&spo->_in, null_deleter ());
  return spi;
}

int main (int argc, char **argv)
{
  auto spi = fun ();
  cout << "returned from fun" << endl;
  cout << ++(spi->_v) << endl;
  return 0;
}
{% endcodeblock %}

{% codeblock 执行结果 %}
Destroy outter
Destory inner
returned from fun
1
{% endcodeblock %}

从结果可以看到

* 离开函数 _fun_ 时, _Outter_ 对象被析构, 同时导致 _Inner_ 对象也被析构
* 最后的整型计算结果输出 _1_ 而不是预期的 _5_——虽然最后计算没有报错, 但保不齐以后执行会不会出什么错误——因为修改了一个被回收了的内存地址

## 正确的示例

将上面代码的第 _16_ 行替换成下面这样

{% codeblock 替换代码 lang:cpp %}
shared_ptr<Inner> spi (spo, &spo->_in);
{% endcodeblock %}

则执行结果符合预期, 并且对象都是在使用结束后 ( _main_ 函数返回 ) 时才析构

{% codeblock %}
returned from fun
6
Destroy outter
Destory inner
{% endcodeblock %}

所以 _aliasing constructor_ 参数的含义是

* 第 _2_ 个参数是要共享的子对象指针
* 第 _1_ 个参数是拥有这个子对象的上级对象的共享指针

## 不明白的地方

有两个不明白的地方

* [错误的示例](#错误的示例) 中的代码, 编译器没有报错, 而且也还没想到用什么办法来让程序崩溃掉
* 将代码的第 _15_ 行替换成

{% codeblock lang:cpp %}
shared_ptr<Outter> spo (make_shared<Outter> (5));
{% endcodeblock %}

即使不使用 _aliasing constructor_ , 对象在从 _fun_ 返回时被析构, 也可以让整型计算的结果是正确的



[^1]:[std::shared_ptr's secret constructor](https://www.justsoftwaresolutions.co.uk/cplusplus/shared-ptr-secret-constructor.html)