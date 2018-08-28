---
layout: post
title: c++ 泛型编程
date: 2018-8-24
categories: cpp
tags: reference
excerpt: 面向对象编程（OOP）和泛型编程都能处理在编写程序时不知道类型的情况。不同之处在于：OOP能处理类型在程序运行之前都未知的情况；而在泛型编程中，在编译时就能获知类型了。
mathjax: true
---

# 模板

模板是C++泛型编程的基础。

## 函数模板

我们可以定义一个通用的**函数模板**，而不是为每个类型都定义一个新函数。

```c++
template <typename T>
int Compare(T const& v1, T const& v2)
{
    if(v1 < v2) return -1;
    if(v1 > v2) return 1;
    return 0;
}
```

当我们调用函数模板时，编译器用函数实参来为我们推断模板函数。
我们调用Compare时，编译器使用实参的类型来确定绑定到模板的参数T的类型。

```c++
auto b = Compare(1, 2);
```

实参类型是int。编译器会推断出模板实参为int，并将它绑定到模板参数T。编译器用推断出来的模板参数来为我们实例化一个特定版本的函数。模板参数不同就会实例化不同版本的函数。除了定义类型参数，还可以在模板中定义非类型参数。一个非类型参数表示一个值而非类型。

### 非类型模板参数

除了定义类型参数，还可以在模板中定义**非类型参数**。一个非类型参数表示一个值而非一个类型。
当一个模板被实例化时，非类型的参数被一个用户提供的或编译器推断出的值所代替。这些值必须是常量表达式，从而允许编译器在编译时实例化模板。

```c++
template<unsigned N, unsigned M>
int Compare(const char(&p1)[N], const char(&p2)[M])
{
    return strcmp(p1, p2);
}

auto i = Compare("hi", "mom"); //调用
int Compare(const char (&p1)[3], const char (&p2)[4]); //编译器实例化版本
```

> 一个非类型参数可以是一个整数，或者是一个指向对象或函数类型的指针或左值引用。

## 类模板

类模板与函数模板的不同之处是，编译器不能为类模板推断模板参数类型。

```c++
template<typename T>
class MyTempClass
{
public:
    MyTempClass() {}
    ~MyTempClass() {}

    void SetValue(const T &v);

private:
    T m_value;
};

template<typename T> //区分不同的版本
void MyTempClass<T>::SetValue(const T &v)
{
    m_value = v;
}

MyTempClass<int> t;
```

### 类模板和友元

当一个类包含一个友元声明时，类与友元各自是否是模板是相互无关的。如果一个模板包含一个非模板友元，则友元被授权可以访问所有模板实例。如果友元自身是模板，类可以授权给所有友元模板实例，也可以只授权给特定的实例。

```c++
template<typename> class BlobPtr;
template<typename> class Blob;
template<typename T>
bool operator==(Blob<T> const&, Blob<T> const&);

template<typename T>
class Blob
{
    friend class BlobPtr<T>;
    friend bool operator==<T>(Blob<T> const&, Blob<T> const&);
    //...
};
```

友元的声明用`Blob`的模板形参作为他们自己的模板实参。因此，友元关系被限定在用相同类型的实例化的Blob与BlobPtr相等的运算符之间：

```c++
Blob<char> ca; //BlobPtr<char>he operator==<char>都是本对象的友元
Blob<int> ia; //BlobPtr<int>he operator==<int>都是本对象的友元
```

一个类也可以将另一个模板的每个实例都声明为自己的友元，或者限定特定的实例为友元：

```c++
template<typename T> class Pal;

class C
{
    friend class Pal<C>; //用C实例化的Pal（Pal<C>）是C的一个友元

    template<typename T> friend class Bar; //Bar的所有实例都是C的友元，这种情况无须前置声明
};

template<typename T> class C2
{
    friend class Pal<T>; //C2的每个实例将相同实例化的Pal声明为友元

    template<typename X> friend class Bar; //Bar的所有实例都是C2的友元，这种情况无须前置声明

    friend class Foo; //不需要Foo的前置声明
}
```

#### 令模板自己的类型参数成为友元

在新标准中，我们可以将模板类型参数声明为友元：

```c++
template<typename T>
class Bar
{
    friend T; //将访问权限授予用来实例化Bar的类型。
}
```

此处我们将用来实例化`Bar`的类型声明为友元。因此，对于某个类型名`Foo`，`Foo`将成为`Bar<Foo>`的友元。

### 类模板的static成员

