---
layout: post
title: enum class 和 plain enum
comments: false
date: 2016-02-02 20:29:07
tags:
  - The C++ Programming Language 4th
categories:
  - C/C++
---

_C++ 11_ 为 _enumeration_ （枚举）类型引入了几个新特性

* _underlying type_
* _enum class_

<!--more-->

## _underlying type_

默认情况下枚举的底层类型是 _int_ 。在 _C++11_ 中可以指定为 _signed_ 或 _unsigned integer_ —— _char_ 也在此列

{% codeblock lang:cpp change underlying type %}
enum COLOR:char
{
  red,
  green,
  black,
  white
};
{% endcodeblock %}

上面的代码把枚举的底层类型换成了 _char_ ，这样 _COLOR_ 对象的大小变成了 _1 byte_ ——可以节省内存

## _enum class_

_C++ 11_ 把枚举分成了两类

* _enum class_
* _plain enum_

_enum class_ 又称作强类型枚举，区别于传统枚举（_plain enum_）的特点是

* 枚举名字（_enumerator name_）的作用域限制在类型内部
* 值不能隐式转换

### 作用域

传统枚举类型的枚举名字暴露在类型所在的作用域，所以很容易出现名字冲突

{% codeblock lang:cpp plan enum 的名字冲突 %}
enum COLOR1
{
  red,
  green,
  black,
  white
};
enum COLOR2
{
  red,
  blue,
  pink
};
{% endcodeblock %}

这样的代码是无法通过编译的（虽然没什么用），因为 _red_ 出现了两次。以往碰到这种情况，通常会把枚举类型放到不同的 _namespace_ 中

使用 _enum class_ 可以直接避免这种情况

{% codeblock lang:cpp 使用 enum class %}
enum class COLOR1
{
  red,
  green,
  black,
  white
};
enum class COLOR2
{
  red,
  blue,
  pink
};

int main (int argc, char **argv)
{
  COLOR1 cl1 = COLOR1::red;
  COLOR2 cl2 = COLOR2::red;
  return 0;
}
{% endcodeblock %}

两个 _red_ 被限制在各自的枚举类型内部，不会引起冲突——所以必须加上类型前缀使用

### 隐式转换

{% codeblock lang:cpp plain enum 的常用范式 %}
enum COLOR
{
  red   = 1,
  green = 2,
  black = 4
};

int main (int argc, char **argv)
{
  COLOR cl = COLOR::red;
  if (cl & red)
  {
  }
  else if (cl & green)
  {
  }
  else if (cl & black)
  {
  }
  return 0;
}
{% endcodeblock %}

这样的用法对 _enum class_ 是行不通的——位操作 _&_ 在不重载操作符的情况下不能用于自定义类型

{% codeblock lang:cpp enum class 版本 %}
enum class COLOR
{

  none  = 0,
  red   = 1,
  green = 2,
  black = 4
};

COLOR operator&(COLOR a, COLOR b)
{
  return static_cast<COLOR> (static_cast<int> (a) & static_cast<int> (b));
}

int main (int argc, char **argv)
{
  COLOR cl = COLOR::red;
  if ((cl & COLOR::red) != COLOR::none)
  {
  }
  return 0;
}
{% endcodeblock %}

代码中重载的 _&_ 操作符返回 _COLOR_ 对象，所以要注意第 _18_ 行的代码不能写成传统形式—— _COLOR_ 类型同样不能隐式转换为 _bool_ 类型