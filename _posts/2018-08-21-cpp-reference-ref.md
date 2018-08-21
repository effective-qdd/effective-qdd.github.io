---
layout: post
title: c++ 引用
date: 2018-8-21
categories: cpp
tags: reference
excerpt: C++11 中新增了右值引用（rvalue reference）。为了与右值引用区分开来，我们把常规引用称为左值引用（lvalue reference）。
mathjax: true
---

# 左值引用

左值引用是具名变量值的别名。左值引用通常不能绑定到右值。
常量左值引用可以接受非常量左值、常量左值、右值对其进行初始化。

```c++
T & ref = object ;

int n = 2; // 非常量左值
int &r1 = n;  // 到对象 n 的左值引用
const int &cr(n); 
volatile int &cv{n};
int &r2 = r1; // 另一到对象 n 的左值引用

int& r3 = const_cast<int&>(cr); // 需要 const_cast

//object 是左值表达式，且其类型为 T 或从 T 派生，而且有相等或更少的 cv 限定，则引用被绑定到左值表达式所标识的对象或其基类子对象。
struct A {};
struct B : A {} b;
A& ra = b; // ra 引用 b 中的 A 子对象

//若 object 的类型与 T 不相同且不从它派生，而 object 拥有到左值的转换函数，其类型为 T 或从 T 派生，有相等或更少的 cv 限定，
//则绑定引用到转换函数所返回的左值所标识的对象（或到其基类子对象）。
struct A { };
struct B : A { operator int&(); };
int& ir = B(); // ir 引用 B::operator int& 的结果

void (&rf)(int) = foo; // 到函数的左值引用
int ar[3];
int (&ra)[3] = ar; // 到数组的左值引用

X &foo(X &&x) //具名的右值引用是左值，虽然声明的是右值引用
{
  X anotherX = x; //调用的是X的拷贝构造函数
  //...
  return x; //如果x是一个右值则不能赋值给non-const左值引用的。
}
```

# 右值引用

右值引用则是不具名（匿名）变量的别名。右值引用只能绑定到一个将要销毁的对象。 
右值引用通常不能绑定到任何的左值，要想绑定一个左值到右值引用，通常需要std::move()将左值强制转换为右值。

```c++
T && ref = object ;

int &&i= 0;
int a = 1;
int &&b = std::move(a); //std::move, 强制把左值变成右值。
a = 2; //此时，b = 2；
const int &cref = 1; // 绑定到右值

//object 是非位域右值或函数左值，且其类型是 T 或从 T 派生，拥有相等或更少 cv 限定，则绑定引用到初始化器表达式的值或其基类子对象
struct A { };
struct B : A { };
extern B f();
const A& rca2 = f();
A&& rra = f(); //到 B 右值的 A 子对象。

int i2 = 42;
int&& rri = static_cast<int&&>(i2); // 直接绑定到 i2

//object 的类型不同于 T 且不从它派生，而 object 拥有到右值或函数左值的转换函数，其类型为 T 或从 T 派生，且拥有有相等或更少的 cv 限定，
//则绑定引用到转换函数的结果或到其基类子对象
struct A { };
struct B : A { };
struct X { operator B(); } x;
const A& r = x;
B&& rrb = x;    // 直接绑定到转换的结果
```