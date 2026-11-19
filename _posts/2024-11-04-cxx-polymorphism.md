---
layout: post
title:  c++ 多态拾遗
date:   2024-11-04 15:58:07
categories: c/c++
tags: 多态
---

* content
{:toc}

### 一、构造函数可以是虚函数吗
构造函数不能是虚函数。在 C++ 中，构造函数是用于初始化对象的特殊成员函数，其目的是在对象创建时进行初始化操作。虚函数的调用是通过对象的指针或引用来进行的，而在对象创建阶段，对象尚未创建完成，因此无法使用虚函数。


### 二、请问析构函数必须为虚函数吗
析构函数不一定是虚函数，但是要是有继承，则父类一定要为虚函数


### 三、虚函数表在什么时候创建？每个对象都有一份虚函数表吗？
1. 当一个类里存在虚函数时，编译器会为类创建一个虚函数表，发生在编译期。
2. 虚函数表只有一份，而有多少个对象，就对应多少个虚函数表指针。

### 四、什么是操作符重载？如何在C++中进行操作符重载？
操作符重载（Operator Overloading）是指重新定义 C++ 中的运算符，使其能够用于用户自定义类型的操作。通过操作符重载，可以使得用户自定义类型的对象可以像内置类型一样使用运算符进行操作

在 C++ 中进行操作符重载，需要定义一个成员函数或友元函数来实现运算符的重载。操作符重载函数的命名约定是使用关键字 operator 后跟要重载的运算符，如 operator+ 代表重载加法运算符。

下面是一些操作符重载的基本语法：

#### 1. 成员函数形式的操作符重载：
``` c++
class Complex {
private:
    double real;
    double imag;

public:
    Complex(double r, double i) : real(r), imag(i) {}

    // 重载加法运算符 +
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }
};

int main() {
    Complex c1(1.0, 2.0);
    Complex c2(2.0, 3.0);
    Complex result = c1 + c2; // 调用重载的加法运算符
    return 0;
}
```

#### 2. 友元函数形式的操作符重载：
``` c++
class Complex {
private:
    double real;
    double imag;

public:
    Complex(double r, double i) : real(r), imag(i) {}

    // 声明友元函数
    friend Complex operator+(const Complex& lhs, const Complex& rhs);

    double getReal() const { return real; }
    double getImag() const { return imag; }
};

// 定义友元函数
Complex operator+(const Complex& lhs, const Complex& rhs) {
    return Complex(lhs.real + rhs.real, lhs.imag + rhs.imag);
}

int main() {
    Complex c1(1.0, 2.0);
    Complex c2(2.0, 3.0);
    Complex result = c1 + c2; // 调用重载的加法运算符
    return 0;
}
```

### 五、解释直接继承可能产生的二义性，以及提供相应的解决方法

多重继承的二义性是指一个类（D）通过多条路径继承了相同的父类（A），从而导致在调用某些方法或访问某些属性时产生歧义或冲突的情况。这种情况通常被称为 “钻石继承” ，因为在类的继承图中，继承关系形成了类似钻石形状的结构。

例如，考虑如下继承关系：
``` shell
   A
  / \
 B   C
  \ /
   D
```

在这个示例中，类 D 继承自类 B 和类 C，而类 B 和类 C 都继承自类 A。如果类 B 和类 C 都定义了同名的方法或属性，并且类 D 想要调用这个方法或访问这个属性，就会导致二义性，因为不清楚应该使用哪个版本。

直接继承可能会导致问题，因为在继承链中的某个节点定义了与其他节点相同的方法或属性，从而引发了二义性。这会使代码难以理解和维护，并且可能导致意外的行为。

解决这种二义性的一种方法是使用虚拟继承（virtual inheritance）（如果是在支持该特性的编程语言中）。虚拟继承允许在类的继承链中只有一个共享的父类实例，从而消除了二义性。在C++中，可以通过在继承关系中使用虚拟继承来解决钻石继承问题。例如：

``` c++
class A { };
class B : virtual public A { };
class C : virtual public A { };
class D : public B, public C { };
```
使用虚拟继承，类 D 将只包含一个 A 的实例，而不是分别继承自类 B 和类 C 的 A 实例，从而避免了二义性。


### 六、动态类型检查和dynamic_cast
在C++中，RTTI（Run-Time Type Information）会为包含虚函数的类提供运行时类型信息。当类具有虚函数时，编译器为类生成RTTI，这些信息可用于dynamic_cast和typeid操作符。因此，在这种情况下，dynamic_cast可以用于安全地将基类指针转换为派生类指针。

然而，对于没有虚函数的类，编译器通常不会生成``RTTI``。这意味着在这种情况下，``dynamic_cast``不能用于类型转换，因为缺乏必要的运行时类型信息。如果你尝试对没有虚函数的类使用``dynamic_cast``，编译器会报错。这就是为什么我们需要使用``static_cast``进行类型转换。``static_cast``是在编译时进行的类型转换，它不依赖于``RTTI``。这意味着``static_cast``可以用于没有虚函数的类之间的类型转换。然而，这种转换是不安全的，因为它不会在运行时检查类型的兼容性。如果类型不匹配，可能会导致未定义行为。

### 七、虚函数内存分布
1. 一般继承（无虚函数覆盖）

- 虚函数按照其声明顺序放于表中。
- 父类的虚函数在子类的虚函数前面。

2. 一般继承（有虚函数覆盖）

- 覆盖的f()函数被放到了虚表中原来父类虚函数的位置。
- 没有被覆盖的函数依旧。

3. 多重继承（无虚函数覆盖）

- 每个父类都有自己的虚表。
- 子类的成员函数被放到了第一个父类的表中。（所谓的第一个父类是按照声明顺序来判断的）