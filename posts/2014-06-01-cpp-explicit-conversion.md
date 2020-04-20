---
title: C++的显式类型转换
date: 2014-06-01 23:20:53
tags:
  - c++
  - 类型转换
categories:
  - 编程语言
---

## 旧式的类型转换

在变量或表达式前加上类型转换符强制进行显式的类型转换，例如：

<!--more-->

```cpp
int n = 61;
char c = (char)n;     // int强制转换为char，有精度损失
char *pc = &c;
int *pi = (int *)pc;  // char *强制转换为int *
```

一般来说，将较大的算术类型赋值给较小的类型时，有潜在的精度损失，编译器一般会给出警告信息。强制类型转换明确地告诉代码的读者和编译器，代码的作者知道并且不在乎精度损失，编译器的警告信息也会关闭。

C++也提供了四种命名的强制类型转换符，分别是static_cast，const_cast，reinterpret_cast和dynamic_cast。

## static_cast

任何有明确意义的类型转换，只要不包含底层const（即所引用的对象为常量），都可以用static_cast。上面的旧式类型转换语句分别可以等效替换如下：

```cpp
int n = 61;
char c = static_cast<char>(n);
char *pc = &c;
int *pi = static_cast<int *>(pc);
```

## const_cast

const_cast只能改变运算对象的底层const，也只有const_cast才能改变表达式的常量属性。它常用于有函数重载的场合，例如，下面的函数是常量版本，参数和返回类型都是`const string`的引用：

```cpp
const string &shorterString(const string &s1, const string &s2)
{
    return s1.size() <= s2.size() ? s1 : s2;
}
```

该函数可以用来比较两个非常量的`string`，但返回的结果依然是`const string`的引用。我们需要一个新版本的函数，当实参不是常量时，返回普通的引用。使用const_cast可以做到这一点：

```cpp
string &shortString(string s1, string s2)
{
    auto &r = shorterString(const_cast<const string &>(s1),
                            const_cast<const string &>(s2));
    return const_cast<string &>(r);
}
```

上面版本的函数首先将`string &`类型的实参强制转换为`const string &`类型，然后调用了shorterString函数的const版本，const版本返回`const string &`类型，这个引用绑定在某个非常量版实参上，再将其转换回一个普通的`string &`类型。这个例子展示了const_cast可以将常量属性去除，也可以添加常量属性。

## reinterpret_cast

reinterpret_cast通常为运算对象的位模式提供较低层次上的重新解释。例如：

```cpp
char a[5] = {'a', 'b', 'c', 'd', 'e'};
printf("%c%c%c%c%c\n", a[0], a[1], a[2], a[3], a[4]);  // 打印char
int *p = reinterpret_cast<int *>(a);
printf("0x%x\n", *p);                                  // 打印int（十六进制形式）
```

首先定义了一个5字节大小的字符数组，然后将数组的首地址转换成`int *`类型的指针。上面的程序在**小端字节序**的机器上运行结果如下：

```bash
abcde
0x64636261
```

将指针解引用得到的数字0x64636261（十六进制形式）大小为4字节，从低字节到高字节依次为0x61，0x62，0x63，0x64，分别是字母a，b，c，d的ASCII码。`int`类型的整数大小只有4字节，因此第5个字母e被舍弃。数据的底层位模式没有改变，只是重新解释了一番。从这个例子也可以看出，reinterprect_cast的结果依赖于机器，假如机器使用**大端字节序**，结果又会不同。

## dynamic_cast

dynamic_cast运算符用于**运行时类型识别**（run-time type identification, RTTI）。它将基类的指针或引用安全地转换成派生类的指针或引用。

它一般用于下面的情况：想使用基类对象的指针或引用执行某个派生类操作，并且该操作不是虚函数。一般来说，只要有可能，应该尽量使用虚函数，当操作被定义为虚函数时，编译器将根据对象的动态类型自动地选择正确的函数版本。假设无法使用虚函数，则可以使用dynamic_cast运算符。与虚函数相比，使用dynamic_cast蕴藏着更多的风险。

## 尽量避免强制类型转换

强制类型转换干扰了正常的类型检查，应尽量避免使用，考虑用其他的方式来实现相同的目标。如果一定要使用，也尽量不使用旧式的强制类型转换，因为和命名的强制类型转换相比，它从表现形式上来说没有那么清晰明了。
