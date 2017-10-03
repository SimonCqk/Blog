> 从C++11开始，初始化有三种方式：括号，`=`赋值和列表（也可`=`后接列表）。

先看如下例子开个头：
```cpp
//Widget是个自定义类

Widget w1; // 调用默认构造函数
Widget w2 = w1; // 创建w2时调用`=`，不是赋值操作，调用了复制构造函数
w1 = w2; // 赋值操作，调用重载的赋值运算符
```
为了解决多种初始化方式引起的困扰，C++11引入了列表初始化。
例如对于一个`vector`可以直接这样初始化：

```vector<int>v{1,3,5};```

列表还可初始化非静态数据成员，但是圆括号就不行：
```cpp
class Widget {
…
private:
int x{ 0 }; // 
int y = 0; // 
int z(0); // 出错！
};
```
另外不可复制的对象可以用列表或一对圆括号初始化，但不能用`=`赋值，如`std::atomic`等。
```cpp
std::atomic<int> ai1{ 0 }; // OK
std::atomic<int> ai2(0); // OK
std::atomic<int> ai3 = 0; // 出错！
```
> std::atomic类型用于多并发操作，多并发操作的话推荐《C++并发编程实战》

因为列表初始化的通用性，所以它被称为“uniform initialiazation”.
但需要注意的是！**列表初始化不允许隐式类型转换**！`double to int`也不行。
```cpp
double x, y, z;
…
int sum1{ x + y + z }; // 报错！
```
如果是`=`赋值或者`()`初始化的话就会直接截断为`int`。

列表初始化还能避免C++之后一个非常诡异的语法分析问题，称之为`the most vexing parase`，可以参考wiki页面：[Most vexing parse](https://en.wikipedia.org/wiki/Most_vexing_parse%20Most%20vexing%20parse)

```cpp
Widget w1(10);	//传入10,调用一个int参数版本的构造函数
Widget w2();	//本意是调用无参版本的构造函数，但是C++语法解析为调用一个返回值为Widget的函数！！！！
Widget w3{};	//正确调用无参版本的构造函数
```
**用列表初始化就可以完美解决这样错误的语法解析。**
但是啊...它再好，也还是有坑的。在Item 2中提到，`auto`会把`{}`初始化的参数推断为`std::initializer_list`，这就给类的构造函数埋下了坑。
在一般的类中，调用时`()`和`{}`的作用一样，里面放参数，然后匹配最优的构造函数进行调用。
但是为了方便列表初始化，我们通常会添加一个接收`std::initializer_list`的构造函数，但是这样有时候就会引发一些问题了。
```cpp
class Widget {
public:
Widget(int i, bool b); // as before
Widget(int i, double d); // as before
Widget(std::initializer_list<long double> il); // 添加了列表初始化版本
…
};

Widget w1(10, true); // 调用第一个构造函数
Widget w2{10, true}; // 本应调用第一个，现在却推断为std::initializer_list类型，调用第三个，10 和 true 都被转化为 long double类型了
Widget w3(10, 5.0); // 调用第二个构造函数
Widget w4{10, 5.0}; // 和上面一样的问题，调用std::initializer_list版本构造函数，并进行了隐私转换
```
甚至复制/移动构造函数也会被劫持。
```
//类中如果有一个operator float() const;重载函数的话

Widget w5(w4); // 调用复制构造函数
Widget w6{w4}; // 调用列表初始化版本，w4->float->long double
Widget w7(std::move(w4)); //调用移动构造函数
Widget w8{std::move(w4)}; //调用std::initializer_list构造函数

```
很多编译器都会优先推断为`std::initializer_list`类型，即使完美匹配一般构造函数。
```cpp
class Widget {
public:
Widget(int i, bool b); 
Widget(int i, double d); 
Widget(std::initializer_list<bool> il); 
};

Widget w{10, 5.0}; // 错误！
//编译器判断为std::initializer_list类型，忽略了第一第二个构造函数，因为int/float能转换为bool类型，但是因为10/5.0在`{}`中不支持隐式转换，所以该函数调用出错了。
```
只有当列表中的类型完全不能转化为`std::initializer_list`中的类型时，才会回过头去匹配一般的构造函数，如果std::initializer_list中是<string>，则以上所有匹配都会是我们所期望的那样。
列表初始化的大致内容就是这些，但是还有一种边界情况需要我们注意的：
```cpp
class Widget {
public:
Widget(); // 默认构造函数
Widget(std::initializer_list<int> il); 
}; 

Widget w1; // 调用默认构造函数
Widget w2{}; // 同样调用默认构造函数！
Widget w3(); // 解析为 调用一个函数...gg了...

//如果你想通过一个空列表调用列表初始化版本的构造函数，那么只能如下：
Widget w4({}); 
Widget w5{{}}; 
//在外多套一层()或{}，这就是这个边界情况
```
最后的最后，`std::vector`也会直接受到列表初始化的影响！但是只要分清楚用法就行了，没有前面那么复杂。
```cpp
std::vector<int> v1(10, 20); //用10个值为20的int初始化v1

std::vector<int> v2{10, 20};//用两个值分别为10和20的int初始化v2
```
模板里面关于这一部分，也有其应用。
```cpp
template<typename T, 
typename... Ts> 
void doSomeWork(Ts&&... params)    //params是0或1或多个参数
{
//利用params建立T类型的对象
//T localObj(std::forward<Ts>(params)...);
//T localObj{std::forward<Ts>(params)...};
…
}


doSomeWork<vector<int>>(10,20);
doSomeWork<vector<int>>{10,20};
//最终初始化的vector是10个还是2个元素，作者也不知道，只有调用者知道
//后续会有make_unique和make_shared函数解决这个问题
```

不论是`()`还是`{}`初始化，都有其优劣处，没有孰好孰坏之分，这也是C++丰富语言特性支持的一部分，为了代码的统一和尽量减少错误，我们可以选定一种并尽量统一使用，避免混杂使用导致一些不必要的麻烦。
