> 虽然如果C++看到一个指针类型被定义为0，它会把0解析为一个空指针，但是`0`终究只是个`int`类型并不是个指针。
`NULL`也是一样的，它也被允许解析为非`int`型的整形，但不是一个指针类型。
在一些接受不同参数类型的重载函数中，如下：
```cpp
void f(int); 
void f(bool);
void f(void*);

f(0); // 调用f(int), 而不是f(void*)
f(NULL); //可能会编译错误，但一般调用f(0),不可能调用f(void*)
```
而如果调用f(NULL)，则其含义是不清晰的，如果NULL被定义为`0L`，因为`long`类型可转化为`int`，`bool`和`void*`都是可以的，这样模棱两可的问题直到C++11定义了`nullptr`才被解决，有着更清晰的语义，同样也不会有这样的困扰，因为`nullptr`不是一个整形，但是准确的说，它也不是一个确定的指针类型，可以把它理解为任意类型的指针，它的准确类型是`std::nullptr_t`，一个可以**隐式转换为任意指针类型的类型**。用`nullptr`代替`NULL`和`0`，可以使得重载函数的调用明确。

最后需要明确的是，一个`int`类型可以转化为`NULL`，但是`nullptr`不能由`int`类型转化而来，必须是一个指针类型。
```cpp
template<typename FuncType,
		 typename MuxType,
		 typename PtrType>
		 //decltype(auto)是C++14的用法
decltype(auto) lockAndCall(FuncType func, 
MuxType& mutex,
PtrType ptr)
{
MuxGuard g(mutex);
return func(ptr);
}
```
这样的一个模板函数，可以看到第一个参数是一个函数类型，第二个应该传入指针类型、
```cpp
//声明这么一个函数
void f(void*);

lockAndCall(f,0);//编译失败，0不能转化为void*
lockAndCall(f,NULL);//同上
lockAndCall(f,nullptr);//成功
```