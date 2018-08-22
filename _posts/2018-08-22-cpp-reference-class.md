---
layout: post
title: c++ 类
date: 2018-8-22
categories: cpp
tags: reference
excerpt: 在C++语言中，我们使用类定义自己的数据类型。通过定义新的类型来反映待解决问题中的各种概念，可以使我们更容易编写，调试和修改程序。
mathjax: true
---

# 定义一个类

当定义一个类时，我们显式地或隐式地指定在此类的对象拷贝、移动、赋值和销毁时做什么。
一个类通过定义五种特殊的成员函数来控制这些操作，包括：拷贝构造函数（copy constructor）、拷贝赋值运算符（copy-assignment operator）、移动构造函数（move constructor）、移动赋值运算符（move-assignment operator）和析构函数（destructor）。

```c++
#include <iostream>
#include <string>

class CObject
{
public:
    CObject()
      : m_index(0)
    {
      std::cout << "[default constructor] m_index =" << m_index << std::endl;
    }

    //拷贝构造函数
    CObject(CObject const& object)
    {
      this->m_index = object.m_index;
      std::cout << "[copy constructor] m_index = " << m_index  << std::endl;
    }

    //移动构造函数
    CObject(CObject const&& object) noexcept
    {
      this->m_index = object.m_index;
      object.m_index = 0;
      std::cout << "[rvalue copy constructor] m_index = " << m_index  << std::endl;
    }

    //拷贝赋值运算符，返回引用是为了避免调用一次拷贝构造函数
    CObject& operator=(CObject const& object)
    {
      if (this != &object)
      {
        this->m_index = object.m_index;
      }
      std::cout << "[copy assignment operator] m_index = " << m_index << std::endl;

      return *this;
    }

    //移动赋值运算符
    CObject& operator=(CObject && object) noexcept
    {
      if (this != &object)
      {
        this->m_index = object.m_index;
        object.m_index = 0;
      }
      std::cout << "[rvalue copy assignment operator] m_index = " << m_index << std::endl;

      return *this;
    }

    //析构函数
    ~CObject()
    {
      std::cout << "[destructor]" << std::endl;
    }

    void SetIndex(int i)
    {
      m_index = i;
      std::cout << "m_index = " << m_index << std::endl;
    }

    void SetValue(std::string const& str)
    {
      m_strValue = str;
    }

    void SetValue(std::string &&str)
    {
      m_strValue = std::move(str);
    }
private:
    int m_index;
    std::string m_strValue;
};

//理论上调用fun1，先调用CObjetc的默认构造函数构造obj，
//return obj时，调用CObject的拷贝构造函数
//当CObject定义了右值拷贝构造函数时，调用右值拷贝构造函数
CObject fun1()
{
    CObject obj;
    obj.SetIndex(1);

    return obj;
}

//调用默认构造函数，构造object1
//因为返回的是引用，拷贝构造函数就不会调用了
CObject &fun2()
{
    static CObject object1;
    return object1;
}

//首先调用默认构造函数，构造object2
//当CObject定义了右值拷贝构造函数时，调用右值拷贝构造函数
//若CObject没有定义右值拷贝构造函数，则调用拷贝构造函数
CObject &&fun3()
{
    static CObject object2;
    return std::move(object2);
}

int main()
{
    CObject obj1; //调用的是默认构造函数
    CObject obj2 = obj1; //调用的是拷贝构造函数
    CObject obj3; //调用的是默认构造函数
    obj3 = obj1; //调用的是拷贝赋值运算符
    CObject obj4; //调用的是默认构造函数
    obj4 = fun1(); //fun1()是右值，调用右值引用拷贝赋值运算符
    CObject obj5 = fun2(); //调用拷贝构造函数
    CObject obj7 = fun3(); //调用右值引用拷贝构造函数

    CObject obj;
    std::string str = "abc";
    obj.SetValue(str); //调用const &版本
    obj.SetValue(std::move(str)); //调用右值引用版本
    obj.SetValue("123"); //调用右值引用版本

    return 0;
}
```

> 如果一个类定义了拷贝构造函数，但没有定义移动构造函数。那么，在此情况下，编译期不会合成移动构造函数，这意味着此类将有拷贝构造函数但不会有移动构造函数。如果一个类没有移动构造函数。函数匹配规则保证该类型的对象会被拷贝，即使我们试图通过调用std::move来移动它，都会调用拷贝构造函数。

## 阻止拷贝

在新标准(C++ 11)发布之前，类是通过将其拷贝构造函数和拷贝赋值运算符声明为private的来阻止拷贝

