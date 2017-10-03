> 这个`decltype`相对于前面的template和auto就更高级一点了，它不仅可以推断出一个变量的类型，还能推断出一个**表达式**的类型。

像这样：
```cpp
const int i=0;
//decltype is const int

bool f(const Class& w);
//decltype is bool(const CLASS&)

if(f(w))
//decltype is bool

vector<type> v;
//decltype v[0] is type& 
//注意operator[]返回一个容器元素的引用
//但是vector<bool>不能返回一个bool&，它返回一个全新的bool对象
```

####在C++11中，新增了尾置返回值的语法，可以利用函数参数推断出返回值类型，而不用担心类型是否在之前已知，常用于template和lambda表达式，C++14中推广到了所有函数

```cpp
//C++11中的写法
template<typename Container , typename Index>
auto authAndAccess(Container& c,Index i)->decltype(c[i])
{
	authenticateUser();
	return c[i];
}
```
考虑从以下例子：
```cpp
template<typename Container , typename Index>
auto authAndAccess(Container& c,Index i)
{
	authenticateUser();
	return c[i];
}

vector<int> v;
authAndAccess(v,5)=10;  // 并不会被编译
```
因为auto用于函数返回值时，应用的是template的规则，v[5]&的引用属性被忽略，赋值操作失败。
在C++14中，可以允许我们将`decltype`和`auto`混用：
```cpp
template<typename Container , typename Index>
decltype(auto) authAndAccess(Container& c,Index i)
{
	authenticateUser();
	return c[i];
}
```
`decltype(auto)`说明符的意思是（`auto`:返回值类型是被推断出来的；`decltype`:推断的规则用我的），这样子就能顺利编译啦。
其他情况下，这样的用法也可以被推广：
```cpp
Widget w;
const Widget& cw=w;
auto myWgt=cw;// type is Widget
decltype(auto) myWgt2=cw;//type is const Widget&
```
现在又有了新问题：authAndAccess函数有时候需要接受一个临时的容器类型，所以我们需要支持接受一个右值传入，那么我们就可以声明为Container&&使之成为一个universal ref，但是如果&&变量需要传给更深层的函数或移交给其他函数，那么右值性质就可能失效，那么我们可以利用std::forward来实现完美转发，关于std::forward可以参考博文：[std::forward-完美转发](http://blog.csdn.net/wangshubo1989/article/details/50485951%20std::forward-%E5%AE%8C%E7%BE%8E%E8%BD%AC%E5%8F%91)
最终版本：

```cpp
template<typename Container , typename Index>
decltype(auto) authAndAccess(Container&& c,Index i)
{
	authenticateUser();
	return std::forward<Container>(c)[i];
}
```
太完美了...感动的想哭...

最后还有一丢丢要注意的东西，比如我们定义了 int x=0; x是变量名，但是(x)就变成了一个表达式，C++定义(x)也是一个左值，`decltype((x))`是一个 `int&`!!!

THE END.
