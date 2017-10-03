> 异常安全`（Exception safety）`有点像怀孕`（pregnancy）`……但是，请把这个想法先控制一会儿。我们还不能真正地议论生育`（reproduction）`，直到我们排除万难渡过求爱时期
`（courtship）`。（此段作者使用的 3 个词均有双关含义，pregnancy 也可理解为富有意义，`reproduction` 也可理解为再现，再生,`courtship` 也可理解为争取，谋求。为了与后面的译文对应，故按照现在的译法。——译者注）

假设我们有一个类，代表带有背景图像的 GUI 菜单。这个类被设计成在多线程环境中使用，
所以它有一个用于并行控制（concurrency control）的互斥体（mutex）：
```cpp
class PrettyMenu {
public:
...
void changeBackground(std::istream& imgSrc); // change background
... // image
private:
Mutex mutex; // mutex for this object
Image *bgImage; // current background image
int imageChanges; // # of times image has been changed
};
```
考虑这个 PrettyMenu 的 changeBackground 函数的可能的实现：
```cpp
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
lock(&mutex); // acquire mutex (as in Item 14)
delete bgImage; // get rid of old background
++imageChanges; // update image change count
bgImage = new Image(imgSrc); // install new background
unlock(&mutex); // release mutex
}
```
从异常安全的观点看，这个函数烂到了极点。异常安全有两条要求，而这里全都没有满足。
当一个异常被抛出，异常安全的函数应该：
没有资源泄露。上面的代码没有通过这个测试，因为如果 `"new Image(imgSrc)"` 表达式产生一个异常，对 `unlock` 的调用就永远不会执行，而那个互斥体也将被永远挂起。
不允许数据结构恶化。如果 "new Image(imgSrc)" 抛出异常，bgImage 被遗留下来指向一个被删除对象。另外，尽管并没有将一张新的图像设置到位，imageChanges 也已经
被增加。（在另一方面，旧的图像被明确地删除，所以我料想你会争辩说图像已经被“改变”了。）
规避资源泄露问题比较容易，因为**Item 13** 解释了如何使用对象管理资源，而 **Item 14** 又引进
了 Lock 类作为一种时尚的确保互斥体被释放的方法：
```cpp
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
Lock ml(&mutex); // from Item 14: acquire mutex and
// ensure its later release
delete bgImage;
++imageChanges;
bgImage = new Image(imgSrc);
}
```
关于像 Lock 这样的资源管理类的最好的事情之一是它们通常会使函数变短。看到对 unlock的调用不再需要了吗？作为一个一般的规则，更少的代码就是更好的代码。因为在改变的时候这样可以较少误入歧途并较少产生误解。
随着资源泄露被我们甩在身后，我们可以把我们的注意力集中到数据结构恶化。在这里我们有一个选择，但是在我们能选择之前，我们必须先面对定义我们的选择的术语。
异常安全函数提供下述三种保证之一：

