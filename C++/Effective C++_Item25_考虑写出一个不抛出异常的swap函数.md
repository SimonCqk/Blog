> 本文转载，原文见：[原博](http://blog.csdn.net/u013074465/article/details/45007575)

> ①如果 std::swap 对于你的类型来说是低效的，请提供一个 swap 成员函数，并确保你的 swap 不会抛出异常。
②如果你提供一个成员 swap，请同时提供一个调用成员swap的非成员swap。对于类（非模板），还要特化 std::swap。
③调用swap时，请为std::swap使用一个using声明式，然后在调用 swap时不使用任何namespace修饰符。
④为“用户定义类型”全特化 std 模板是好的，但绝不要尝试在std中加入任何全新的东西。

 swap是一个有趣的函数。最早作为`STL`的一部分被引入，后来它成为异常安全编程`（exception-safeprogramming）`的支柱，和用来处理自我赋值可能性的常见机制。因为 swap太有用了，所以正确地实现它非常重要，但是伴随它不同寻常的重要性而来的，是一系列不同寻常的复杂性。

          swap两个对象的值就是互相把自己的值赋予对方。缺省情况下，swap动作可由标准程序库提供的swap算法完成，其典型的实现完全符合你的预期：

```cpp
namespace std {  
template<typename T> // std::swap的典型实现，置换a和b的值  
void swap(T& a, T& b)  
{  
    T temp(a);  
    a = b;  
    b = temp;  
}  
}  
```
只要你的类型支持拷贝（通过拷贝构造函数和拷贝赋值运算符），缺省的swap实现就能交换类型为T的对象，而不需要你做任何特别的支持工作。它涉及三个对象的拷贝：`从a到temp，从 b到a，以及从temp到b`。对一些类型来说，这些赋值动作全是不必要的。

这样的类型中最重要的就是那些由一个指针组成，这个指针指向包含真正数据的类型。这种设计方法的一种常见的表现形式是`"pimpl手法"（"pointerto implementation"）`。如果以这种手法设计`Widget `类，可能就像这样：

```cpp
class WidgetImpl { // 针对Widget数据设计的类  
public:  
    ...  
  
private:  
    int a, b, c; // 可能有很多数据，意味着复制时间很长  
    std::vector<double> v;  
    ...  
};  
  
class Widget { // 这个类使用pimpl手法  
public:  
    Widget(const Widget& rhs);  
    Widget& operator=(const Widget& rhs)  
    { // 复制Widget时,令其复制WidgetImpl对象  
        ...    
            *pImpl = *(rhs.pImpl);  
        ...  
    }  
    ...  
  
private:  
    WidgetImpl *pImpl; // 指针，所指对象内含Widget数据  
};   
```
 为了交换这两个Widget对象的值，我们实际要做的就是交换它们的`pImpl`指针，但是缺省的交换算法不仅要拷贝三个Widgets，而且还有三个`WidgetImpl`对象，效率太低了。当交换 Widgets的是时候，我们应该告诉`std::swap`我们打算执行交换的方法就是交换它们内部的 `pImpl`指针。这种方法的正规说法是：针对`Widget`特化`std::swap`。如下令Widget声明一个名为是`swap`的`public`成员做真正的交换工作，然后将`std::swap`特化，令它调用该成员函数：
```cpp
class Widget {  
public:  
    ...  
    void swap(Widget&other)  
    {  
        using std::swap; // 此声明是必要的  
        swap(pImpl, other.pImpl); // 若要置换Widget就置换其pImpl指针  
    }  
    ...  
};  
  
namespace std {  
  
    template<> // 这是std::swap针对“T是Widget”的特化版本  
    void swap<Widget>(Widget& a, Widget& b)  
    {  
        a.swap(b); // 若要置换Widget, 调用其swap成员函数  
    }  
}  
```
这样不仅能够编译，而且和`STL`容器保持一致，所有`STL`容器都既提供了`public swap`成员函数，又提供了`std::swap`的特化来调用这些成员函数。这个函数开头的`"template<>"`表明它是`std::swap`的一个全特化版本，函数名后面的`"<Widget>"`表明这一特化版本针对“T是Widget” 而设计。换句话说，当通用的`swap`模板用于`Widgets`时，便会启用这个版本。通常，我们改变`std namespace`中的内容是不被允许的，但允许为为标准模板（如swap）制造特化版本，使它专属于我们自己的类（如Widget）。
           
可是，假设Widget和WidgetImpl是类模板而不是类，或许我们可以试图将WidgetImpl中的数据类型加以参数化：

```cpp
template<typename T>  
class WidgetImpl { ... };  
template<typename T>  
class Widget { ... };  
```
可能的方法一：在`Widet`内（以及`WidgetImpl`内，如果需要的话）放一个swap成员函数就像以往一样简单，但是我们却在特化std::swap时遇到乱流。我们想写成这样：

```cpp
namespace std {  
template<typenameT>  
void swap<Widget<T> >(Widget<T>&a, Widget<T>& b)  
{ a.swap(b); }  
}                    //错误，不合法！  
```
我们企图偏特化一个函数模板，这行不通；C++只允许偏特化类模板。
可能的方法二：想偏特化一个函数模板，惯常做法是简单地为它添加一个重载版本：

```cpp
namespace std {  
template<typename T> // std::swap的一个重载版本  
void swap(Widget<T>& a, Widget<T>&b)  
{ a.swap(b); }  
}                   //这也不合法  
```
 通常，重载函数模板没有问题，但是std是一个特殊的命名空间，其规则也比较特殊。它认可完全特化std中的模板，但它不认可在std中增加新的模板（或类，函数，以及其它任何东西）。std的内容完全由C++标准委员会决定，其禁止我们膨胀那些已经声明好的东西。不要添加新东西到std内。

正确的方法，既使其他人能调用swap，又能让我们得到更高效的模板特化版本。我们还是声明一个非成员swap来调用成员swap，只是不再将那个非成员函数声明为std::swap的特化或重载。例如，如果Widget相关机能都在namespace WidgetStuff中：
```cpp
namespace WidgetStuff {  
    ... // 模板化的WidgetImpl等等  
  
    template<typename T> // 内含swap成员函数  
    class Widget { ... };  
    ...  
  
    template<typename T> //非成员swap函数，这里并不属于std命名空间  
    void swap(Widget<T>& a, Widget<T>& b)  
    {a.swap(b);}  
}  
```
现在，如果某处有代码打算置换两个Widget对象，调用了swap，C++的名字查找规则将找到`WidgetStuff`中的Widget专用版本。

现在从客户的观点来看一看，假设你写了一个函数模板来交换两个对象的值，哪一个swap应该被调用呢？std中的通用版本，还是std中通用版本的特化，还是T专用版本（肯定不在std中）？如果T专用版本存在，则调用它；否则就回过头来调用std中的通用版本。如下这样就可以符合你的希望：
```cpp
template<typename T>  
void doSomething(T& obj1, T& obj2)  
{  
    using std::swap; // 令std::swap在此函数内可用  
    ...  
    swap(obj1, obj2); // 为T类型对象调用最佳swap版本  
    ...  
}
```  
当编译器看到这个swap调用，他会寻找正确的swap版本来调用。如果T是`namespace WidgetStuff`中的Widget，编译器会利用参数依赖查找`（argument-dependent lookup）`找到WidgetStuff中的swap；如果T专用swap不存在，编译器将使用std中的swap，这归功于此函数中的using声明式使std::swap在此可见。尽管如此，相对于通用模板，编译器还是更喜欢T专用的std::swap特化，所以如果std::swap对T进行了特化，则特化的版本会被使用。

 需要小心的是，不要对调用加以限定，因为这将影响C++挑选适当函数：

```cpp
std::swap(obj1, obj2);   //这是错误的swap调用方式  
```
这将强制编译器只考虑std中的swap（包括任何模板特化），因此排除了定义在别处的更为适用的T专用版本被调用的可能性。

总结：

第一，如果swap的缺省实现为你的类或类模板提供了可接受的性能，你不需要做任何事。任何试图交换类型的对象的操作都会得到缺省版本的支持，而且能工作得很好。

第二，如果swap缺省实现效率不足（这几乎总是意味着你的类或模板使用了某种pimpl手法），就按照以下步骤来做：

1.   提供一个public的swap成员函数，能高效地交换你的类型的两个对象值，这个函数应该永远不会抛出异常。

2.   在你的类或模板所在的同一个namespace中，提供一个非成员的swap，用它调用你的swap成员函数。

3.   如果你写了一个类（不是类模板），为你的类特化std::swap，并令它调用你的swap 成员函数。

最后，如果你调用swap，确保在你的函数中包含一个using 声明式使std::swap可见，然后在调用swap时不使用任何namespace修饰符。

警告： 
       ** 绝不要让swap的成员版本抛出异常。这是因为swap非常重要的应用之一是为类（以及类模板）提供强大的异常安全（exception-safety）保证。如果你写了一个swap的自定义版本，那么，典型情况下你提供一个更有效率的交换值的方法，也保证这个方法不会抛出异常。这两种swap的特型紧密地结合在一起，因为高效的交换几乎总是基于内置类型（如pimpl手法下的指针）的操作，而对内置类型的操作绝不会抛出异常。**