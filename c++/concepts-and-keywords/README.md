# Concepts and KeyWords

[https://learn.microsoft.com/en-us/cpp/cpp/trivial-standard-layout-and-pod-types?view=msvc-170](https://learn.microsoft.com/en-us/cpp/cpp/trivial-standard-layout-and-pod-types?view=msvc-170)

## Layout

这个单词表示的是像class、struct或者union这样的对象的成员是如何在内存中分布的。在一些情况下

但是当一个类或者struct包含某些C++特性的时候，比如虚基类、虚函数、成员变量拥有不同的访问权限，这个时候编译器就可以自由的选择Layout。这时的Layout可能取决的编译器所采取的优化设置并且很多时候对象不会在占有一块连续的内存，比如当一个类含有虚函数的时候，这些所有的类都会共享一个虚函数表。这种类型十分有用，但是也有限制。因为这个Layout的不确定性，所以其无法传到其他语言写的程序里，比如说C。由于可能占用的内存不连续就会导致不能使用更加底层高效的memcpy函数进行复制，或者通过网络进行序列化。

为了能够让编译器、C++程序、元编程等可以根据对象的不同类别而采用不同的操作。C++14引入了Class 和struct的三个类别。trivial，standard-layout，和POD或者是Plain Old Data。C++有下面的模板函数来区分这些类型。`is_trivial<T>`, `is_standard_layout<T>` and `is_pod<T>` 。

#### Trivial

当一个class 或者struct有编译器提供的或者说是显式默认的特殊成员函数时，然后就是trivial type。

#### Standard layout types

当一个类不包含特定的C++语言特性，也就是C语言没有的特性，并且所有的成员都是相同的获取权限。它就是standard-layout type.





具有上面Trivial 和 Standard Layout type 全部特征的就是Plain-Old-Data 类型。

