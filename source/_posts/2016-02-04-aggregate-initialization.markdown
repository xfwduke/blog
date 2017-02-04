---
layout: post
title: aggregate initialization
comments: false
date: 2016-02-04 17:06:38
tags:
  - The C++ Programming Language 4th
categories:
  - C/C++
---

_aggregate initialization_ 是 _list initialization_ 的一种形式，而 _list initialization_ 是 _C++ 11_ 中新增的的初始化对象的方法

{% codeblock lang:cpp list initialization %}
int a {10};
vector<string> svec {"hello", "world"};
{% endcodeblock %}

<!--more-->

## _list initialization_

首先来简单看下 _list initialization_

{% codeblock lang:cpp 示例 %}
class Test
{
 private:
  int val1;
  double val2;

 public:
  Test (int a, double b) : val1 (a), val2 (b)
  {}
};

int main (int argc, char ** argv)
{
  Test ins1 (1, 1.2);
  Test ins2 {3, 7.8};   // list initialization
  Test * ptr1 = new Test (7, 5.1);
  Test * ptr2 = new Test {3, 9.9};  // list initialization

  Test narrow1 (2.2, 1.0);  // narrow 2.2 to 2 is ok
  Test narrow2 {5.2, 7};    // can't narrow 5.2 to 5, compile error
  return 0;
}
{% endcodeblock %}

代码的第 _15, 17 20_ 行就是 _list initialization_ ，最大的好处就是不允许隐式转换导致精度丢失（ _narrow_ ）

## _aggregate initialization_

_aggregate initialization_ 的形式也是用 _{}_ 来初始化对象，但对于能适用的类有明确的要求，同时也提供了附加的特性

### _aggregate type_

只有 _aggregate type_ 才能使用 _aggregate initialization_ ，具体是[^1]

[^1]:[http://en.cppreference.com/w/cpp/language/aggregate_initialization](http://en.cppreference.com/w/cpp/language/aggregate_initialization)

* 数组——任何类型的数组
* 满足下列要求的类（ _class, struct, union_ ）
  * 没有非静态（ _non-static_ ）的 _private_ 和 _protect_ 数据成员
  * 没有用户定义的构造函数（ _defaulted_ 和 _deleted_ 是允许的）
  * 没有基类
  * 没有数据成员默认初始化

### 特性

特性有一大版，结合例子看还是比较容易理解的[^1]

* Each array element or non-static class member, in order of array subscript/appearance in the class definition, is copy-initialized from the corresponding clause of the initializer list.
* If the initializer clause is an expression, implicit conversions are allowed as per copy-initialization, except if they are narrowing (as in list-initialization) (since C++11).
* If the initializer clause is a nested braced-init-list (which is not an expression), the corresponding class member is list-initialized from that clause: aggregate initialization is recursive.
* If the object is an array of unknown size, and the supplied brace-enclosed initializer list has n clauses, the size of the array is n
* Static data members and anonymous bit-fields are skipped during aggregate initialization.
* If the number of initializer clauses exceeds the number of members to initialize, the program is ill-formed (compiler error)
* If the number of initializer clauses is less than the number of members or initializer list is completely empty, the remaining members are initialized by their default initializers, if provided in the class definition, and otherwise (since C++14) by empty lists, which performs value-initialization. If a member of a reference type is one of these remaining members, the program is ill-formed (references cannot be value-initialized)
* If the aggregate initialization uses the form with the equal sign (T a = {args..}), (until C++14) the braces around the nested initializer lists may be elided (omitted), in which case as many initializer clauses as necessary are used to initialize every member or element of the corresponding subaggregate, and the subsequent initializer clauses are used to initialize the following members of the object. However, if the object has a sub-aggregate without any members (an empty struct, or a struct holding only static members), brace elision is not allowed, and an empty nested list {} must be used.
* When a union is initialized by aggregate initialization, only its first non-static data member is initialized.



{% codeblock lang:cpp 例子 %}
{% raw %}
#include <string>
#include <array>
struct S {
    int x;
    struct Foo {
        int i;
        int j;
        int a[3];
    } b;
};
 
union U {
    int a;
    const char* b;
};
int main()
{
    S s1 = { 1, { 2, 3, {4, 5, 6} } };
    S s2 = { 1, 2, 3, 4, 5, 6}; // same, but with brace elision
    S s3{1, {2, 3, {4, 5, 6} } }; // same, using direct-list-initialization syntax
    S s4{1, 2, 3, 4, 5, 6}; // error in C++11: brace-elision only allowed with equals sign
                            // okay in C++14
 
    int ar[] = {1,2,3}; // ar is int[3]
//  char cr[3] = {'a', 'b', 'c', 'd'}; // too many initializer clauses
    char cr[3] = {'a'}; // array initialized as {'a', '\0', '\0'}
 
    int ar2d1[2][2] = {{1, 2}, {3, 4}}; // fully-braced 2D array: {1, 2}
                                        //                        {3, 4}
    int ar2d2[2][2] = {1, 2, 3, 4}; // brace elision: {1, 2}
                                    //                {3, 4}
    int ar2d3[2][2] = {{1}, {2}};   // only first column: {1, 0}
                                    //                    {2, 0}
    std::array<int, 3> std_ar2{ {1,2,3} };    // std::array is an aggregate
    std::array<int, 3> std_ar1 = {1, 2, 3}; // brace-elision okay
 
    int ai[] = { 1, 2.0 }; // narrowing conversion from double to int:
                           // error in C++11, okay in C++03
 
    std::string ars[] = {std::string("one"), // copy-initialization
                         "two",              // conversion, then copy-initialization
                         {'t', 'h', 'r', 'e', 'e'} }; // list-initialization
 
    U u1 = {1}; // OK, first member of the union
//    U u2 = { 0, "asdf" }; // error: too many initializers for union
//    U u3 = { "asdf" }; // error: invalid conversion to int
 
}
{% endraw %}
{% endcodeblock %}