与任何其他类相同，类模板可以声明static成员

```c++
template<typename T>
class Foo
{
public:
    static std::size_t Count() { return ctr; }

private:
    static std::size_t ctr;
};

template<typename T>
size_t Foo<T>::ctr = 0;
```

每个Foo的实例都有其自己的static成员实例。即，对任意给定类型T，都有一个`Foo<T>::ctr`和一个`Foo<T>::Count`成员。所有`Foo<T>`类型的对象共享相同的ctr对象和count函数：

```c++
Foo<std::string> fs; //实例化static成员Foo<std::string>::ctr和Foo<std::string>::Count

Foo<int> fi1, fi2, fi3; //所有三个对象共享相同的Foo<int>::ctr和Foo<int>::Count
```

#### 使用类的类型成员

默认情况下，C++语言假定通过作用域运算符访问的名字不是类型。因此，如果我们希望使用一个模板类型参数的类型成员，就必须显示地告诉编译器改名字是一个类型。我们通过使用关键字`typename`来实现这一点：

```c++
template<typename T>
typename T::value_type top(T const& c)
{
    if(!c.empty())
        return c.back();
    else
        return typename T::value_type();
}
```

> 当我们希望通知编译器一个名字表示类型时，必须使用关键字typename，而不能使用class。

### 类的成员模板

一个类，可以包含本身是模板的成员函数。这种成员被称为**成员模板**。成员模板不能是虚函数。

```c++
template<typename T>
class MyClass
{
public:
    template<typename U>
    void Fun2() {};
};

template<typename T>
template<typename U>
void MyClass<T>::Fun()
{}
```

## 模板实参推断

从函数实参来确定模板实参的过程被称为**模板实参推断**。
在模板实参推断过程中，编译器使用函数调用中的实参类型来寻找模板实参，用这些模板实参生成的函数版本与给定的函数调用最为匹配。

### 类型转换与模板类型参数

* const转换：可以将一个非const对象的引用（或指针）传递给一个const的引用（或指针）形参。
* 数组或函数指针转换：如果函数形参不是引用类型，则可以对数组或函数类型的实参应用正常的指针转换。一个数组实参可以转换为一个指向其首元素的指针。类似的，一个函数实参可以转换为一个该函数类型的指针。

```c++
template<typename T> T Foo(T, T);

template<typename T> T Bar(T const&, T const&);

string s1("aaa");
string const s2("bbb");

Foo(s1, s2); //调用Foo(string, string); const被忽略
Bar(s1, s2); //调用Bar(string const&, string cpnst&);

int a[10];
int b[20];
Foo(a, b); //调用Foo(int*, int*);
Bar(a, b); //错误，如果形参是一个引用，则数组不会转换为指针。
```

### 函数模板显示实参

#### 指定显示模板实参

我们可以定义表示返回类型的第三个模板参数，从而允许用户控制返回类型：

```c++
template<typename T1, typename T2, typename T3>
T1 Sum(T2, T3);
```

没有任何函数实参的类型可以用来推断T1的类型。每次调用Sum时，调用者都必须为T1提供**显示模板实参**。

```c++
int i;
short s;
auto val = Sum<long>(i, s); //long Sum(int, short);

//糟糕的设计
template<typename T1, typename T2, typename T3>
T3 Sum(T2, T1);
auto v2 = Sum<short, int, long>(s, i); //用户必须指定三个模板参数。
```

#### 尾置返回类型

``` c++
template <typename IT>
auto Func(IT beg, IT end) -> decltype(*beg) //调用decltype(*beg)来获取此表达式的类型
{
    return *beg;
}

//获得元素的类型
template <typename IT>
auto Func(IT beg, IT end) -> typename remove_reference<decltype(*beg)>::value //调用remove_reference,去掉引用，使用typename来告诉编译器，type表示一个类型
{
    return *beg;
}
```

### 函数指针和模板推断

当我们用一个函数模板初始化一个函数指针或为一个函数指针赋值时，编译器使用指针的类型来推断模板实参。

```c++
template<typename T> int Compare(T const&, Tconst&);

int (*pF)(int const&, int const&) = Compare;
```

pf中的参数决定了T的模板实参类型。本例中，T的模板实参类型为int。指针pf指向Compare的int版本实例。如果不能从函数指针类型确定模板实参，则产生错误：

```c++
//func的重载版本；每个版本接受一个不同的函数指针类型
void Func(int(*)(string const&, string const&));
void Func(int(*)(int const&, int const&));

Func(Compare); //错误，使用Compare的哪个实例？
```

