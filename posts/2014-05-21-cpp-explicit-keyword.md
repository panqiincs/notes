---
title: C++的explicit关键字
date: 2014-05-21 15:17:35
tags:
  - c++
  - explicit
  - 类型转换
categories:
  - 编程语言
---

C++的explicit关键字修饰构造函数（或类型转换函数，C++11），表明该构造函数是explicit的，不能用作隐式的类型转换和拷贝初始化。

下面的代码中，类的构造函数没有explicit关键字修饰：

<!--more-->

```cpp
class A {
public:
    A(string s) : name_(s) { }
    string getName() { return name_; }
private:
    string name_;
}

// 参数类型为A
void useA(A a)
{
    return a.getName();
}
```

所以，隐式的类型转换和拷贝初始化是合法的：

```cpp
string name = "Lucy";

cout << useA(name) << endl;  // 正确：形参name执行了从string到A的隐式转换

A a1(name);                  // 正确：直接初始化
A a2 = name;                 // 正确：拷贝初始化，包含隐式转换
```

顺带说明一下，编译器只允许一次类型转换，执行了两次隐式类型转换的用法是错误的，后面给出了正确的用法：

```cpp
// 有两次隐式转换，C风格字符串 -> string -> A，错误！
cout << useA("Lucy");

// 正确，显式转换成string，隐式转换成A
cout << useA(string("Lucy"));
// 正确，隐式转换成string，显式转换成A
cout << useA(A("Lucy"));
```

下面是加了explicit关键字修饰的版本：

```cpp
class B {
public:
    B(string s) : name_(s) { }
    string getName() { return name_; }
private:
    string name_;
}

// 参数类型为B
void useB(B b)
{
    return b.getName();
}
```

构造函数为explicit，所以隐式的类型转换和拷贝初始化是不合法的。尽管编译器不会将explicit的构造函数用于隐式转换过程，但我们可以用它来显式地强制进行转换：

```cpp
string name = "Lucy";

cout << useB(name);                  // 错误：拒绝从string到A的隐式转换
cout << useB(B(name));               // 正确：显式构造一个B对象，然后作为参数传入
cout << useB(static_cast<B>(name));  // 正确：static_cast执行显式的转换

B b1(name);                          // 正确：直接初始化
B b2 = name;                         // 错误：拒绝包含了隐式转换的拷贝初始化
B b3 = B(name);                      // 正确：显式构造一个B对象，拷贝到b3
B b4 = static_cast<B>(name);         // 正确：static_cast执行显式的转换
```

应尽可能将构造函数声明为explicit，避免隐式的类型转换，可以减少出错。
