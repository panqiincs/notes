---
title: C++的访问控制
date: 2016-06-29 13:09:05
tags:
  - c++
  - 面向对象
categories:
  - 编程语言
---

## 访问说明符

封装是面向对象编程的基本思想之一，它实现了类的接口和实现的分离。封装能阻止用户直接访问类的内部实现，但可以通过类提供的接口使用类的功能。C++语言使用访问说明符（access specifier）来加强类的封装性。

## 成员的访问说明符

### public和private

定义在public说明符之后的成员在整个程序内可被访问，public成员定义类的接口。定义在private说明符之后的成员可以被类的成员函数访问，但是不能被使用该类的代码访问，private成员封装了类的实现细节。下面的代码：

<!--more-->

```cpp
class Foo {
public:
    int getX() { return x; }
    void setX(int n) { x = n; }
private:
    int x;
};


Foo f;
f.x = 3;           // 错误：用户不能访问类的private成员
f.setX(3);         // 正确：用户只能访问public成员
int m = f.x;       // 错误：用户不能访问类的private成员
int n = f.getX();  // 正确：用户只能访问public成员
```

用户只能通过public接口来使用类的功能，private成员被隐藏起来，只在类内部可见。不过类可以允许其他类或其他函数访问其非public成员，方法是令其他类或函数成为它的友元（friend）。

### protected

和private成员类似，protected成员对于类的用户来说是不可访问的。但protected成员对于派生类的成员和友元是可访问的。此外，protected成员还有一条重要性质：派生类的成员和友元只能通过派生类对象来访问基类的受保护成员，派生类对于基类对象中的受保护成员没有任何访问特权。下面的代码：

```cpp
class Base {
protected:
    int prot_mem;
};
class Sneaky: public Base {
    friend void clobber(Sneaky &);  // 能访问Sneaky::prot_mem
    friend void clobber(Base &);    // 不能访问Base::prot_mem
private:
    int j;
};

// 正确：clobber能访问Sneaky对象的private和protected成员
void clobber(Sneaky &s) { 
    s.j = s.prot_mem = 0;
}
// 错误：clobber不能访问Base的protected成员    
void clobber(Base &b) { 
    b.prot_mem = 0;
}
```

如果派生类及其友元能访问基类对象的protected成员，那么上面的第二个`clobber`函数虽然不是`Base`的友元，却能够改变`Base`对象的内容，简单地就规避掉了protected提供的访问保护。这显然是不合理的，所以规定这种行为是不合法的。


## 派生访问说明符

除了成员访问说明符，派生访问说明符也会影响访问权限。派生访问说明符对于派生类的成员（及友元）能否访问其直接基类的成员没有什么影响，对基类的访问权限只与基类中的访问说明符有关。派生访问说明符的目的是控制派生类用户（包括派生类的派生类）对于基类成员的访问权限。

下面的代码中，三个派生类继承自基类`Base`，分别是public继承、protected继承和private继承。三个派生类都包含基类的成员，派生类用户对这些成员的访问权限在注释中标明了：

```cpp
// A是基类
class A {
public:
    int x;  // x是public成员
protected:
    int y;  // y是protected成员
private:
    int z;  // z是private成员
};

// public继承自A
class B : public A {
    // x为public
    // y为protected
    // z无访问权限
};

// protected继承自A
class C : protected A {
    // x为protected
    // y为protected
    // z无访问权限
};

// private继承自A
class D : private A {
    // x为private
    // y为private
    // z无访问权限
};
```

使用public继承意味着派生类继承了基类的所有接口，每一个派生类对象同时也是一种基类对象，即：**public继承意味is-a（是一种）的关系**。最常见的继承方式即为public继承。

使用private继承意味着只有实现部分被继承，接口部分应略去。如果派生类以private形式继承基类，意思是派生类对象根据基类对象实现而得，即：**private继承意味implemented-in-terms-of（根据某物实现出）**。

## 参考

1. [C++ Primer第5版](https://book.douban.com/subject/25708312/)
2. [Effective C++](https://book.douban.com/subject/5387403/)
3. [Difference between private, public, and protected inheritance](https://stackoverflow.com/questions/860339/difference-between-private-public-and-protected-inheritance)
