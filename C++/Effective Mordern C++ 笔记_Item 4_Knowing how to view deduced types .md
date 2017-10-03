> 本篇为转载，转自[原博](http://blog.csdn.net/zhangyifei216/article/details/52661385)

在Item3中学习了C++11新特性decltype，decltype可以获取变量或者表达式的类型，但是获取到的类型只能用于定义其他的变量和类型，不能打印出来，也不能用来操作。毕竟是编译期实现，用来做类型反射就算了，那么至少也应该可以打印输出下吧，毕竟书中得来终觉醒。那么本文就介绍几种方法来得到decltype的返回类型的名字。

* IDE Editors 
最简单的就是依靠C++的IDE帮你识别出decltype的返回类型，IDE毕竟不是万能的，所以你要识别的类型要尽可能的简单，不能过于复杂。

* Compiler Diagnostic 
借助于编译器的诊断错误信息。通过错误使用decltype推导出来的类型让编译器报出编译错误，在编译错误的信息中可以发现decltype推导出来的类型名称。例如下面的这个例子:
```cpp
template<typename T>
class TD

TD<decltype(x)> xType;
```
使用g++编译后，会出现编译出错，诊断信息如下:
```cpp
错误：聚合‘TD<int> xType’类型不完全，无法被定义
   TD<decltype(x)> xType;
```
从上面的诊断信息就可以得出`decltype(x)`的结果就是`int`。
* Runtime Output 
最后一种方式就比较专业了，而且还是运行时获取，不光光可以用来验证decltype的返回类型，还可以做运行时的检查，和一些额外的操作，实现的手段则是利用了`typeinfo`
```cpp
#include <typeinfo>
int x = 0;
std::cout << typeid(decltype(x)).name() << std::endl;
```
上面会输出x的类型的名称，这里应该会输出`int`，但也不尽然，typeid的输出结果取决于编译器，MSVC的输出是`int`，而g++的输出则是`i`，也就是c++对int的名称重写后的结果。g++其实也可以实现和MSVC的输出结果一样.
```cpp
#include <string>
#include <cxxabi.h>
#include <memory>
#include <typeinfo>
#include <iostream>

std::string demangle(const char* name) {

    int status = -4; // some arbitrary value to eliminate the compiler warning

    // enable c++11 by passing the flag -std=c++11 to g++
    std::unique_ptr<char, void(*)(void*)> res {
        abi::__cxa_demangle(name, NULL, NULL, &status),
        std::free
    };

    return (status==0) ? res.get() : name ;
}

int main() {
  int x;
  std::cout << demangle(typeid(decltype(x)).name()) << std::endl;
}
```
到此为止`typeinfo`看似解决了问题，其实不然，通过typeinfo得到的类型会忽略`cv`限制符还有`引用`，真的是差强人意啊。但是对const的指针类型是不会忽略const限制符的。具体可以参考`typeid`获取完整类型

幸好可以借助于boost的`Boost.Type-Index`库得到精确的类型。
```cpp

template<typename T>
void f(const T& param) {
  using std::cout;
  using boost::typeindex::type_id_with_cvr;

  cout << "param = "
       << type_id_with_cvr<T>().pretty_name()
       << '\n';

  cout << "param = "
       << type_id_with_cvr<decltype(param)>().pretty_name()
       << '\n';
}
```