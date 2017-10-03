> 引言：觉得这一Item挺重要的所以特地拿出来说一说。之前在知乎看到过陈硕大大的某回答中有这么一句话让我印象深刻，大致是“想知道一个项目靠不靠谱（编码者基础扎不扎实），只要点开一个文件看看自己设计的类，有没有禁用或者正确实现`constructor，copy assignment，copy constructor`”，可见这一细节在具体工程实践中的重要性。

   如果你设计的类，每一个实例都有其独立性，比如书中提到的"地产商出售的每一栋房子"都是独一无二的资产，或者模拟光子运行轨迹的时候，每个光子粒其运行轨迹，反射角度都是随机数生成，也是独一无二的，那么就应该禁用其`复制构造函数和赋值运算符`，否则就会打破这一设计原则。
	不同于其他的类成员函数，`复制构造函数和赋值运算符`是特殊的函数，没有显示声明的时候，编译器都会自动生成一份默认的版本，但很多时候默认生成的版本往往和我们得期望会有些出入，所以要么正确的实现他们，要么禁用他们。
	编译器自动生成的版本都是`public`的，在C++11以前，我们可以通过将`复制构造函数和赋值运算符`声明为`private`来阻止这样的生成，这样当我们尝试复制或赋值时编译器就会报错。
		C++11加入了`delete`关键字，可以更直观的禁止这样的操作。
```cpp
class Photon{
public:
Photon(const Photon&)  = delete;
Photon& operator=(const Photon&) = delete;
```
像这样就禁止了编译器自动生成复制构造函数和赋值运算符。

当然，你也可以定义一个`uncopyable`的基类，然后让不可复制的类继承自它就可以了，这也是另一种有效的途径，`boost`中的`noncopyable`就是这么实现的。
```cpp
class UnCopyable{
public:
UnCopyable(const UnCopyable&)  = delete;
UnCopyable& operator=(const UnCopyable&) = delete;
}

class Photon:public UnCopyable{
...
}
```