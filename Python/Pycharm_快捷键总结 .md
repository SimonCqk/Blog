	> 之前也零散的看过一些Python，因为这个暑假项目的缘故，这几天开始全面系统的学习Python，IDE当然是选择JB家的Pycharm啦，用的顺手真的爽的一比，快捷键用的溜，真的能极大地提高效率，现在对Pycharm的一些快捷键做一个小总结.

#### JB家的IDE很多快捷键都是通用的，所以借鉴了Intellij的一些快捷键用法 <其实算是一篇转载啦...>

`ctrl+alt+L`，格式化代码，曾经用Eclipse的时候，格式化代码一定要先选中某个区域的代码块再进行格式化，用了JetBrains的IDE之后就习惯性地格式化一下，还必须手贱多格式化几次才舒服，不过有后遗症，以后到各种编辑框中总喜欢`ctrl+alt+L`几下才舒服
`alt+enter`万能键，不同场景有不同的表现方式，在还没import(install)模块名上使用会import(install)相对应的模块，在方法名上使用补注释，在测试中使用帮你补充断言代码……不管出错了不知道怎么办，知道怎么办想这么办时都按下这个快捷键，有惊喜
`ctrl+space` 万能提示键，PyCharm的会根据上下文提供补全。不仅是代码提示，`ctrl+f`搜索的时候，不知道该键入啥时就按下，经常给人惊喜。在windows下由于输入法无法使用这个快捷键，`ctrl+alt+s`打开设置键入 keymap，在Keymap中搜索basic可以找到并修改它
`ctrl+shift+f10`运行当前文件，十分智能，如果是测试代码，则以测试模式run，并列出测试通过情况
`ctrl+w`扩展选取和`ctrl+shift+w`缩减选区，根据当前层级一层一层向上伸展选中区域，例如value ->key:value -> dict -> ... ，每按一次就扩展一级，`ctrl+shift+w`用来取消上一次的扩展选取操作，注意后遗症，大部分软件中的`ctrl+w`是关闭！是`ctrl+shift+箭头`而不是`ctrl+w`
`ctrl+alt+shift+T`重构选区，可以将选区抽取成变量，方法等。不同的重构方式有不同的快捷键，比如将a.b.c中的a.b抽取出来，选中a.b，键入`ctrl+alt+v`可见效果
`ctrl+q`查注释
`ctrl+shift+i`没注释时查源码
`ctrl+p`调用方法时查看参数，在括号内按下，就知道当前位置该给啥参数，不过由于是Python，这里使用`ctrl+space`也可以达到很好的效果
`shift+enter`另起一行，光标在行内任意位置都能另起一行，且不破坏当行结构
`ctrl+alt+enter`向上另起一行
`ctrl+shift+数字键`与`ctrl+数字键`书签功能
`ctrl+shift+a`搜索IDE功能，比如想看看这个文件的历史，就键入history 可以找到 Local history
变量或表达式的最后键入`.(点)`，出现提示后，看最下面的部分有各种代码模板，例如现有变量a，在下一行键入 a.(点)print 然后键入`tab`，Python2会变成`print a`，Python3会变成`print(a)`，还有if,else等各种常用的代码模板
双击`shift`搜索一切，不管是IDE功能、文件、方法、变量……都能搜索

####借鉴自Intellij


* 智能提示Intellij首当其冲的当然就是Intelligence智能！基本的代码提示用Ctrl+Space，还有更智能地按类型信息提示Ctrl+Shift+Space，但因为Intellij总是随着我们敲击而自动提示，所以很多时候都不会手动敲这两个快捷键(除非提示框消失了)。用F2/ Shift+F2移动到有错误的代码，Alt+Enter快速修复(即Eclipse中的Quick Fix功能)。当智能提示为我们自动补全方法名时，我们通常要自己补上行尾的反括号和分号，当括号嵌套很多层时会很麻烦，这时我们只需敲Ctrl+Shift+Enter就能自动补全末尾的字符。而且不只是括号，例如敲完if/for时也可以自动补上{}花括号。
最后要说一点，Intellij能够智能感知spring、hibernate等主流框架的配置文件和类，以静制动，在看似“静态”的外表下，智能地扫描理解你的项目是如何构造和配置的。

* 重构Intellij重构是另一完爆Eclipse的功能，其智能程度令人瞠目结舌，比如提取变量时自动检查到所有匹配同时提取成一个变量等。尤其看过《重构-改善既有代码设计》之后，有了Intellij的配合简直是令人大呼过瘾！也正是强大的智能和重构功能，使Intellij下的TDD开发非常顺畅。
切入正题，先说一个无敌的重构功能大汇总快捷键Ctrl+Shift+Alt+T，叫做Refactor This。按法有点复杂，但也符合Intellij的风格，很多快捷键都要双手完成，而不像Eclipse不少最有用的快捷键可以潇洒地单手完成(不知道算不算Eclipse的一大优点)，但各位用过Emacs的话就会觉得也没什么了(非Emacs黑)。此外，还有些最常用的重构技巧，因为太常用了，若每次都在Refactor This菜单里选的话效率有些低。比如Shift+F6直接就是改名，Ctrl+Alt+V则是提取变量。