我们可以通过使用显示模板实参来消除func调用的歧义

```c++
Func(Compare<int>); //传递Compare(int const&, int const&);
```

### 模板实参推断和引用

为了理解如何从函数调用进行类型推断，考虑下面的例子：

```c++
template<typename T> void f(T& p);
```

其中函数参数p是一个模板参数T的引用，非常重要的是记住两点：编译器会应用正常的引用绑定规则；const是底层，不是顶层的。

#### 从左值引用函数参数推断类型

当一个函数是模板类型参数的一个普通（左值）引用时（T&），绑定规则告诉我们，只能传递给他一个左值（如，一个变量或一个返回引用类型的表达式）。实参可以是const类型，也可以不是。如果实参是const的，则T将被推断为const类型：

```c++
template<typename T> void Func(T&); //实参必须是一个左值
//对Func的调用，使用实参所引用的类型作为模板参数类型
Func(i); //i是一个int，模板参数类型T是int
Func(ci); //ci是一个const int；模板参数T是const int
Func(6); //错误：传递给一个&参数的实参必须是一个左值
```

如果一个函数参数的类型时T const&，正常的绑定规则告诉我们可以传递给它任何类型的实参——一个对象（const或非const）、一个临时对象或是一个字面常量值。当函数参数本身是const时，T的类型推断的结果不会是一个const类型。const已经是函数参数类型的一部分，因此，它不会是模板参数的一部分：

```c++
template<typename T> void Func2(T const&); //可接受一个右值
//Func2中的参数是const&；实参中的const是无关的
//在每个调用中，Func2的函数参数都被推断为int const&
Func2(i); //i是一个int，模板参数T是int
Func2(ci); //ci是一个int const，但模板参数T是int
Func2(6); //一个const&参数可以绑定到右值，T是int
```

#### 从右值引用函数参数推断类型

当一个函数参数是一个右值引用（T&&）时，正常的绑定规则告诉我们可以传递给它一个右值

```c++
template <typename T> void Fun3(T &&);
Fun3(10); //实参是一个int类型的右值，模板参数T是int
```

#### 引用折叠和右值引用参数

当我们将一个左值传递给函数右值引用参数，且此右值引用指向模板类型参数（T &&）时，编译器推断模板类型参数为实参的左值引用类型。因此，当我们调用`Func3(i)`时，编译器推断T的类型为int&，而非int。
如果我们间接创建一个引用的引用，则这些引用形成了“折叠”。在所有情况下（除一例外），引用会折叠成一个普通的左值引用类型。
在新标准中，折叠规则扩展到右值引用。只有在一种特殊情况下（例外）引用会折叠成右值引用：右值引用的右值引用。
对一个给定类型X：

* X& &、X& &&和X&& &都折叠成类型X&
* 类型X&& &&折叠成X&&

这两个规则导致了两个重要结果：

* 如果一个函数的参数是一个指向模板类型参数的右值引用（T&&），则它可以被绑定到一个左值
* 如果实参是一个左值，则推断出的模板实参类型将是一个左值引用，且函数参数将被实例化为一个左值引用参数（T&）

#### 编写接受右值引用参数的模板函数

模板参数可以推断为一个引用类型，这一特性对模板类的代码可能令人惊讶的影响：

```c++
template <typename T> void Func(T&& val)
{
    T t = val;
    t = Bar(t);
    if(t == val)
    {
        //若t是引用类型，则一直为true
    }
}
```

当我们对一个右值调用`Func`时，例如字面值常量8，T为`int`。在此情况下，局部变量t的类型为`int`，且通过拷贝参数val的值被初始化。当我们对t赋值时，参数val保持不变。
另一方面，当我们对一个左值i调用`Func`时，则T为`int&`。当我们对t赋值时，赋予它类型`int&`。因此，对t的初始化将其绑定到val。当我们对t赋值时，也同时改变了val的值。在`Func`的这个实例化版本中，`if`判断是`true`。

> 在实际中，右值引用通常用于两种情况：**模板转发其实参**或**模板被重载**

目前应该注意的是，使用右值引用的函数模板通常使用如下重载：

```c++
template <typename T> void f(T &&); //绑定到非const右值
template <typename T> void f(const T &); //绑定到左值和const右值
```

第一个版本将绑定到可修改的右值，而第二个版本将绑定到左值或const右值

## 模板与重载

