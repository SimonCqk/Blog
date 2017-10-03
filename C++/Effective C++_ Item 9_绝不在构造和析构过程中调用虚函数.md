> 这个应该是C++面试的经典题了，所以值得拿出来说一说

比如在一个继承体系中，基类的构造函数中调用了一个基类的成员函数，你把它声明为`virtual`，至少你在设计的时候是认为它有`virtual`属性的。
```cpp
class Base{
public:
	Base();
	virtual void someFunction()=0;
	...
	}
Base::Base(){
	...
	someFunction();
	}
class DerivedOne:public Base{
public:
	DerivedOne();
	virtual void someFunction() override;
	...
}
class DerivedTwo: public Base{
public:
	DerivedTwo();
	virtual void someFunction() override;
	...
}

```
接下来实例化对象，当我们执行
```cpp
DerivedOne b;
```
因为`DerivedOne`是一个派生类，那么会先调用基类的构造函数`Base()`，构造派生类中的基类子部分，但是这是`Base()`函数调用的不是我们期望的`DerivedOne`中被`override`的函数，这就是问题所在，**在基类构造期间，virtual函数不会下降到derived阶层，这时virtual函数与普通类内成员函数等价**，其实也很好理解，在构造基类期间，派生类根本没有被构造出来，何来`override`之说？一直在`DerivedOne`对象被构造出来之前，`b`的类型都是`Base`，可以通过`typeid`和`dynamic_cast`验证。
析构函数也是同样的道理，析构的顺序是`自下而上`的，所以当`derived class`被析构后，它的对象就不存在了，等到`base class`运行阶段，`virtual`函数也就是基类本身的那个函数了，`virtual`属性失去了意义。
如果希望创建一个基类，运行时动态绑定适当的`override`函数，那么一定不要在构造/析构函数中调用`virtual`，可以有其他的解决方案。
我们可以对期望函数通过参数传递信息，然后再`derived class`构造函数的初始化列表中将信息传递进去。
```cpp
class Base{
public:
	Base();
	void someFunction(const string& s)=0;  //non-virtual
	...
	}
Base::Base(const string& s){
	...
	someFunction(s);
	}
class DerivedOne:public Base{
public:
	DerivedOne():Base(createInfo());  //形如这样
	...
private:
	string createInfo(...);
}


```
