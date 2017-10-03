> 特别提这一章是因为自己习惯于使用`shared_ptr`，而一直忽略`unique_ptr`的使用，甚至对`exclusive-ownership`的对象我也是一律采用`shared_ptr`的方案，而丝毫没有考虑`unique_ptr`的感受...所以必须总结一番`unique_ptr`的特性。

首先先简单回顾一番智能指针的发展历程...
在C++98之前，标准库中已经有了`std::auto_ptr`这样的智能指针，这可以看作是C++对`RAII`思想的一种尝试，所以`auto_ptr`的成熟度是不合格的。`C++11`带来了三种现代指针：`shared_ptr / unique_ptr / weak_ptr`，其中`shared_ptr`与`weak_ptr`是配合使用的，而`unique_ptr`则完全替代了`auto_ptr`。
所以除非是不得已要维护C++98之前的代码，那么请弃用`auto_ptr`。

好的下面是我们今天的主角——`unique_ptr`。
不仅`unique_ptr`的行为接近原生指针，它的内存占用也可以认为是和原生指针一样的（使用默认的`deleter`），可以说是非常轻量级了，因此大多场景下也应该是我们的优先考虑对象。
一个`unique_ptr`会**独占**它所指向的实例，但更需要注意的是——`unique_ptr`是`moveable but not copyable`的。`uncopyable`很好理解，毕竟和`unique`对应上了，其拷贝构造函数和赋值操作符都通过`=delete`禁用了。但是`moveable`一开始却十分困惑我。然而存在及合理——一方面，我猜测是为了优化性能，其次是可以将源指针的管理权限通过`move`转移到目标指针，**注意是转！移！而不是赋值/复制**，但更重要的，`moveable`的特性为`unique_ptr` **存入容器和作为函数返回值** 提供了可能。

重点说作为函数返回值。
`unique_ptr`作为函数返回值的一个典型应用就是`工厂模式`，实现如下：
```
template<class T,     typename... Types>
unique_ptr<T> getInstance(Types&&... Args)
{
    return (unique_ptr<T>(new T(std::forward<Types>(Args)...)));   
}

/* 既然用了C++14那么也可以更彻底一点
template<class T,     typename... Types>
auto getInstance(Types&&... Args)
{
    return (unique_ptr<T>(new T(std::forward<Types>(Args)...)));   
}
*/

//通过如下调用使用
auto new_instance=getInstance(params);
```
经过测试，在`return`语句中构造`unique_ptr`对象确实不会调用拷贝构造函数，查阅相关资料后发现：
> 当函数返回一个对象时，理论上会产生临时变量，那必然是会导致新对象的构造和旧对象的析构，这对效率是有影响的。C++编译针对这种情况允许进行优化，哪怕是构造函数有副作用，这叫做返回值优化（RVO),返回有名字的对象叫做具名返回值优化(NRVO).

酷的不行...

`unique_ptr`和`shared_ptr`一样，有一个默认的删除器（直接使用`delete`删除对应指针），但是用户也可以根据所管理的对象构成设计自己的删除器，就像下面这样：
```
// C++11/14风格 + lambda表达式
auto delInstance = [](SomeClass* pInstance)
{
makeLogEntry(pInstance);
// sth else.
delete pInstance;
};

//在构造unique_ptr时传入删除器
std::unique_ptr<SomeClass,decltype(delInstance)>
 pIns(nullptr,delInstance);

```
注：指定模板类型时使用`decltype`推导`delInstance`的类型，在构造实例时传入具体的`delInstance`方法作为删除器，书中更建议使用`lambda`来设计删除器，积极拥抱新标准。
（此时`unique_ptr`的大小可就和原生指针不一样了，具体大小取决于删除器这个`function object`是如何工作的，不合理的删除器设计往往会导致`unique_ptr`的大小膨胀）

`unique_ptr`是有两种形式的——单独的实体（`std::unique_ptr<T>`）和数组（`std::unique_ptr<T[]>`），具体如何使用取决于使用者，但要注意前者没有重载`[]`，而后者没有重载`*` / `->`。

总之，对于`exclusive-ownership`的对象，应该尽量使用`unique_ptr`，这才是其实际意义所在，并且`unique_ptr`与`shared_ptr`之前的转换也十分方便，甚至可以用直接前面的`getInstance`函数构造一个`shared_ptr`对象。
