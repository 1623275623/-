---
description: C++ standard 把constructor 和copy constructor 分为两种trivial 的 和 nontrivial的。
---

# 深度探索C++对象模型

## 关于对象



一个object的大小由什么决定？

1. 非静态成员的总大小
2. 内存对齐所花费的空间
3. 为了支持virtual 而产生的负担。

#### 指针的类型

一个指针不管指向什么东西，它的大小是固定的。

```
ZooAnimal *px;
int* pi;
Array<String> *pta;
```

从内存需求的角度来看上面的三个指针没有任何区别。他们三个都需要有足够的内存来存放一个机器地址，通常是一个word。指向不同类型的指针间的差异不在于其表现形式或者其实际内容。是其所保存地址的对象的具体类型。也就是说，指针类型会教导编译器如何解释某个特定地址中的内存内容及其大小。



**转换(cast)其实只是一种编译命令。大部分情况下它并不改变一个指针所含的真正地址，它只影响“被指内存的大小和其内容”的解释方式。**



##

## 构造函数语意学

#### default constructor

什么时候编译器会为我们生成一个构造函数呢？在需要的时候，这个需要的时候指的是编译器需要的时候，而不是程序需要的时候。

:interrobang: 什么时候编译器会生成一个trivial 的 copy constructor?

假如我们有下面的类，这个类编译器是不负责合成构造函数的。

```cpp
class Point
{
public:
    float x;
    float y;
    float z;
};
```

在书中说，

> 对于Class X，如果没有任何 user-defined constructor，那么会有一个默认的constructor 被隐式的声明出来。一个被隐式声明出来的constructor是一个 trivial （没什么用的） constructor。

但是自己通过compiler 得出的结果是什么都没有。

那么什么情况下算是编译器需要的情况呢？

1. 该类的成员是一个类成员，并且该成员有默认构造函数。
2. 该类的父类有默认构造函数
3. 该类有虚函数
4. 该类的父类有虚函数。

针对第一种情况来说。

> 如果一个Class A内含一个或一个以上的 member class objects，那么 class A的每一个constructor 必须调用每一个member class 的defcult constructor。&#x20;

编译器必须扩张已经存在的constructors，并且在其中安插一些代码，使得User code 被执行之前先调用必要的 constructor。

如果C++内部有多个class member objects，那么语言要求以member objets 的生明顺序来调用各自的constructor.&#x20;

在C++合成的default constructor中，只有base class subobjects 和 member class objetcs会被初始化。其他所有的nostatic data member （如整数、整数指针、整数数组等等）都不会被初始化，如果我们需要初始化，这些工作必须由我们来完成。



#### Member Initialization List

在下列是必须使用初始化列表的情形：

* 当初始化一个 reference member的时候；
* 当初始化一个const member 的时候
* 当调用一个base class的constructor的时候
* 当调用一个member class的constructor的时候



### Copy constructor的构造操作

有三种情况会调用到拷贝构造函数：

1. 一个是对Class Object做显示的初始化操作。
2. 当Object被当做参数的时候。
3. 当函数传回一个class Object的时候。

如果我们没有手动

这里我们需要注意的一点是编译器也会根据实际的情况判断是否需要合成拷贝构造函数。当我们的类的成员全部都是基础成员的时候。

一般情况下如果类内没有显式的定义拷贝构造函数，那么当用到的时候编译器会递归的使用memberwise initialization的方式拷贝。就是逐个成员的拷贝

但是这样的操作实际上是如何完成的?

一个良好的编译器可以为大部分class objects产生bitwise copies，因为它们有bitwise copy semantics;

> Copy constructor 在必要的时候才会由编译器产生出来

这个必要指的就是class 不展现bitwise copy semantics.

什么时候class 不展现出 “bitwise copy semantics”&#x20;

1. 当class 有一个member object，而且当这个member object声明了一个copy constructor的时候，不论这个copy constructor是被类设计者显示生明的，还是由于某种原因隐式声明的。
2. 当class 的父类存在 copy constructor的时候。
3. 当class生明了一个或者多个虚函数的时候
4. 当class继承自一个串链，当中有一个或者多个的virtual base classes时。

## Function 语意学

C++的设计准则之一就是：nostatic member function 至少必须和一般的nomember function有相同的效率。

实际上member function会被内化为 nonmember 的形式。下面就是转化步骤：

1. 改写函数的signature 以安插一个额外的参数到member function中，用以提供一个存取管道，使class object得以将此函数调用。额外的参数被称为this 指针。
2. 将每一个对 nonstatic data member 的存取操作 改为由this指针来进行存取。
3. 对函数的名称经过“mangling 处理”使其在程序中成为独一无二的存在。

现在函数转换完成。然而对于函数的每次的调用操作也必须转换。

```
obj.magnitude();      ----->      magnitude__7Point3dFv(&obj);
ptr-> magnitude();    ----->      magnitude__7Point3dFv(ptr);

```
