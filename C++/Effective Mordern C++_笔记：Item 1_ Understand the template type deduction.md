写在前面：
========
>基础的东西学完之后，最好的进阶方式就是项目+啃书了，项目也在准备当中，啃书还是绝对不能落下的。现在C++11/14已经成为主流，正好在图书馆借到了《Effective Mordern C++》，以读书笔记的形式做一个记录。而且这本书还没有中文版，所以也能当做是提高自己英文阅读能力的有效手段。

###Item 1: Understand the template type deduction.
这么一段模板的简单示例代码：
```cpp
template<typename T>
void f(ParamType expr);
```
调用f:
```cpp
f(expr)
```
编译时，编译器通过expr推断T和ParamType的类型，ParamType通常包括const和引用。
```cpp
template<typename T>
void f(const T& param);//ParamType is const T&

int x=0;
f(x);
```
这个小例子中，T被推断为`int`，ParamType被推断为`const int&`。
但是要注意的是，T的推断不仅仅依靠`expr`，同时也受`ParamType`的影响。有以下三种需要注意的情况。

* ①ParamType是指针或引用，但不是universal reference (顾名思义， universal意思是可以接受任何类型)
* ②ParamType是universal reference
* ③ParamType既不是指针也不是引用。

①：记住两点：1.如果expr的类型是引用，则引用被忽略。2.然后根据ParamType模式匹配expr的类型来确定T。
直接上例子：
```cpp
template<typename T>
void f(T& param);//param 是一个引用

int x=27;//x是一个int 
const int cx=x;//cx是一个const int
const int& rx=x;//rx是一个const int 的引用

f(x); //T是int , param是int&
f(cx);//T是const int， param是const int&
f(rx); //T是const int， param是const int&
```
传入T时，实参自带的引用都被忽略了。、
再看下面的例子。
```cpp
template<typename T>
void f(const T& param);//param 是一个ref-to-const

int x=27;//x是一个int 
const int cx=x;//cx是一个const int
const int& rx=x;//rx是一个const int 的引用

f(x); //T是int , param是const int&
f(cx);//T是const int， param是const int&
f(rx); //T是const int， param是const int&
```
和前面的例子一样，rx自带的`引用`属性被忽略了。
指针同上。

②：
这里类型推断变得比较抽象起来。universal ref被声明为得像右值引用(e.g. T&&)，当左值传入时，类型推断会有一些不一样。
关于左右值，可以参考博客：[C++11/左值/右值][左值右值]

[左值右值]: http://blog.csdn.net/hyman_yx/article/details/52044632
同样需要记住两点：1.当`expr`是左值，`T`和`ParamType`都被推断为左值。2.当`expr`是右值，则按之前的规则推断类型。
上例子：
```cpp
template<typename T>
void f(T&& param);//param 是一个 universal ref

int x=27;//x是一个int 
const int cx=x;//cx是一个const int
const int& rx=x;//rx是一个const int 的引用

f(x); //x是左值，所以T是int&，param是int&
f(cx);//cx是左值，所以T是const int&，param是const int&
f(rx); //rx是左值，所以T是const int&，param是const int&
f(27); //27是右值，所以T是int , param 是 int&&
```
这些规则在《C++ Primer 5th》当中也有描述，被称作引用折叠：
> * X& &,X& &&和X&&　&都折叠成类型&
* 类型X&& &&折叠成X&&。


③：
```cpp
template<typename T>
void f(T param); //param通过值传入
```
还是记住两点：1.expr's带引用，则忽略引用。2.除此之外，const 和 volatile 性质同样被忽略。

- 只要记住，是通过值传入，passed by value ！！！

但看一个很特殊的例子：
```cpp
template<typename T>
void f(T param) 

const char* const ptr="test!";

f(ptr);  
```
这是一个const指针指向一个const对象，当值传入时，指针自身的const属性被忽略，但是它所指对象的const属性被保留了，这个需要特别注意一下！
③：
```cpp
template<typename T>
void f(T param); //param通过值传入
```
还是记住两点：1.expr's带引用，则忽略引用。2.除此之外，const 和 volatile 性质同样被忽略。

- 只要记住，是通过值传入，passed by value ！！！

但看一个很特殊的例子：
```cpp
template<typename T>
void f(T param) 

const char* const ptr="test!";

f(ptr);
```
这是一个const指针指向一个const对象，当值传入时，指针自身的const属性被忽略，但是它所指对象的const属性被保留了，这个需要特别注意一下！
####数组参数
虽然数组和指针在大部分情况下可以互相转化，但是某些情况还是需要注意一下——即数组作为一个参数传入`值传入`的模板时。
一般的模板，数组名（假设为char name[]）传入，会被推断为const char* .
但是！重点来了，虽然一般的推断会将数组名自动转换为相应的数组，但是模板中，可以`引用真正的数组！`。
既然模板参数可以引用数组，那么就可以利用此特性写一些奇技淫巧了。
我们可以直接推断出数组的大小。

```cpp
//返回值是编译期常量值的数组大小
//数组匿名，因为我们只关心数组包含的元素数目
template<typename T,size_t N>
constexpr size_t arraySize(T (&)[N]) noexcept
{
//  constexpr 使得在编译期常量结果已知
return N;
}

int keyVals[]={1,8,9,3,4,5};
//然后就可以这么用了
int mappedVals=[arraySize(keyVals)];
//我们也应该学着用新标准更轻量化标准化的array
array<int,arraySize(keyVals)>mappedVals;
```

> 函数作为一个参数时，同数组，默认也被推断为函数数组。T&则是函数的引用。

## 差不多的要点都在这了，Item 1 完结。