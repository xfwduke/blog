---
layout: post
title: trivial type
comments: false
date: 2016-02-03 20:38:32
tags:
  - The C++ Programming Language 4th
categories:
  - C/C++
---

_trivial type_ 表示结构相对简单的数据类型

{% blockquote @C++ Reference http://www.cplusplus.com/reference/type_traits/is_trivial/ %}
A trivial type is a type whose storage is contiguous (trivially copyable) and which only supports static default initialization (trivially default constructible), either cv-qualified or not. It includes scalar types, trivial classes and arrays of any such types.
{% endblockquote %}

<br>

这段话很精简的描述了什么是 _trivial type_ ，但要完全看懂却很不容易

<!--more-->

## _cv-qualified_

_cv-qualified_ 即是用 _const / volatile_ 修饰的类型[^1]

[^1]:[http://stackoverflow.com/questions/15413037/what-does-cv-unqualified-mean-in-c](http://stackoverflow.com/questions/15413037/what-does-cv-unqualified-mean-in-c)

* _const-qualified——with the `const` cv-qualifier_
* _volatile-qualified——with the `volatile` cv-qualifier_
* _const-volatile-qualified——with both the `const` and `volatile` cv-qualifiers_

## _scalar types_

{% blockquote @C++ Reference http://www.cplusplus.com/reference/type_traits/is_scalar/ %}
A scalar type is a type that has built-in functionality for the addition operator without overloads (arithmetic, pointer, member pointer, enum and std::nullptr_t).
{% endblockquote %}

<br>

有内建加法操作的类型都属于 _scalar type_ ，包括`算术类型`、`指针`、`枚举`和`nullptr`

_type_trait::is_scalar_ 可以用来判断类型是不是 _scalar type_

{% codeblock lang:cpp is_scalar 原型 %}
template<class T>
struct is_scalar : std::integral_constant < bool,
         is_arithmetic<T>::value || is_pointer<T>::value ||
         is_member_pointer<T>::value || is_enum<T>::value ||
         is_same<nullptr_t, typename remove_cv<T>::type>::value> {};
{% endcodeblock %}

## _contiguous ( trivial copyable )_

{% blockquote @C++ Reference http://www.cplusplus.com/reference/type_traits/is_trivially_copyable/ %}
A trivially copyable type is a type whose storage is contiguous (and thus its copy implies a trivial memory block copy, as if performed with memcpy), either cv-qualified or not. …… for scalar types, trivially copyable classes and arrays of any such types.

A trivially copyable class is a class (defined with class, struct or union) that:

* uses the implicitly defined copy and move constructors, copy and move assignments, and destructor.
* has no virtual members.
* its base class and non-static data members (if any) are themselves also trivially copyable types.
{% endblockquote %}

<br>

从定义来看， _trivially copyable type_ 在存储上是连续的，可以用 _memcpy_ 复制对象；当一个类是 _trivially copyable class_ 时，就肯定是 _trivially copyable type_

### 存在指针成员
{% codeblock lang:cpp 反例 %}
class Test
{
 public:
  Test () = default;

  Test (int len, int start) : _len (len), _ptr (new int (_len))
  {
    for (int i = 0; i < _len; ++i)
    {
      _ptr[i] = start++;
    }
  }

  void disp ()
  {
    for (int i = 0; i < _len; ++i)
    {
      cout << _ptr[i] << " ";
    }
  }

  ~Test ()
  {
    delete[] _ptr;
  }

 private:
  int _len;
  int * _ptr;
};

int main (int argc, char ** argv)
{
  Test ins1(10, 4);
  Test ins2;
  memcpy (&ins2, &ins1, sizeof (Test));
  ins2.disp ();
  return 0;
}
{% endcodeblock %}

_Test_ 类违背了 _trivially copyable class_ 定义的第一条。执行的结果会遇到 _runtime error_ —— _memcpy_ 无法正确深复制指针成员，导致 __ptr_ 指向的内存被释放两次

### 存在虚成员

有虚成员的类用 _memcpy_ 复制对象结果是未定义的，在某些编译器上结果可能是正确的，但不保证会一直正确下去——这是目前能查到的东西，试了半天也没找到什么反例

## _trivial class_

上面 _trivially copyable_ 展示了 _trivially_ 到底是个什么东西，说起来也很简单

* 只能存在默认复制构造函数，不允许自定义
* 不存在虚成员
* 基类和非静态成员也满足上面两条

而 _trivial class_ 就是满足一系列 _trivially XXX_ 的类，包括

* _trivially copyable_
* _trivially constructible_
* _trivially move_
* _trivially assignment_
* _trivially destructor_

同时还必须

* 没有虚成员
* 非静态成员不能在类定义中初始化—— _C++ 11_ 的新特性，实际就相当于用户自定义构造函数了
* 基类和非静态成员也满足前面的条件

看起来定义很长，但看过每一个 _trivially XXX_ 定义后会发现， _trivially class_ 实际是不能存在任何用户自定义的资源管理函数——构造、复制、移动和析构