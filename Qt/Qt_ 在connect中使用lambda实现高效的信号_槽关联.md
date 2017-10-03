在Qt中，使用` QCoreApplication::connect()`函数连接信号`(signal)`与槽`(slots)`的功能。
以下是`connect`函数的三种重载形式：
> static QMetaObject::Connection connect(const QObject *sender, const char *signal, const QObject *receiver, const char *member, Qt::ConnectionType = Qt::AutoConnection);
> 
> static QMetaObject::Connection connect(const QObject *sender, const QMetaMethod &signal,const QObject *receiver, const QMetaMethod &method,Qt::ConnectionType type = Qt::AutoConnection);
>
>inline QMetaObject::Connection connect(const QObject *sender, const char *signal,const char *member, Qt::ConnectionType type = Qt::AutoConnection) const;
                        
可以在函数声明中明显看出各个形参对应的意义，分别是`[信号发送者，信号，接收者，槽]`，比如需要实现一个widget中点击button关闭窗口的功能，在Qt4这等老版本中，`connect`的写法是这样的：
```cpp
connect(button,SIGNAL(QPushButton::cliked()),widget,SLOT(QWidget::close()));
```
每次都要添加`SIGNAL/SLOT`修饰符就显得很麻烦，所以在Qt5中，改进了`connect`的写法，可以直接用**&**代替修饰符
```cpp
connect(button,&QPushButton::cliked,widget,&QWidget::close);
```
这样的写法更简洁但是有一个问题就是无法向`signal/slot`函数传参，只能对无参版本的信号/槽生效，所以在有参数的情况下，只能选用第一种写法。

在`connect`函数中，还有一个重要的限制！**在传参的情况下，slot函数的参数个数一定要小于等于signal函数的参数个数**，这样当`signal`函数是`clicked()`这样的无参形式时，我们不能与带参数的`slot`函数进行关联，这就意味着：如果我们想通过点击一个按钮实现一个较为复杂的功能，而这个功能的实现必须按需将实参传入slot函数，那么用传统的`connect`方法时行不通的。

在Qt5的新版本中，`connect`函数添加了对**lambda**的支持！——可以用lambda替代`signal/slot`函数。

这一feature的引进大大简化了`slot`函数的实现，更自由地扩充其功能，并且可以不受`signal`参数的影响（**因为lambda可以捕获参数**）而且`connect`的形式也进一步精简了。

比如我希望在点击一个button时，实现数据库的更新，页面的跳转，然后页面的关闭，那么只需要这么写：
```cpp
connect(button,&QPushButton::cliked,[this,new_data,new_page]{
  UpdateDatabase(new_data);
  OpenNewPage(new_page);
  close();
});
```

甚至连信号的接受者都不用指定了。

**lambda is powerful！！！**