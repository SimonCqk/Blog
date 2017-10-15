> 智能指针大法好，但是坑儿免不了，不要哭来不要闹，C艹面前谁敢diao

利用RAII来管理资源自然是高效，但是对于一些比较特殊的类型，还是会有不得不注意的地方，没错说的就是`std::shared_ptr`和`this`指针的搭配。
`shared_ptr`的内部实现包括一个指向目标对象的`raw pointer`和一个包含`reference count`等其他维护sp状态的数据结构，称之为控制块`（control block）`，形如下图：

![control block](http://img.blog.csdn.net/20171015102856657?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

一个被`shared_ptr`包裹的对象原则上只能有一块`control block`，这样子对象的生命周期才能被正确管理，`control block`的建立遵循以下三条原则：

- 当`shared_ptr`是从一个`raw pointer`构造而来时
- 当`shared_ptr`从一个`unique_ptr`构造而来时
- 当`shared_ptr`通过`make_shared`函数构造而来时

以上可以看出，连续的从同一个`raw pointer`构造智能指针是一种非常不好的编程习惯：
```cpp
auto pt=new Object;
std::shared_ptr spt(pt);
std::shared_ptr spt2(pt);
```
以上，应该绝对避免，或者就像下面这样写：
```cpp
auto pt=new Object;
std::shared_ptr spt(pt);
std::shared_ptr spt2{spt};
```
接下来进入正题！

当在**类内**想通过对`this`指针进行包裹来达到自动管理资源的目的时，很容易写出这样的代码：
```cpp
class Object;
std::vector<std::shared_ptr<Object>> spts_obj;
//...
class Object{
public:
	void process();
}
//...
void Object::process(){
   spts_obj.emplace_back(this);
}
```
想法没毛病，只是想通过多个`shared_ptr`管理同一个对象罢了，但是！！ 当通过`this`指针构造`shared_ptr`时，是会重新构造一个`control block`时的，不同的`control block`管理着同一个对象，指甲盖想想都知道大错特错了。
这时候就该`std::enable_shared_from_this()`出场了。[cppreference](http://en.cppreference.com/w/cpp/memory/enable_shared_from_this)
`std::enable_shared_from_this()`是一个模板类，使得多个通过`this`构造的智能指针共享同一块`control block`成为可能。
它有两个重要的成员函数：

- shared_from_this：返回一个共享同一个`this`指针的`shared_ptr`对象
- weak_from_this：返回一个共享同一个`this`指针的`weak_ptr`对象

以及一个成员对象：

 - weak_this：一个追踪`control block`状态的`std::weak_ptr`对象

将之前的代码改写一下，就可以做到多个`shared_ptr`安全管理同一个`this`指针了。
```cpp
class Object;
std::vector<std::shared_ptr<Object>> spts_obj;
//...
class Object: public std::enable_shared_from_this<Object>{
public:
	void process();
}
//...
void Object::process(){
   spts_obj.emplace_back(shared_from_this());
}
```
**注意一定是`public`继承!!!**
虽然这样的继承写法看起来有点鬼畜，但是成功达到了目的不是么。