* 代码生成这一点类似Eclipse，虽不是独到之处，但因为日常使用频率极高，所以还是罗列在榜单前面。常用的有fori/sout/psvm+Tab即可生成循环、System.out、main方法等boilerplate样板代码，用Ctrl+J可以查看所有模板。后面“辅助”一节中将会讲到Alt+Insert，在编辑窗口中点击可以生成构造函数、toString、getter/setter、重写父类方法等。这两个技巧实在太常用了，几乎每天都要生成一堆main、System.out和getter/setter。
另外，Intellij IDEA 13中加入了后缀自动补全功能(Postfix Completion)，比模板生成更加灵活和强大。例如要输入for(User user : users)只需输入user.for+Tab。再比如，要输入Date birthday = user.getBirthday();只需输入user.getBirthday().var+Tab即可。

* 编辑编辑中不得不说的一大神键就是能够自动按语法选中代码的Ctrl+W以及反向的Ctrl+Shift+W了。此外，Ctrl+Left/Right移动光标到前/后单词，Ctrl+[/]移动到前/后代码块，这些类Vim风格的光标移动也是一大亮点。以上Ctrl+Left/Right/[]加上Shift的话就能选中跳跃范围内的代码。Alt+Forward/Backward移动到前/后方法。还有些非常普通的像Ctrl+Y删除行、Ctrl+D复制行、`Ctrl+ </>` 折叠代码就不多说了。
关于光标移动再多扩展一点，除了Intellij本身已提供的功能外，我们还可以安装ideaVim或者emacsIDEAs享受到Vim的快速移动和Emacs的AceJump功能(超爽！)。另外，Intellij的书签功能也是不错的，用Ctrl+Shift+Num定义1-10书签(再次按这组快捷键则是删除书签)，然后通过Ctrl+Num跳转。这避免了多次使用前/下一编辑位置Ctrl+Left/Right来回跳转的麻烦，而且此快捷键默认与Windows热键冲突(默认多了Alt，与Windows改变显示器显示方向冲突，一不小心显示器就变成倒着显式的了，冏啊)。

* 查找打开类似Eclipse，Intellij的Ctrl+N/Ctrl+Shift+N可以打开类或资源，但Intellij更加智能一些，我们输入的任何字符都将看作模糊匹配，省却了Eclipse中还有输入*的麻烦。最新版本的IDEA还加入了Search Everywhere功能，只需按Shift+Shift即可在一个弹出框中搜索任何东西，包括类、资源、配置项、方法等等。
类的继承关系则可用Ctrl+H打开类层次窗口，在继承层次上跳转则用Ctrl+B/Ctrl+Alt+B分别对应父类或父方法定义和子类或子方法实现，查看当前类的所有方法用Ctrl+F12。
要找类或方法的使用也很简单，Alt+F7。要查找文本的出现位置就用Ctrl+F/Ctrl+Shift+F在当前窗口或全工程中查找，再配合F3/Shift+F3前后移动到下一匹配处。
Intellij更加智能的又一佐证是在任意菜单或显示窗口，都可以直接输入你要找的单词，Intellij就会自动为你过滤。

* 其他辅助以上这些神键配上一些辅助快捷键，即可让你的双手90%以上的时间摆脱鼠标，专注于键盘仿佛在进行钢琴表演。这些不起眼却是至关重要的最后一块拼图有：
	-  命令：Ctrl+Shift+A可以查找所有Intellij的命令，并且每个命令后面还有其快捷键。所以它不仅是一大神键，也是查找学习快捷键的工具。
    - 新建：Alt+Insert可以新建类、方法等任何东西。
    - 格式化代码：格式化import列表Ctrl+Alt+O，格式化代码Ctrl+Alt+L。
    - 切换窗口：Alt+Num，常用的有1-项目结构，3-搜索结果，4/5-运行调试。Ctrl+Tab切换标签页，Ctrl+E/Ctrl+Shift+E打开最近打开过的或编辑过的文件。
    - 单元测试：Ctrl+Alt+T创建单元测试用例。
    - 运行：Alt+Shift+F10运行程序，Shift+F9启动调试，Ctrl+F2停止。
    - 调试：F7/F8/F9分别对应Step into，Step over，Continue。
此外还有些我自定义的，例如水平分屏Ctrl+|等，和一些神奇的小功能Ctrl+Shift+V粘贴很早以前拷贝过的，Alt+Shift+Insert进入到列模式进行按列选中。

* 最终榜单这榜单阵容太豪华了，后几名都是如此有用，毫不示弱。
	- Top #10切来切去：Ctrl+Tab
	- Top #9选你所想：Ctrl+W
	- Top #8代码生成：Template/Postfix +Tab
	- Top #7发号施令：Ctrl+Shift+A
    -  Top #6无处藏身：Shift+Shift
	-  Top #5自动完成：Ctrl+Shift+Enter
	- Top #4创造万物：Alt+Insert
	太难割舍，前三名并列吧！
	-  Top #1智能补全：Ctrl+Shift+Space
	-  Top #1自我修复：Alt+Enter
	-  Top #1重构一切：Ctrl+Shift+Alt+T
