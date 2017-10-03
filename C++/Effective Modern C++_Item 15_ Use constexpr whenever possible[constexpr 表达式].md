> 学习+使用`C++`已经有挺长一段时间了，时间越长越是感叹C++语法之多之深，所以那些简历上会写`精通C++`的人权当是不懂事了。
>  `constexpr`是C++11新加入的一个关键字，我一直对它的用法有些困惑，所以平时写项目的时候从来不敢乱加这个关键字，在这一个Item的开头，作者也写着：
>  >If there were an award for the most confusing new word in C++11, constexpr would probably win it.

> 所以很有必要对这个关键字做一个总结。

`constexpr`，可以明显知道是`const-expression`，常数表达式的意思，它表明其修饰的值不仅仅是`const`的，而且还是编译期就可知的，如果只是用`constexpr`修饰简单的表达式，那其实很好理解。

    constexpr auto size=10;

意思就是size这个变量是一个编译期可知的常量，其值是10。
这里要和`const`区别一下：

* const并不区别修饰对象是编译期还是运行期是常量，总之它是个常量
* 而constexpr限制其对象是编译期就可知的常量 （如果需要通过以变量的形式指定一个array的大小，那么应该用constexpr而不是const）

所以`constexpr`的语义比`const`更强。

但是`constexpr`的微妙之处在于它可以用于修饰函数——当它用于修饰函数时，并不意味着函数的返回值是`const`或编译期可知的，这就很有趣了（这不是瞎搞事情么...这是我的第一反应）

然而真相是：

* **`constexpr`可用于指定上下文中传入函数的参数必须是编译期可知的常量，否则编译不会通过**
* **如果`constexpr`函数被调用时，传入的部分参数是运行时可知的，那么它就会表现的和普通的函数一样**

有那么一个场景我们经常会碰到，至少我经常碰到：在指定`std::array`的大小时，我们通常希望能够按需更灵活地指定，而不是直接简单单的`std::array<int,10>,std::array<double,100>`，我们可以通过`constexpr`做到：
```cpp
constexpr // pow's a constexpr func
int pow(int base, int exp) noexcept // that never throws
{
… // impl is below
}
constexpr auto numConds = 5; // # of conditions
std::array<int, pow(3, numConds)> results; // results has
// 3^numConds
// elements
```
以上通过重写一个`constexpr`类型的pow函数，数组的大小可以由`(3^numConds)`指定。

C++11的`constexpr`函数通常比简单，一般只能直接返回一个`return`表达式，在C++14对其进行了扩展，可以支持更复杂的`if-else`表达式。

这样子的话`constexpr`表达式就可以被推广到更多复杂的应用场景，只要满足编译期可知的条件就行了，甚至可以用于类的构造函数和类的`setters`（getter就不用说了只是简单的返回值而已）！

这一Item的目的就是告诉我们，尽可能的使用`constexpr`去修饰表达式或者函数，这意味着修饰之后的对象都可以被用于`上下文需要常量`的地方，这是对常量使用的一种长期保证，常量的好处就不赘述了，并且假使有错或者有误用，也能更快地在编译期发现。