> 最近参加的一个Program，主题是生物识别，其中的PCA/LDA特征值提取部分需要大量用到线性代数矩阵论的知识，但是稍不注意numpy中的乘法规则就很容易得到错误的结果，最终导致后续结果的崩盘，尤其是较大规模的矩阵，更是很难发现错误。

Numpy中的矩阵乘法分为两大情况，`使用numpy.array`和使用`numpy.matrix`. Numpy确实重载了`*`操作符，可以直接对`array`或者`matrix`对象进行乘法运算，但是在不同对象上，其意义是有区别的。
### 对于array对象
`*（或者multiply）`代表的是**并不是矩阵的乘法规则，而是简单的数量积**，即对应位置元素相乘后的积相加。
```python
import numpy as np
a=np.array([[1,2],[3,4]])
b=np.array([[4,3],[2,1]])
```
运行结果如下：
![array*](http://img.blog.csdn.net/20170727222415285?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
如果在`array`对象上要进行严格的矩阵乘法，即矢量乘法，则必须使用**.dot()或者.matmul()**函数，两者是等效的，我们可以通过查阅官网文档得知。
[numpy.dot](https://docs.scipy.org/doc/numpy/reference/generated/numpy.dot.html)
![dot()](http://img.blog.csdn.net/20170727222829486?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
[numpy.matmul](https://docs.scipy.org/doc/numpy/reference/generated/numpy.matmul.html)
![matmul()](http://img.blog.csdn.net/20170727222852481?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
现在可以在IDLE上验证结果：
![dot/matmul](http://img.blog.csdn.net/20170727223110271?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
可以看到两者的运行结果一致，都是矩阵的矢量积结果。
### 对于matrix对象
对于`matrix`，情况就悄悄相反了。
 `*` **表示的是矢量积，如果希望以数量积方式运行，则必须使用`np.multiply`函数**。因为`*`重载矩阵运算规则只限于`matrix`对象。
在IDLE中重新验证：
![matrix](http://img.blog.csdn.net/20170727224237035?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
结果已经非常清晰了。

### 总结一下：

* 对于`array`对象，`*`和`np.multiply`函数代表的是数量积，如果希望使用矩阵的乘法规则，则应该调用`np.dot`和`np.matmul`函数。
* 对于`matrix`对象，`*`直接代表了原生的矩阵乘法，而如果特殊情况下需要使用数量积，则应该使用`np.multiply`函数。