- 函数提供基本保证（the basic guarantee），允诺如果一个异常被抛出，程序中剩下的每一件东西都处于合法状态。没有对象或数据结构被破坏，而且所有的对象都处于内部调和状态（所有的类不变量都被满足）。然而，程序的精确状态可能是不可预期的。例如，我们可以重写 `changeBackground`，以致于如果一个异常被抛出，`PrettyMenu` 对象可以继续保留原来的背景图像，或者它可以持有某些缺省的背景图像，但是客户无法预知到底是哪一个。（为了查明这一点，他们大概必须调用某个可以告诉他们当前背景图
像是什么的成员函数。）
- 函数提供强力保证（the strong guarantee），允诺如果一个异常被抛出，程序的状态不会发生变化。调用这样的函数在感觉上是极其微弱的，如果它们成功了，它们就完全成功，如果它们失败了，程序的状态就像它们从没有被调用过一样。
与提供强力保证的函数一起工作比与只提供基本保证的函数一起工作更加容易，因为调用提供强力保证的函数之后，仅有两种可能的程序状态：像预期一样成功执行了函数，或者继续保持函数被调用时当时的状态。与之相比，如果调用只提供基本保证的函数引发了异常，程序可能存在于任何合法的状态。
- 函数提供不抛出保证（the nothrow guarantee），允诺决不抛出异常，因为它们只做它们答应要做的。所有对内建类型（例如，ints，指针，等等）的操作都是不抛出（nothrow）的（也就是说，提供不抛出保证）。这是异常安全代码中必不可少的基础构
件。
假定一个带有空的异常规格（exception specification）的函数是不抛出的似乎是合理的，但这不一定正确的。例如，考虑这个函数：
```cpp
int doSomething() throw(); // note empty exception spec.
```
这并不是说 doSomething 永远不会抛出异常；而是说如果 doSomething 抛出一个异常，
它就是一个严重的错误，应该调用 unexpected 函数 [1]。实际上，doSomething 可能根本不提供任何异常保证。一个函数的声明（如果有的话，也包括它的异常规格（exception specification））不能告诉你一个函数是否正确，是否可移植，或是否高效，而且，即便有，它也不能告诉你它会提供哪一种异常安全保证。所有这些特性都由函数的实现决定，而不是它的声明能决定的。
[1] 关于 unexpected 函数的资料，可以求助于你中意的搜索引擎或包罗万象的 C++ 课本。（你或许有幸搜到 set_unexpected，这个函数用于指定 unexpected 函数。）

