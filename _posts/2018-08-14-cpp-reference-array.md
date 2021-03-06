---
layout: post
title: C++ 数组
date: 2018-8-14
categories: cpp
tags: reference
excerpt: 数组是一种复合类型。数组的声明形如a[d]，其中a是数组的名字，d是数组的维度。维度说明了数组中元素的个数。
mathjax: true
---

# 显示初始化数组

数组的维度必须是一个常量表达式。

```c++
constexpr int sz = 10; //常量表达式
int arr[sz];
```

可以对数组的元素进行列表初始化，此时允许忽略数组的维度。如果在声明是没有指明维度，编译器会根据初始值的数量计算并推测出来；相反，如果指明了维度，那么初始值的总量不应该超出指定的大小。如果维度比提供的初始值大，则用提供 的初始值初始化靠前的元素，剩下的元素被初始化成默认值。

## 一维数组

```c++
int a1[2] = {0, 1};
int a2[] = {0, 1};
int a3[5] = {0, 1, 2}; //等价于a3[] = {0, 1, 2, 0, 0};
char a4[] = {'c', '+', '+'}; //size = 3
char a5[] = {'c', '+', '+', '\0'}; //size = 4
char a6[] = "c++"; //size = 4，自动添加表示字符串结束的空字符
char a7[] = a6; //错误，不允许使用一个数组初始化另外一个数组
a2 = a6; //错误，不能把一个数组直接赋值给另外一个数组。
```

## 二维数

二维数组在存储时按行优先连续存储，数组名是一个二维指针，如 int a[3][2] 中，a 是一个二维指针，而a[0],a[1],a[2]都相当于普通的一位数组的数组名，是一个固定值的指针。

```c++
int a[2][3] = {0}; //所以元素都初始化为0
int a[2][3] = { 1,2,3,4,5,6 };
int a[2][3] ={ {1,2,3}, {4,5,6} };
int a[][3] = { 1, 2, 3, 4, 5, 6 } ; //省略第一维的定义，但是不能省略第二维的定义
```

# 复杂的数值声明

默认情况下类型修饰符从右向左依次绑定。要想理解数组声明的含义，最好的办法是从数组的名字开始按照由内向外的顺序阅读。

```c++
int a[10];
int *ptrs[10]; //ptrs是含有10个int*的数组
int (*pa)[10] = &a; //pa指一个含有10个整数的数组
int (&ra)[10] = a; //ra引用一个含有10个整数的数组
int *(&ra2)[10] = ptrs; //ra2是数组的引用，该数组含有10个指针
```

> 使用数组下标时，通常将其定义为size_t类型。size_t是一种机器相关的无符号类型，它被设计得足够大以便能表示内存中任意对象的大小。

# 数组与指针

对数组的元素使用取地址符就能得到指向该元素的指针。

```c++
int nums[] = {1, 2, 3, 4, 5};
int *p = &nums[0]; //p指向nums的第一个元素
```

在很多时候用到数组名字的地方，编译器都会自动地将其替换为一个指向数组首元素的指针。

```c++
int *p2 = nums; //等价于 p2 = &nums[0]
```

只要指针指向的是数组中的元素（或者数组尾元素的下一个位置），都可以执行下标运算。**内置数组的下标可以是负数**

```c++
int *p3 = &nums[2];
int j = p3[1]; //p3[1]等价于*(p3 + 1)，就是nums[3]指向的那个元素
int k = p3[-2]; //p3[-2]等价于*(p3 - 2)，就是nums[0]指向的那个元素
```

> 当使用一个auto变量的初始值时，推断得到的类型是指针而不是数组。

# 数组与函数

## 数组作为函数的参数

C++语言允许将变量定义成数组的引用，基于同样的道理，形参也可以是数组的引用。

```c++
// 一维数
void add1(int (&a)[5]) //数组的引用
{
  a[0] += 10;
}

// 二维数
void add2(int b[][3]) //数组的指针
{
  b[0][0] += 10;
}
```

## 返回数组指针的函数

定义一个返回数组指针的函数，则数组的维度必须跟在函数名字的后面。然而，函数的形参列表也跟在函数名字后面且形参列表应该先于数组的维度。返回函数指针的函数形式如下所示：
*Type (\*function(parameter list)) [dimension]*
举个栗子

```c++
int (*func(int i)) [10];
```

可以按照以下的顺序来逐层理解该声明的含义：

* `func(int i)` 表示调用func函数是需要一个int类型的实参
* `(*func(int i))` 表示我们可以对函数调用的结果执行解引用操作
* `(*func(int i)) [10]` 表示解引用func的调用将得到一个大小是10的数组
* `int (*func(int i)) [10]` 表示数组中的元素是int类型

我们还可以使用类型别名简化定义

```c++
int r1[10] = { 0 };
int(*func(int i))[10] //参数列表可以省略
{
  return &r1;
}
// c++ 11，如果我们知道函数返回的指针指向哪个数组，就可以使用decltype关键字声明返回类型
decltype(r1) *func(int i)
{
    return &r1;
}
auto a = func(1);
std::cout << (*a)[0] << std::endl;

//使用typedef简化函数定义
typedef int arrType[10];
using arrType = int[10]; // c++ 11
arrType *func(int i); //func返回一个指向含有10个int类型的数组的指针
```

对于二维数，函数返回的是一个二维数组指针，而这个二维数组的尺寸是 `int arr[m][n]`

```c++
int r2[2][3] = { 0 };
int(*func2(int i))[2][3] //参数列表可以省略
{
  return &r2;
}
// c++ 11，如果我们知道函数返回的指针指向哪个数组，就可以使用decltype关键字声明返回类型
decltype(r2) *func2(int i)
{
    return &r2;
}
auto b = func2(1);
std::cout << (*b)[0][0] << std::endl;

//使用typedef简化函数定义
typedef int arrType2[2][3];
using arrType2 = int[2][3]; // c++ 11
arrType2 *func(int i); //func返回一个指向含有10个int类型的数组的指针
```

> **因为数组不能被拷贝，所以函数不能返回数组。不过，函数可以返回数组的指针或者引用**

# C-Style数组与标准库算法

使用std::begin()返回数组起始地迭代器，使用std::end()返回数组结尾（即最末元素的后一元素）的迭代器。

```c++
int a[] = { 0, 1, 2 };
auto b = std::begin(a);  // *b = 0; 指向数组a首元素的指针
auto e = end(ia); // 指向数组a尾元素的下一个位置，不能执行解引用和递增操作。

if (std::find(std::begin(a), std::end(a), 2) != std::end(a))
{
    std::cout << "found a 2 in array a!\n";
}
```

初始化vector对象：

```c++
int a[] = {1, 2, 3};
std::vector<int> ivec(std::begin(a), std::end(a));
```

# C++ 11 std::array

std::array是一个**固定长度**的类数组容器。但是不像C-style数组，它不能自动退化（decay）成指针（T*）。

```c++
#include <array>

std::array<int, 3> a1 = {1, 2, 3};
std::sort(a1.begin(), a1.end());
```

# 数组和std::shared_ptr

c++ 11中使用std::shared_ptr管理动态分配的数组，必须构造一个删除器。

```c++
auto sp = std::shared_ptr(new int[len], [](char *p){delete []p;});
```

或者删除器也可以这样写

```c++
std::shared_ptr<int> sp( new int[10], std::default_delete<int[]>() );
```

c++ 17中std::shared_ptr可以直接用来管理动态分配的数组，不须要再构造一个删除器。

```c++
shared_ptr<int[]> sp(new int[10]);
```