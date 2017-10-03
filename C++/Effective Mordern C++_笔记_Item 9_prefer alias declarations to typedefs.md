> 转载自原博：[原博](http://blog.csdn.net/zhangyifei216/article/details/52751587%20%E5%8E%9F%E5%8D%9A)

C++11中引入的`std::unique_ptr`智能指针是个好用的东西，在我们使用`unique_ptr`的时候往往会写出这样的类型`std::uniqeu_ptr<std::unordered_map<std::string,std::string>>`,看上去很臃肿，因此大多数的时候我们会选择使用`typedef`进行类型的重定义，简化类型名称。可是在C++11中引入了一个`using`别名的机制，相比较而言引入这个机制会有其深层次的用意。
```cpp
typedef void(*FP)(int,const std::string&);
using FP = void(*)(int,const std::string&);
```
看上去`using`别名机制更加清晰，这或许是引入`using`别名机制的原因之一吧。
```cpp
template<typename T>
using aliasList = std::list<T>;

aliasList<int> li;

template<typename T>
struct aliasList {
    typedef std::list<T> type;
};
aliasList<int>::type li;
```

```typedef```没有办法在模板声明的作用域中做类型重定义，必须放在一个自定义类型作用域内。而`using`没有这个限制除了上面提到的两点外`using`还有一些其他的优点，比如对于嵌套类型来说不需要使用`typename`。
```cpp
template<typename T>
class Widget {
 private:
    typename aliasList<T>::type list;
};
```
使用`typedef`定义的嵌套类型在模板中需要使用`typename`，至于为什么可以参考**cppreference **
对于`using`来说则不一样，因为使用了using就不会有`::type`这样的后缀，因为这个后缀会让编译器迷惑这到底是类型还是成员变量，使用`using`则避免了这个问题，因为没有`::type`，并且`aliasLitst<T>`肯定是一个类型。尽管`using`有以上诸多优点但是C++11中的`type_trais`却大量用了`typedef`，不过好在C++14中标准委员会认识到了using是最好的方式并对C++11中的全部进行了替换。
```cpp
std::remove_const<T>::type // C++11: const T → T
std::remove_const_t<T> // C++14 equivalent

std::remove_reference<T>::type // C++11: T&/T&& → T
std::remove_reference_t<T> // C++14 equivalent

std::add_lvalue_reference<T>::type // C++11: T → T&
std::add_lvalue_reference_t<T> // C++14 equivalent
```
到了C++14上面的`::type`都可以去掉了。