**异常安全函数必须提供上述三种保证中的一种。**如果它没有提供，它就不是异常安全的。于是，选择就在于决定你写的每一个函数究竟要提供哪种保证。除非要处理遗留下来的非异常安全的代码（本 Item 稍后我们要讨论这个问题），只有当你的最高明的需求分析团队为你的应用程序识别出的一项需求就是泄漏资源以及运行于被破坏的数据结构之上时，不提供异常安全保证才能成为一个选项。
作为一个一般性的规则，你应该提供实际可达到的最强力的保证。从异常安全的观点看，不抛出的函数`（nothrow functions）`是极好的，但是在 C++ 的 C 部分之外部不调用可能抛出异常的函数简直就是寸步难行。使用动态分配内存的任何东西（例如，所有的 STL 容器）如果不能找到足够的内存来满足一个请求（参见 **Item 49**），在典型情况下，它就会抛出一个
bad_alloc 异常。只要你能做到就提供不抛出保证，但是对于大多数函数，选择是在基本的保证和强力的保证之间的。
在 changeBackground 的情况下，提供差不多的强力保证并不困难。首先，我们将PrettyMenu 的 bgImage 数据成员的类型从一个内建的 Image* 指针改变为 **Item 13** 中描述的智能资源管理指针中的一种。坦白地讲，在预防资源泄漏的基本原则上，这完全是一个好主
意。它帮助我们提供强大的异常安全保证的事实进一步加强了 **Item 13** 的论点——使用对象（诸如智能指针）管理资源是良好设计的基础。在下面的代码中，我展示了` tr1::shared_ptr`
的使用，因为当进行通常的拷贝时它的更符合直觉的行为使得它比 `auto_ptr `更可取。
第二，我们重新排列 changeBackground 中的语句，以致于直到图像发生变化，才增加imageChanges。这是一个很好的策略——直到某件事情真正发生了，再改变一个对象的状态
来表示某事已经发生。
这就是修改之后的代码：
```cpp
class PrettyMenu {
...
std::tr1::shared_ptr<Image> bgImage;
...
};
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
Lock ml(&mutex);
bgImage.reset(new Image(imgSrc)); // replace bgImage's internal
// pointer with the result of the
// "new Image" expression
++imageChanges;
}
```
注意这里不再需要手动删除旧的图像，因为在智能指针内部已经被处理了。此外，只有当新
的图像被成功创建了删除行为才会发生。更准确地说，只有当tr1::shared_ptr::reset 函数的参数`（"new Image(imgSrc)" 的结果）`被成功创建了，这个函数才会被调用。只有在对 reset 的
调用的内部才会使用 delete，所以如果这个函数从来不曾进入，delete 就从来不曾使用。同样请注意一个管理资源（动态分配的 Image）的对象（tr1::shared_ptr）的使用再次缩短了changeBackground 的长度。
正如我所说的，这两处改动差不多有能力使 changeBackground 提供强力异常安全保证。美中不足的是什么呢？参数 imgSrc。如果 Image 的构造函数抛出一个异常，输入流（inputstream）的读标记（readmarker）可能已经被移动，而这样的移动就成为对程序的其它部分来说可见的一个状态的变化。直到 changeBackground 着手解决这个问题之前，它只能提供基本异常安全保证。
无论如何，让我们把它放在一边，并且依然假装 changeBackground 可以提供强力保证。
（我相信你至少能用一种方法做到这一点，或许可以通过将它的参数从一个 istream 改变到包含图像数据的文件的文件名。）有一种通常的设计策略可以有代表性地产生强力保证，而且熟悉它是非常必要的。这个策略被称为 **"copy and swap"**。它的原理很简单。先做出一个你要
改变的对象的拷贝，然后在这个拷贝上做出全部所需的改变。如果改变过程中的某些操作抛出了异常，最初的对象保持不变。在所有的改变完全成功之后，将被改变的对象和最初的对象在一个不会抛出异常的操作中进行交换。
这通常通过下面的方法实现：将每一个对象中的全部数据从“真正的”对象中放入到一个单独的
实现对象中，然后将一个指向实现对象的指针交给真正对象。这通常被称为 **"pimpl idiom"**，
**Item 31** 描述了它的一些细节。对于 PrettyMenu 来说，它一般就像这样：
```cpp
struct PMImpl { // PMImpl = "PrettyMenu
std::tr1::shared_ptr<Image> bgImage; // Impl."; see below for
int imageChanges; // why it's a struct
};
class PrettyMenu {
...
private:
Mutex mutex;
std::tr1::shared_ptr<PMImpl> pImpl;
};
void PrettyMenu::changeBackground(std::istream& imgSrc)
{
using std::swap; // see Item 25
Lock ml(&mutex); // acquire the mutex
std::tr1::shared_ptr<PMImpl> // copy obj. data
pNew(new PMImpl(*pImpl));
pNew->bgImage.reset(new Image(imgSrc)); // modify the copy
++pNew->imageChanges;
swap(pImpl, pNew); // swap the new
// data into place
} // release the mutex
```
在这个例子中，我选择将 PMImpl 做成一个结构体，而不是类，因为通过让 pImpl 是 private就可以确保 PrettyMenu 数据的封装。将 PMImpl 做成一个类虽然有些不那么方便，却没有增加什么好处。（这也会使有面向对象洁癖者走投无路。）如果你愿意，PMImpl 可以嵌套在
PrettyMenu 内部，像这样的打包问题与我们这里所关心的写异常安全的代码的问题没有什么关系。
**copy-and-swap** 策略是一种全面改变或丝毫不变一个对象的状态的极好的方法，但是，在通
常情况下，它不能保证全部函数都是强力异常安全的。为了弄清原因，考虑一个
changeBackground 的抽象化身—— someFunc，它使用了 copy-and-swap，但是它包含了对
另外两个函数（f1 和 f2）的调用：
```cpp
void someFunc()
{
... // make copy of local state
f1();
f2();
... // swap modified state into place
}
```
很明显，如果 f1 或 f2 低于强力异常安全，someFunc 就很难成为强力异常安全的。例如，假
设 f1 仅提供基本保证。为了让 someFunc 提供强力保证，它必须写代码在调用 f1 之前测定整
个程序的状态，并捕捉来自 f1 的所有异常，然后恢复到最初的状态。
即使 f1 和 f2 都是强力异常安全的，事情也好不到哪去。如果 f1 运行完成，程序的状态已经发生了毫无疑问的变化，所以如果随后 f2 抛出一个异常，即使 f2 没有改变任何东西，程序的状态也已经和调用 someFunc 时不同。
问题在于副作用。只要函数仅对局部状态起作用（例如，someFunc 仅仅影响调用它的那个对象的状态），它提供强力保证就相对容易。当函数的副作用影响了非局部数据，它就会困难得多。例如，如果调用 f1 的副作用是改变数据库，让 someFunc 成为强力异常安全就非常困难。一般情况下，没有办法撤销已经提交的数据库变化，其他数据库客户可能已经看见了数据库的新状态。
类似这样的问题会阻止你为函数提供强力保证，即使你希望去做。另一个问题是效率。**copyand-swap** 的要点是这样一个想法：改变一个对象的数据的拷贝，然后在一个不会抛出异常的操作中将被改变的数据和原始数据进行交换。这就需要做出每一个要改变的对象的拷贝，这可能会用到你不能或不情愿动用的时间和空间。强力保证是非常值得的，当它可用时你应该提供它，除非在它不能 100% 可用的时候。
当它不可用时，你就必须提供基本保证。在实践中，你可能会发现你能为某些函数提供强力保证，但是效率和复杂度的成本使得它难以支持大量的其它函数。无论何时，只要你作出过一个提供强力保证的合理的成果，就没有人会因为你仅仅提供了基本保证而站在批评你的立场上。对于很多函数来说，基本保证是一个完全合理的选择。
如果你写了一个根本没有提供异常安全保证的函数，事情就不同了，因为在这一点上有罪推定是合情合理的，直到你证明自己是清白的。你应该写出异常安全的代码。除非你能做出有说服力的答辩。请再次考虑 someFunc 的实现，它调用了函数 f1 和 f2。假设 f2 根本没有提供异常安全保证，甚至没有基本保证。这就意味着如果 f2 发生一个异常，程序可能会在 f2 内部泄漏资源。这也意味着 f2 可能会恶化数据结构，例如，已排序数组可能不再排序，一个正在从一个数据结构传送到另一个数据结构去的对象可能丢失，等等。没有任何办法可以让someFunc 能弥补这些问题。如果 someFunc 调用的函数不提供异常安全保证，someFunc本身就不能提供任何保证。
请允许我回到怀孕。一个女性或者怀孕或者没有。局部怀孕是绝不可能的。与此相似，一个软件或者是异常安全的或者不是。没有像一个局部异常安全的系统这样的东西。一个系统即使只有一个函数不是异常安全的，那么系统作为一个整体就不是异常安全的，因为调用那个函数可能发生泄漏资源和恶化数据结构。不幸的是，很多 C++ 的遗留代码在写的时候没有留意异常安全，所以现在的很多系统都不是异常安全的。它们混合了用非异常安全（exceptionunsafe）的方式书写的代码。
没有理由让事情的这种状态永远持续下去。当书写新的代码或改变现存代码时，要仔细考虑如何使它异常安全。以使用对象管理资源开始。（还是参见 Item 13。）这样可以防止资源泄漏。接下来，决定三种异常安全保证中的哪一种是你实际上能够为你写的每一个函数提供的最强的保证，只有当你不调用遗留代码就别无选择的时候，才能满足于没有保证。既是为你的函数的客户也是为了将来的维护人员，文档化你的决定。一个函数的异常安全保证是它的接口的可见部分，所以你应该特意选择它，就像你特意选择一个函数接口的其它方面。
四十年前，到处都是 goto 的代码被尊为最佳实践。现在我们为书写结构化控制流程而奋斗。二十年前，全局可访问数据被尊为最佳实践。现在我们为封装数据而奋斗，十年以前，写函
数时不必考虑异常的影响被尊为最佳实践。现在我们为写异常安全的代码而奋斗。
时光在流逝。我们生活着。我们学习着。

**Things to Remember**

- 即使当异常被抛出时，异常安全的函数不会泄露资源，也不允许数据结构被恶化。这样的函数提供基本的，强力的，或者不抛出保证。
- 强力保证经常可以通过 copy-and-swap 被实现，但是强力保证并非对所有函数都可用。
- 一个函数通常能提供的保证不会强于他所调用的函数中最弱的保证。
