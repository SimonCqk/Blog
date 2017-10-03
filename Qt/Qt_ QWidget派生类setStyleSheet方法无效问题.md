> 最近在做一个基于Qt的桌面应用，准备总结一下开发过程中遇到的一些问题。

我需要创建一个继承自`QWidget`的类来设计自己的窗口，使用`StyleSheet`无疑能方便快捷地配置窗口的一些风格，但是我在应用继承自基类的`setStyleSheet`的方法时发现，设置的style sheet并不能在最终的页面上生效，查阅了官方文档之后发现只需要在派生类中重写一下`paintEvent`方法。
具体如下：
官方文档中的指引![offical](http://img.blog.csdn.net/20170906225227277?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
于是我们只要在自己的类中：
```cpp
// .h file
class SubClass : public QWidget{
...
protected:
void paintEvent(QPaintEvent *);
...
}

// .cpp file
void SubClass::paintEvent(QPaintEvent *) {
  QStyleOption opt;
  opt.init(this);
  QPainter p(this);
  style()->drawPrimitive(QStyle::PE_Widget, &opt, &p, this);
}

```
这样当我们再次应用`->setStyleSheet`方法时就可以生效了。
Qt是一个相当成熟的GUI框架，甚至可以说不仅仅是一个GUI框架，它有自己的`Web Engine`等等，更有好的是它官方文档极其丰富，我们遇到的绝大多数问题都能在文档中找到答案。
遇到问题，首先想到的应该是自己动手检索解决，而不是做一个伸手党等着别人帮你fix bugs，解决不了的就利用好`stackoverflow`这样的网站，提问时一定要遵循**Minimal, Complete, and Verifiable** [view](https://stackoverflow.com/help/mcve)原则，也是在节省帮助你定位问题的人的时间。