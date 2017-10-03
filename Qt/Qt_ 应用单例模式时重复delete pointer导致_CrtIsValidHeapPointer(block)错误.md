> 前景：在写Qt桌面应用时，有一个场景是TabWidget管理四个子窗口，每个子窗口都只能有一个实例方便窗口的切换，自然而然就想到要用`singleton-pattern`，我应用时能正常启动，但是关闭main window的时候每次都弹出_CrtIsValidHeapPointer(block) ...

既然是关闭窗口的时候弹出的错误，那么肯定是`main window`析构的时候发生了错误，根据错误信息也能知道是堆内存中出错了。
当时我的单例模式是这么写的（参考了scott meyers的版本）：
```cpp
class ConfParas : public QWidget
{
    Q_OBJECT

public:
    static ConfParas* getInstance()
    {
        static ConfParas theConfParas;
        return &theConfParas;
    }
    explicit ConfParas(QWidget *parent = 0);
    ConfParas(const ConfParas&)=delete;
    ConfParas& operator=(const ConfParas&)=delete;
    //sth else...
}
```
自以为没毛病... `static`修饰`theConfParas`，只会在第一次调用`getInstance`方法的时候生成，然后每次返回地址就好了，不需要纯指针版本的`if(!theConfParas)`判断语句。
但是任凭我怎么修改代码都不能消除这个error，后来结合文档才找到了原因，这和`Qt`本身的框架设计有关。

**当有一个指针指向`QObject`对象时，`Qt`就会接管这个指针，负责它的生命周期。`Qt`会默认此指针是动态申请的，所以内存被分配在堆上，并且在合适的时候调用`delete`删除此指针。**

在我这个版本的`getInstance()`中，`theConfParas`由于是静态对象，所以是分配在栈上的，在运行时（run time）被管理。而当我将此地址通过`return &theConfParas;`传递给一个`QObject`对象，即`ConfParas`类时，转移权就交给了`Qt`。

我们都知道在C++中，**`static`对象是整个程序生命周期内最后被析构的**，而我这个`theConfParas`的控制权已经交给了`Qt`，`Qt`会在合适的时候`delete`它，但是主程序并不知道，所以当`Mainwindow`退出时，会对它控制范围内的`QObject`对象做一个内存回收，这之后C++的内存回收机制又会对`static`对象做一次内存回收，这里就出现了`double-delete`，'theConfParas'的地址被删除两次，造成了`_CrtIsValidHeapPointer(block)`错误。

当我用经典的单例模式写法，这样的问题就解决了...
所以年轻人啊，不要追求什么极简巧妙，`runnable`才是王道呀。
所以程序作如下变动：
```cpp
class ConfParas : public QWidget
{
    Q_OBJECT

public:
    static ConfParas* getInstance()
    {
        if (!theConfParas)
            theConfParas = new ConfParas();
        return theConfParas;
    }
    explicit ConfParas(QWidget *parent = 0);
    ConfParas(const ConfParas&)=delete;
    ConfParas& operator=(const ConfParas&)=delete;
    //sth else...
private:
    static ConfParas *theConfParas; //在.cpp文件中初始化为nullptr.
}
```
如果一开始就将它作为`QObject`的属性，那么它的生命周期一开始就会交由`Qt`管理，就不会发生以上的问题。

其实当`_CrtIsValidHeapPointer(block)`错误发生时，我们就应该非常小心了，在堆内存上发生的错误，一定都不是小错误，`double-delete`可以说是很严重的错误了（默泪），所以一定要好好排错。
在使用其他框架时，也应该优先去查询对应类目的文档，一个成熟的框架其官方文档一定是详尽的。