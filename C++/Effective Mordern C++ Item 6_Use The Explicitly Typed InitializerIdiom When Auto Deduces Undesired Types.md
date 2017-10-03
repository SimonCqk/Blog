> 啊...别人总结的精简干练，那么...再怒转一篇？？？
> [原博](http://blog.csdn.net/zhangyifei216/article/details/52685125)

在Item5中提到了使用`auto`所带来的诸多优点，在Item2中提到了auto的类型推导规则和模板类型推导基本一致，推导出来的类型有的时候并不是我们所想要的类型(会忽略CV限制符和引用)，那么本文继续探究auto的其它缺点。
```cpp
std::vector<bool> features();
auto ret = features();
```
上面的ret是bool类型吗? 表面看起来是没什么问题的，其实不然，`vector<bool>`的`operator[]`的返回值其实并不是bool类型，`vector<bool>`比较特殊，它返回的是`vector<bool>::reference`，返回一个`bool`引用类型不就完了嘛，标准库干嘛非要这么大费周章的搞了个这样的类型呢？，原因有以下几个:

* 因为bool占用一个字节，标准库为了节省内存，改用bit来表示
* 因为operator[]需要返回一个内部元素的引用，但是没办法对一个bit进行引用
* 为了让返回的类型统一，无论是bool类型，还是其它类型

为此标准库为了实现上述三个目标就封装了一个内部的类型`vector<bool>::reference`，因此auto在这里老老实实得到了一个`vector<bool>::reference`类型，而如果使用下面的代码:
```cpp
bool ret = features();
```
features返回的`vector<bool>::reference`类型会隐式转换为bool类型。

在这个场景下auto弄巧成拙，幸好我们可以通过使用static_cast强制进行类型转换得到我们想要的类型。（static_cast可以用来去引用）
```
auto ret = static_cast<bool>(features);
```
