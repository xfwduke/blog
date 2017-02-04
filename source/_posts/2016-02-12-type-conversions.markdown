---
layout: post
title: type conversions
comments: false
date: 2016-02-12 13:52:14
tags:
  - C++ Primer
  - The C++ Programming Language 4th
categories:
  - C/C++
---

_C/C++_ 的类型转换分为隐式转换（_implicit conversion_）和显示转换（_explicit conversion_）

_C_ 的类型转换相对简单：显示转换只有一种形式；而隐式转换大多针对内置的算术类型，副作用多是精度损失

_C++_ 引入了类和继承，类型转换变得更加复杂

<!--more-->

## 隐式转换 ##

在 *C* 时代隐式转换只能针对内置类型对象或者指针，*struct* 对象是没办法做隐式转换的。*C++* 则提供了在完全不相关的类对象指间做隐式转换的办法

### *standard conversion* ###

基本类型（*short、int、double* 等算术类型，布尔类型，枚举及 *char*）间的隐式转换被称为 *standard conversion* [^1]

[^1]: [http://www.cplusplus.com/doc/tutorial/typecasting/](http://www.cplusplus.com/doc/tutorial/typecasting/)

转换规则也比较简单

* 负数转换为无符号数会得到补码（*2's complement*）
* 浮点类型转换为整型会截断小数部分
* 非零数值转换为布尔类型为 *true* ，*0* 为 *false*

### *non-fundamental types* ###

非基础类型（*non-fundamental types*）包括数组、指针和函数——说白了就是各种指针，转换规则是[^1]

* 空指针可以转换给任何类型的指针
* 任何类型的指针都可以转换为 *void* 指针
* 派生类指针可以转换为父类指针

### *user-define class* ###

为了实现类对象的隐式转换，类至少要包含下列 *3* 个部件的任何一个

* 只有一个参数的构造函数——用源类型构造目标类型
* 重载了赋值操作符
* 实现了类型转换操作符

{% codeblock lang:cpp 自定义类的隐式转换 %}
#include <iostream>

using namespace std;
class B;

class A
{
 public:
  A () = default;

  A & operator = (const B & b)
  {
    cout << "In copy assignment" << endl;
    return *this;
  }

  A (const B & b)
  {
    cout << "in constructor" << endl;
  }
};

class B
{
 public:
  operator A ()
  {
    cout << "in type-caster" << endl;
    return A ();
  }
};

int main (int argc, char ** argv)
{
  B insb;

  A insa0 (insb);
  A insa1 = insb;
  insa1 = insb;
  return 0;
}
{% endcodeblock %}

这段代码展示了如何实现从 *class B* 到 *class A* 的隐式转换，执行结果是

{% codeblock 结果 %}
in constructor
in type-caster
In copy assignment
{% endcodeblock %}

需要注意的是，代码的第 *42，43* 行从形式上看都是对象构造，却分别调用了构造函数和类型转换操作符，这和 *C++* 的重载版本选择策略有关，详细的可以看这篇文章[^3]，只要稍微修改下代码就可以产生二义性错误

[^3]: [http://stackoverflow.com/questions/1384007/conversion-constructor-vs-conversion-operator-precedence](http://stackoverflow.com/questions/1384007/conversion-constructor-vs-conversion-operator-precedence)

## 显示转换 ##

从实际情况来看隐式转换不能带来什么好处，反而总是会引发各种难以定位的 *bug*——特别是在使用自定义类的场景。所以大多数情况下，在可能引起隐式转换的地方（单参数构造函数）都会用 *explicit* 关键字禁止隐式转换[^4]

[^4]: [http://en.cppreference.com/w/cpp/language/explicit](http://en.cppreference.com/w/cpp/language/explicit)

*C++* 提供了 *4* 种显示转换的途径来让用户控制类型转换

* *dynamic_cast*
* *static_cast*
* *reinterpret_cast*
* *const_cast*

另外还要加上从 *C* 继承来的 *C-style cast*


### 什么时候需要转换 ###

*C/C++* 并不太赞成类对象的类型转换——虽然语言提供了转换的机制

* 没有继承关系的类之间的转换没有意义——即使是在用一个类对象构造另一个时，也建议禁止隐式转换
* 有继承关系的类如果使用对象做类型转换，会丢失多态特性
* 不添加额外的代码——前面提到的 *3* 个部件——甚至没办法实现无关类之间的转换
* 基础类型间的转换可能导致精度丢失

所以在显示转换部分，主要关注的是指针和引用

### *dynamic_cast* ###

*dynamic_cast* 只能用于类指针（ *void\** 也可以 ）和引用

{% blockquote @dynamic_cast http://www.cplusplus.com/doc/tutorial/typecasting/#dynamic_cast %}
Its purpose is to ensure that the result of the type conversion points to a valid complete object of the destination pointer type.
{% endblockquote %}

<br>

简单来说 *dynamic_cast* 要求源指针或引用指向的对象要能构造出一个完整的目标类型的对象。如果可以，则返回目标类型的指针或引用；否则以某种形式返回错误——对于指针和引用，错误的返回形式是不同的

判断 *dynamic_cast* 结果是否非常简单——向指针或引用指向的实际对象类型的基类转换是正确的，否则返回错误

{% codeblock lang:cpp dynamic_cast 演示 %}
#include <iostream>
#include <exception>

using namespace std;

class Base
{
 public:
  virtual void where ()
  {
    cout << "In Base" << endl;
  }
};

class Derived : public Base
{
 public:
  void where ()
  {
    cout << "In Derived" << endl;
  }
};

int main (int argc, char ** argv)
{

  Base * pbb = new Base;
  Base * pbd = new Derived;
  Derived * pd;

  Base ins_b;
  Derived ins_d;
  try
  {
    pd = dynamic_cast<Derived *>(pbb);
    if (pd == nullptr)
    {
      cout << "cast point error" << endl;
    }
    else
    {
      pd->where ();
    }
  }
  catch (const exception & e)
  {
    cout << e.what ();
  }
  try
  {
    pd = dynamic_cast<Derived *>(pbd);
    if (pd == nullptr)
    {
      cout << "cast point error" << endl;
    }
    else
    {
      pd->where ();
    }
  }
  catch (const exception & e)
  {
    cout << "Exception: " << e.what ();
  }

  try
  {
    Derived & rd = dynamic_cast<Derived &>(ins_b);
    rd.where ();
  }
  catch (const exception & e)
  {
    cout << "Exception: " << e.what () << endl;
  }

  try
  {
    Derived & rd = dynamic_cast<Derived &>(ins_d);
    rd.where ();
  }
  catch (const exception & e)
  {
    cout << "Exception: " << e.what () << endl;
  }

  try
  {
    Derived && rrd = dynamic_cast<Derived &&>(ins_b);
    rrd.where ();
  }
  catch (const exception & e)
  {
    cout << "Exception: " << e.what () << endl;
  }
  try
  {
    Derived && rrd = dynamic_cast<Derived &&>(ins_d);
    rrd.where ();
  }
  catch (const exception & e)
  {
    cout << "Exception: " << e.what () << endl;
  }

  return 0;
}
{% endcodeblock %}

上面这段代码

* 定义了 *2* 个 *Base* 类型的指针 *pbb* 和 *pbd* ，分别指向 *Base* 实例和 *Derived* 实例；尝试将这两个指针转换为 *Derived* 指针
* 定义了 *Base* 实例 *ins_b* 和 *Derived* 实例 *ins_d*；尝试将这两个实例转换为 *Base* 类型的引用 

{% codeblock 结果 %}
cast point error
In Derived
Exception: std::bad_cast
In Derived
Exception: std::bad_cast
In Derived
{% endcodeblock %}

从结果可以看到

* 操作指针时，返回 *nullptr* 表示错误
* 操作引用时，抛出 *std::bad_cast* 异常表示错误

### *static_cast* ###

{% blockquote @static_cast http://www.cplusplus.com/doc/tutorial/typecasting/#static_cast %}
static_cast can perform conversions between pointers to related classes, not only upcasts (from pointer-to-derived to 
pointer-to-base), but also downcasts (from pointer-to-base to pointer-to-derived). 

No checks are performed during runtime to guarantee that the object being converted is in fact a full object of the destination type.
{% endblockquote %}

<br>

有几个关键信息

* 只能用于相关类的指针——相关类是指有继承关系的类
* 可以随意的向基类或派生类做转换
* 结果是否正确由用户保证， *static_cast* 不做运行时检查——所以称作 *static* ，*dynamic_cast* 会做运行时检查

*static_cast* 的特性还有

* 可用于基本类型对象间的任意转换
* *void \** 可以和任意类型的指针相互转换
* 当不相关的类有 *conversion constructor* 或 *conversion operator* 时，可以用 *static_cast* 做对象的类型转换
* 可以用来将 *lvalue* 转换为 *rvalue reference*

{% codeblock lang:cpp static_cast 示例 %}
class Other
{
};

class Base
{
};

class Derived : public Base
{
 public:
  operator Other ()
  {
    return Other ();
  }
};

int main (int argc, char ** argv)
{
  // conversion with void *
  Other * po = new Other;
  void * pv = static_cast<void *>(po);
  po = static_cast<Other *>(pv);

  // may be get runtime-error
  Base * pb = new Base;
  Derived * pd = static_cast<Derived *>(pb);

  // call conversion operator in Derived
  Derived ins_d;
  Other ins_o = static_cast<Other>(ins_d);

  // compile error
//  Derived * ppd = new Derived;
//  Other * ppo = static_cast<Other *>(ppd);

  // get a rvalue reference
  Other && rro = static_cast<Other &&>(ins_o);

  // fundamental type conversion
  int vi = static_cast<int>(1.2);
  return 0;
}
{% endcodeblock %}

### *reinterpret_cast* ###

{% blockquote Stanley B.Lippman; Josée Lajoie; Barbara E. Mo, C++ Primer 4.11.3 %}
*reinterpret_cast* 通常为运算对象的位模式提供较低层次上的重新解释
{% endblockquote %}

<br>

这句话真的不太能说明白 *reinterpret_cast* 是干什么用的。说的更通俗些其实是

{% blockquote @reinterpret_cast http://www.cplusplus.com/doc/tutorial/typecasting/#reinterpret_cast %}
reinterpret_cast converts any pointer type to any other pointer type, even of unrelated classes.
{% endblockquote %}

{% codeblock lang:cpp %}
class Other
{
};

class Base
{
};

int main (int argc, char ** argv)
{
  Base * pb = new Base;
  Other * po = reinterpret_cast<Other *>(pb);
  return 0;
}
{% endcodeblock %}

用 *reinterpret_cast* 转换完全不相关的类指针—— *dynamic_cast* 会获得 *nullptr* ，而 *static_cast* 无法编译

### *const_cast* ###

用于修改指针或引用指向的对象的常量属性。用途说明非常短，但并不简单

* 只能用于引用或引用
* 用于指针时，只影响指针的 *low-level const*
* 可以去掉也可以加上常量属性


{% codeblock lang:cpp point const level %}
int i {10};
const int * p1 = &i;  // low-level const
int const * p2 = &i;  // low-level const
int * const p3 = &i;  // top-level const
{% endcodeblock %}

*const_cast* 最大的用途是用于函数重载

{% codeblock lang:cpp  const_cast 用于函数重载 %}
void disp (string & s)
{
  cout << s << endl;
}

void disp (const string & s)
{
  disp (const_cast<string &>(s));
}
{% endcodeblock %}

代码中的例子并不是一个合理的设计——只实现 *const* 版本更合理，但至少可以展示下用法——如果非 *const* 版的函数逻辑非常复杂，使用 *const_cast* 就可以很容易的实现它的 *const* 版本

需要强调的是 **如果指针指向的对象本身就是常量，使用** *const_cast* **可以让编译器不再阻止通过指针修改对象的值，但结果是未定义的。** 更详细的可以参考 {% post_link const-cast-的幽灵 %}
