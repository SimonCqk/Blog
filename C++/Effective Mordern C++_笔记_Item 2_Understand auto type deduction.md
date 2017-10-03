> 首先声明，在大部分情况下，auto的类型推断和Item 1中Template的推断是一致的，所以重复部分不再赘述，可以参考上一篇。

* 变量声明为`auto`，auto在此处就是扮演template中`T`的角色。
* `auto`也遵守template的三种情况分类，详情看上一篇。

####在C++11中，变量的声明有四种方式
```cpp
int x=27;
int x(27);
int x={27};
int x{27};
//四种方式的结果都是声明了一个值为27的int类型变量
```
花括号的声明方式，是通过列表初始化实现的(initializer_list)，所以这样的声明方式，在用auto推断类型时，就会有一些地方需要注意。
```cpp
auto x=27;//int类型，值是27
auto x(27);//同上
auto x={27};//std::initializer_list<int>类型，值是{27}
auto x{27};//同上
```
当然，列表中的元素必须类型完全一致，即使是int与float这样可以隐式转换的类型也不能混用。

*  对待列表初始化类型的不同是`auto`和`template`类型推断唯一的区别，如果**列表类型传入模板**，则编译不会通过！

还要注意的是，有时候我们需要auto声明一个函数的返回值，这时候，**auto应用的是template的规则**，即**不能推断列表类型**。看下面两个小例子。
```cpp
auto fun(){
	return {1,2,3};//error!
	}
```
在C++14中，lambda表达式的参数列表也可以用auto声明，但是同样的不能推断列表类型，如下。
```cpp
vector<int> v;
auto lam_exm=[&v](const auto& newValue){ v=newValue ;}; //C++14

lam_exm({1,2,3}); //error!
```

####Item 2结束！