```c++
class NonCopy
{
public:
    NonCopy();
    ~NonCopy();
private:
    NonCopy(const NonCopy&);
    NonCopy &operator=(const NonCopy&);
}
```

在新标准下，可以通过将拷贝构造函数和拷贝赋值运算符定义为删除的函数来阻止拷贝。

```c++
class NonCopy
{
public:
    NonCopy();
    ~NonCopy();

    NonCopy(const NonCopy&) = delete;
    NonCopy &operator=(const NonCopy&) = delete;
}
```

## 友元

类可以允许其他类或者函数访问它的非公有成员，方法是令其他类或者函数成为它的友元。

### 友元函数

如果类想把一个函数作为它的友元，只需要增加一条以`friend`关键字开始的的函数声明语句：

```c++
class A
{
friend void PrintA(A const& a);

public:
    A() {}
    ~A() {}

private:
    int i{ 1 };
};

void PrintA(A const& a)
{
    std::cout << a.i << std::endl;
}
```

### 友元类

类可以把其他类定义为友元

```c++
class A
{
friend class B;

public:
    A() {}
    ~A() {}

private:
    int i{ 1 };
};

class B
{
public:
    B() {};
    ~B() {};

    void PrintB()
    {
        std::cout << m_a.i << std::endl;
    }

private:
    A m_a;
};
```

### 友元成员函数

类还可以把其他类的成员函数定义成友元。

```c++
class A;
class C
{
public:
    C() {};
    ~C() {};

    void PrintC(A const&);
};

class A
{
friend void C::PrintC(A const&);

public:
    A() {}
    ~A() {}

private:
    int i{ 1 };
};

void C::PrintC(A const& a)
{
    std::cout << a.i << std::endl;
}
```

要想令某个成员函数作为友元，我们必须仔细组织程序的结构以满足声明和定义的彼此依赖关系。我们必须按照如下的方式设计程序：

* 受限定义class C，其中声明PrintC函数，但是不能定义它
* 接下来定义class A，包括对C友元的声明
* 最后定义PrintC，此时它才可以使用A的成员

## 隐式转换

如果构造函数只接受一个实参，则它实际上定义了转换为此类类型的隐式转换机制，我们把这种构造函数称为**转换构造函数**。

```c++
class D
{
public:
    D() {}
    D(std::string const& s) : m_str{s} {}
    ~D() {}

    void Combine(D const& d)
    {
        m_str += d.m_str;
    }

private:
    std::string m_str;
};

D d("aaa");
std::string s = "bbb";
d.Combine(s);
```

只允许一步类类型转换，例如，因为下面的代码隐式地使用两种转换规则,所以它是错误的：

```c++
D d("aaa");
// 1. 把"bbb"转换成std::string
// 2. 再把这个临时的string转换成D
d.Combine("bbb");
```

如果要完成上述的调用，可以显示地把字符串转换成std::string或者D对象：

```c++
D d("aaa");
d.Combine(std::string("bbb"));

d.Combine(D("bbb"));
```

### 抑制构造函数定义的隐式转换

在要求隐式转换的程序中，我们可以通过将构造函数声明为`explicit`加以阻止：

```c++
class D
{
public:
    D() {}
    explicit D(std::string const& s) : m_str{s} {}
    ~D() {}

    void Combine(D const& d)
    {
        m_str += d.m_str;
    }

private:
    std::string m_str;
};
```

> 关键字explicit只对一个实参的构造函数有效。需要多个实参的构造函数不能用于执行隐式转换，所以无需将这些构造函数指定为explicit的。

## 类型转换运算符

类型转换运算符（conversion operator）是类的一种特殊成员函数，它负责将一个类类型的值转换成其他的类型。类型转换函数的一般形式如下所示：

```c++
operator T() const;
```

其中T表示某种类型。类型转换运算符可以面向任意类型（除了void）进行定义，只要该类型能作为函数的返回值类型。因此，我们不允许转换成数组或者函数类型，但允许转换成指针（包括数组指针和函数指针）或者引用类型。

> 一个类型转换函数必须是类的成员函数，它不能声明返回类型，形参列表也必须为空。类型转换函数通常应该是const

定义含有类型转换运算符的类

```c++
class SmallInt
{
public:
    SmallInt(int i = 0) : val(i)
    {
        if(i < 0 || i > 255)
            throw std::out_of_range("Bad SmallInt value");
    }

    operator int() { return val; }

private:
    int val;
}

SmallInt si;
si + 3; //首先将si隐式地转换成int，然后执行整数加法
```