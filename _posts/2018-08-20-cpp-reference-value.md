---
layout: post
title: c++ 值类别
date: 2018-8-20
categories: cpp
tags: reference
excerpt: 目前c++有三种基本值类别：纯右值（prvalue，pure rvalue）、亡值（xvalue，eXpiring value）、左值（lvalue， left value）。
mathjax: true
---

# 值类别

* 泛左值（glvalue，generalized lvalue）是其求值确定一个对象、位域或函数的个体的表达式；
* 纯右值（prvalue）是这样的表达式，其求值
  * 计算某个运算符的操作数的值（这种纯右值没有结果对象），或者
  * 初始化某个对象或位域（这种纯右值具有结果对象）。所有类和数组的纯右值都拥有结果对象，即使它被舍弃。
* 亡值（xvalue）是代表其资源能够被重新使用的对象或位域的泛左值；
* 左值（lvalue）是非亡值的泛左值；
* 右值（rvalue）是纯右值或者亡值。

> 左值、亡值和纯右值，它们表征了表达式的属性，而这种属性的区别主要体现在使用上，如能否做运算符的左操作数、能否使用移动语义。左值可以看作“对象”，右值可以看作“值”。但是不能简单的通过“=”来区分左值和右值。

## 左值

有名字的变量一定是左值。如：int a = 0; a是左值。
进一步来讲，能够用&取地址的是左值表达式。左值也可以理解为Location Value，表示在内存中可以寻址，赋值
**字符串字面值是左值，如“abc”是左值，因为它可以取地址，但是不能被赋值。**
常见左值表达式：

```c++
a = b, a += b；
++i, --i; //是左值
*p;

字符串字面值，例如 "Hello, world!"；

Foo &retFoo(); //返回一个引用，retFoo调用是一个左值
Foo f1;
retFoo() = f; //正确， retFoo()返回一个左值
```

## 纯右值

纯粹的字面值。如：1是纯右值
求值结果相当于字面值或是一个不具名的临时对象。纯右值也可以理解为Read Value。可以读出值。
常见纯右值表达式：

```c++
i++, i--; //是纯右值

a + b, a % b，和其他内建的算术表达式

字面值（除字符串字面值）。1, true, nullptr;
&a;
a.m; //对象成员表达式

Lambda表达式，[](int x){ return x * x; }
Foo retVal(); //返回一个值，retval调用时一个右值
```

## 亡值

在C++11之前的右值和C++11中的纯右值是等价的。C++11中的将亡值是随着右值引用的引入而新引入的。

* 返回右值引用的函数的调用表达式
* 转换为右值引用的转换函数的调用表达式。比如返回右值引用T&&的函数返回值、std::move的返回值，或者转换为T&&的类型转换函数的返回值。

常见亡值表达式：

```c++
函数调用或重载运算符表达式，其返回类型为右值引用，例如 std::move(x);
```

## 聚合类

**聚合类**使得用户可以直接访问其成员，并且具有特殊的初始化语法形式。当一个类满足如下条件时，我们就说它是聚合的：

* 所有成员都是public的
* 没有定义任何构造函数
* 没有类内初始值
* 没有基类，也米有virtual函数

例如常见的聚合类：

```c++
struct Point
{
  int x;
  int y;
};

//我们使用成员初始化列表，并用它初始化聚合类的数据成员
Point p = {1, 2};
```

## 字面值常量类

除了算数类型、引用和指针外，某些类也是字面值类型。和其他类不同，字面值类型的类可能含有`constexpr`。
数据成员都是字面值类型的聚合类是字面值常量类。
如果一个类不是聚合类，但他符合下述要求，则它也是一个字面值常量类：

* 数据成员都必须是字面值类型
* 类必须至少含有一个constexpr构造函数
* 如果一个数据成员含有类内初始值，则内置类型成员的初始值必须是一条常量表达式；或者如果成员属于某种类类型，则初始值必须使用成员自己的constexpr构造函数
* 类必须使用析构函数的默认定义，改成员负责销毁类的对象

### constexpr构造函数

尽管构造函数不能是const的，但是字面值常量类的构造函数可以是constexpr函数。事实上，一个字面值常量类必须至少提供一个constexpr构造函数。
constexpr构造函数可以声明成=default的形式。否则，constexpr构造函数就必须既符合构造函数的要求（意味着不能包含返回语句），又符合constexpr函数的要求（意味着它能拥有的唯一可执行语句就是返回值语句）。综合这两点可知，constexpr构造函数体一般来说应该是空的。我们通过前置关键字constexpr就可以声明一个constexpr构造函数了：

```c++
class ConstObject
{
public:
  constexpr ConstObject(int v) : m_val(v){}

  constexpr int Value() const { return m_val; }

private:v
  int m_val;
};

template<int n>
struct constN
{
  constN() { std::cout << n << '\n'; }
};

constexpr ConstObject co(1);
constN<co.Value()> out;
```

## 类的静态成员

我们通过在成员的声明之前加上关键字`static`使得其与类关联在一起。

```c++
class StaticObject
{
public:
  StaticObject() {}
  ~StaticObject() {}

  static int Value() { return m_val; }

private:
  static int m_val;
};

//定义并初始化
int StaticObject::m_val = 1;

int v = StaticObject::Value();
```

### 静态成员的类内初始化

通常情况下，类的静态成员不应该在类的内部初始化。然而，我们可以为静态成员提供const整数类型的类内初始化，不过要求静态成员必须字面值常量类型的`constexpr`。

```c++
class StaticObject
{
public:
  StaticObject() {}
  ~StaticObject() {}

  static int Value() { return m_val; }

private:
  static constexpr int m_val = 1;
};

//定义并初始化
constexpr int StaticObject::m_val;

int v = StaticObject::Value();
```

> 即使一个常量静态数据成员在类内部被初始化了，通常情况下也应该在类的外部定义一下该成员

### 静态成员用于某些特殊场景

静态数据成员可以是不完全的类型。特别的静态数据成员可以就是它所属的类类型，而非静态数据成员则受到限制，只能声明成它所属类的指针和引用：

```c++
class Bar
{
public:
  //...

private:
  static Bar men1;
  Bar* men2;
  Bar men3; //错误，数据成员必须是完全类型
}
```

我们可以使用静态成员作为默认实参：

```c++
class Screen
{
public:
  //bkground表示一个在类中稍后定义的静态成员
  Screen& Clear(char = bkground);

private:
  static const char bkground;
}
```