函数模板可以被另外一个模板或普通非模板函数重载。
如果涉及函数模板，则函数匹配规则会在以下几方面收到影响：

* 对于一个调用，其候选函数包括所有模板实参推断成功的函数模板实例。
* 候选的函数模板总是可行的，因为模板实参推断会排除任何不可行的模板。
* 与往常一样，可行函数（模板与非模板）按类型转换来排序。当然，可以用于函数模板调用的类型转换是非常有限的
* 与往常一样，如果恰有一个函数提供比任何其他函数都更好的匹配，则选择此函数。但是，如果有多个函数提供同样好的匹配。则：
  * 如果同样好的函数中只有一个是非模板函数，则选择此函数
  * 如果同样好的函数中没有非模板函数，而有多个函数模板，且其中一个模板比其他模板更特例化，则选择此版本
  * 否则，此调用有歧义

```c++
template <typename T> string Info(const T &);
template <typename T> string Info(T *); //指针版本

string s("hi");
cout << Info(s) <<endl; //只有第一个版本是可行的。s是一个非指针对象
cout << Info(&s) <<endl; //都可以生成可行的实例，
//Info(const string*&),第一个版本，T被绑定到string*
//Info(string *),第二个版本，T被绑定到string
//第二个版本的实例是此调用的精确匹配。第一个版本的实例需要进行普通指针到const指针的转换。
//选择第二个版本

//多个可行的版本
const string *sp = &s;
cout << Info(sp) << endl;
//此例中的两个函数模板都是可行的，而且都是精确匹配
Info(const string* &); //T被绑定到string*
Info(const string*); //T被绑定到const string
//此时，正常函数匹配规则无法区分这个两个函数，但是根据重载函数模板的特殊规则，此调用被解析为Info(T *），即更特例化的版本。
//设计这条规则的原因是，没有它，将无法对一个const的指针调研指针版本的Info
//问题在于，模板Info(const T&)本质上可以用于任何类型，包括指针类型。
```

> 当有多个重载模板对一个调用提供同样好的匹配时，应选择最特例化的版本。对于一个调用，如果一个非函数模板与一个函数模板提供同样好的匹配，则选择非模板版本。

## 可变参数模板

一个**可变参数模板**就是一个接受可变数目参数的模板函数或模板类。可变数目的参数被称为**参数包**。
存在两种参数包：

* 模板参数包，表示零个或多个模板参数
* 函数参数包：表示零个或多个函数参数

```c++
//Args是一个模板参数包；表示零个或多个模板类型参数
//args是一个函数参数包；表示零个或多个函数参数
//class...或typename... 指出接下来的参数表示零个或多个类型的列表
template <typename T, typename... Args>
void Foo(T const&, Args const& ... args);

int i = 0;
double d = 3.14;
string s = "hi";
Foo(i, d, s, 10); //包中有三个参数
Foo(i); //空包
//当我们需要知道包中有多少个元素时，可以使用 sizeof... 运算符
cout << sizeof...(Args) << endl; //类型参数数目
cout << sizeof...(args) << endl; //函数参数数目
```

## 模板特例化

```c++
//函数模板特例化
template <typename T> int Comapre(const T&, const T&);

template <> //函数模板特例化
//其中T为const char *。接受字符串常量。常量指针为char *const。所以最后特例化版本是const char *const &
int Comapre(const char *const &p1, const char *const &p2); 
//类模板特例化
template <typename T>
class Foo
{
public:
    Foo() {}
    ~Foo() {}

    void Bar() {}
}

template <>
class Foo<int>
{
public:
    Foo() {}
    ~Foo() {}

    void Bar() {}
}

//特例化成员而不是类
template<>
void Foo<int>::Bar
{}

Foo<int> f;
f.Bar(); //使用特例化版本(int)

//类模板部分特例化（半特化）partial specialization
template <typename T, typename U> class C
{
public:
    void F() {}
}

template <typename U> class C<int ,U>
{
public:
    void F() {}
}

template <typename T> class C<T ,double>
{
public:
    void F() {}
}

template <typename T, typename U> class C<T*, U>
{
public:
    void F() {}
}

template <typename T, typename U> class C<T, U*>
{
public:
    void F() {}
}

template <typename T, typename U> class C<T*, U*>
{
public:
    void F() {}
}

template <typename T> class C<T, T>
{
public:
    void F() {}
}
```

> 模板及其特例化版本应该声明在同一个头文件中。所有同名模板的声明应该放在前面，然后是这些模板的特例化版本。现场