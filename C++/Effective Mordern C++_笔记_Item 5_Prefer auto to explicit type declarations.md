很多时候我们定义一个变量，都是直接的` int x; `根据编译器的不同，没有初始化的变量往往会有不同的未定义结果，进而产生各种各样的问题，在《Effective C++》中也有建议要养成初始化变量的习惯，使用auto可以省去判断各种变量类型的麻烦，但是auto是根据初始化值来判断的，所以**变量必须初始化**。

除此之外，很多时候为了声明一个变量的类型，往往需要很长很啰嗦的前缀，比如在一个模板中，声明一个迭代器指向目标的类型：
```cpp
template<typename It> 
void dwim(It b, It e) 
{ 
while (b != e) {
    typename std::iterator_traits<It>::value_type
	currValue = *b;
	}
}
```
这样的代码让人看着就头疼...有了auto就可以直接 `auto currValue = *b` 了。

* auto用于返回值的类型判断
```cpp
auto derefUPLess = 
	[](const std::unique_ptr<Widget>& p1, 
	const std::unique_ptr<Widget>& p2) 
	{ return *p1 < *p2; };
```
C++14中auto可以直接用于lambda表达式的参数中，那么我们甚至可以进一步的简化：
```cpp
auto derefUPLess = 
	[](const auto& p1, const auto& p2) 
	{ return *p1 < *p2; };
	//简直不要太方便！！！
```

#### `auto`还有其他的优点。
例如在使用`auto`和`std::function`时，两者的内存占用和运行效率就会有区别。
std::function是C++11中的一个可调用对象的模板，类似函数指针的作用，可以用来定义函数，lambda表达式和重载了`()`运算符的类。下面的例子：
```cpp
bool(const std::unique_ptr<Widget>&,const std::unique_ptr<Widget>&)
```
这样的话上面的 derefUPLess这个函数就可以重写为不带auto的版本:
```cpp
std::function<bool(const std::unique_ptr<Widget>&,const std::unique_ptr<Widget>&)> fuc;

fuc= [](const std::unique_ptr<int>& p1,const std::unique_ptr<int>& p2) 
{
    return *p1 < *p2;
  };
```
虽然函数的目的都是一样的，但是内在还是有一些本质的区别，`auto`声明的是一个闭包类型，它所占内存即闭包类型需要的内存大小，而`std::function`是一个模板类，它声明的类型是模板的一个实例，但其分配的内存大小是固定的，当遇到内存大小不足时，会重新分配堆内存。结果就是：std::function对象基本会比auto对象占用更多的内存，且运行效率不及auto对象。

还有容器的size_type类型本质是unsigned，但是unsigned在32位和64位系统上的大小分别是32bits和64bits，如果用auto声明，不仅可以避免写出std::vector<int>::size_type这样啰嗦的代码，还能避免不同机器下的大小不同问题。

再看别的特殊情况：
```cpp
std::unordered_map<std::string, int> m;

for (const std::pair<std::string, int>& p : m)
{
… 
}
```
看起来没什么大问题，但是在unordered_map中，key是const类型的，我相信大多数人都会忽略这一点，平常的编码习惯也让我们直接这么写了，所以在哈希表中pair的类型应该是<const string , int>，无法被引用，于是编译器就会进行两个pair类型之间的转换（虽然只有一个const的区别，但这是两个完全不同的个pair类型），建立一个临时的std::pair<std::string, int>对象，然后让p引用它，一次循环结束再将这个临时对象销毁，造成了大量不必要的操作，资源浪费，以及运行效率的降低。而简单的auto又把这些我们不容易注意到的点给避免了。
```cpp
for (const auto& p : m)
{
… 
}
```

C++是一门强类型语言，有些人会担心`auto`将降低对变量类型的把握能力，这些其实可以借助Item4的办法在一定程度上解决，但在大多数情况下，自动推导的`auto`类型比一个容易出错的确定类型更能提高我们的效率，所以在大部分情况下，还是要多用auto的呀！！！