---
title: C++的const成员函数
date: 2018-02-04 12:20:28
tags:
  - c++
  - this指针
  - const
categories:
  - 编程语言
---

在类的成员函数的参数列表后面加上关键字const，表示它是一个const成员函数。考虑下面的类：

<!--more-->

```cpp
class Foo {
public:
    Foo(): value_(0) {}
    Foo(int val): value_(val) {}
    
    // const成员函数
    int cvalue() const { return value_; }
    // 非const成员函数
    int value() { return value_; }
    void setValue(int a) { value_ = a; }
private:
    int value_;
};
```

其中，`value()`和`cvalue()`功能相同，但`cvalue()`是const成员函数，`setValue()`修改了类的成员变量。我们分别创建类的普通对象`f`和常量对象`cf`，并调用成员函数：

```cpp
Foo f;
const Foo cf;
    
f.value();         // OK
f.cvalue();        // OK
f.setValue(1);     // OK
cf.value();        // 编译Error
cf.cvalue();       // OK
cf.setValue(1);    // 编译Error
```

`cf`是常量对象，无法调用修改对象内容的`setValue()`，可以理解，但`cf`也无法调用`value()`，只能调用const成员函数`cvalue()`！要解释这个问题，首先介绍一下this指针。

调用成员函数实际上是在替某个对象调用它。成员函数通过一个名为**this指针**的额外隐式参数来访问调用它的那个对象。`value()`是普通的成员函数，`int Foo::value()`的实际形式是`int Foo::value(Foo *const this)`，即，this本身是一个常量（注意，不是指向常量的指针）。在调用成员函数时，编译器将调用过程`f.value()`重写为`int Foo::value(&f)`。

当调用常量对象`cf`的成员函数时，用`cf`对象的地址初始化this指针。根据初始化规则，只有指向常量的指针才能够存放常量对象的地址。正是这个原因，在常量对象上调用非常量成员函数是非法的。因此，我们应该把this声明成`const Foo *const`，毕竟`value()`的函数体内不会改变this所指的对象。C++的做法是将const关键字放在成员函数的参数列表之后，也就得到了const成员函数`cvalue()`。

综上所述，我们可以得到const成员函数的特点：

1. 使得接口容易理解，提醒调用者，该const成员函数无法修改类对象的内容
2. 使得操作“常量对象”成为可能，因为常量对象只能调用const成员函数

所以，对于那些不修改对象内容的成员函数，应尽可能地定义成const，不仅能给接口的调用者传递信息，还能够扩大使用范围。
