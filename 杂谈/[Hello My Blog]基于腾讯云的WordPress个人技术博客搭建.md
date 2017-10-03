下午花了两个多小时的时间搭建了自己的第一个博客，本来想直接转载一份别人的教程作为第一篇文章试试水，但是自己的在搭建的过程中也遇到了一些问题，所以决定自己写一篇做一个简单的分享顺便避免小白白再踩坑吧。博客基于腾讯云+64位CentOS 7.2+PHP7+MySQL+WordPress，是一套纯开源的博客系统。

技术博客这种东西，于个人，可以梳理自己的知识点，记录自己的学习曲线，巩固自己的理论基础，不断回顾也便于查阅，锻炼加强自己表达的能力，积攒到一定的质与量后甚至可以写进简历作为亮点。于他人也是一种资源，百利而无一害的事情，将其带入工作中更是一件益事，对未来的成长与进阶来说都起着垫脚石的作用。

接下来进入正题。
<h1>前期准备：</h1>
<ul>
 	<li>  首先要选定一个大厂的云服务器，腾讯/阿里等都面向学生开放了非常优惠的校园云政策，博主图方便直接选择了腾讯的“云+校园”（没错腾讯给了我500W广告费）（还有腾讯云的口碑确实不错。</li>
 	<li>然后进入腾讯云的“云+校园计划”页面完成注册和学生认证，这里就不赘述了。每天中午12：00在首页会有200个名额发放，基本上守住整点都能抢到，难度并不大。<img class="alignnone wp-image-24 size-large" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170319224228-1024x480.jpg" alt="" width="660" height="309" /></li>
 	<li>成功抢到资格后，进入个人中心的代金券一栏可以看到已经发放入账户。之后便可以开始我们的主机购买与域名的注册。需要注意的是：<strong>主机代金券每月发放一次，域名代金券每年发放一次，直至毕业，需要每月主动去“云+校园”的首页自己申领。</strong>具体的可以参考腾讯官方的论坛页面：<a href="http://bbs.qcloud.com/thread-21113-1-1.html">[校园专区] “一元扶持”优惠政策</a>
<h1>主机与域名：</h1>
</li>
 	<li><strong>首先购买主机。</strong>在云产品的目录下选择<strong>&lt;云服务器&gt;</strong>进入选购页面<img class="alignnone wp-image-25 " src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170319224605-300x221.jpg" alt="" width="450" height="331" /></li>
 	<li>代金券是64的，所以选择1核1G内存的服务器就OK了。</li>
 	<li>因为要用到第三方的博客模板，所以镜像提供方选择从服务市场选择。<img class="alignnone wp-image-26" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170319225129-300x194.jpg" alt="" width="353" height="228" /></li>
 	<li>博主选择的是杭州康展的Wordpress博客开源系统，有人推荐成都一个公司的基于Ubuntu的Wordpress模板，因为自己的虚拟机装的是CentOS，个人更喜欢Centos吧。（听说Ubuntu版本可以直接上手用，但是Centos的初次使用需要一些配置，这也是我后面踩的小坑啦所以你们自己权衡）选择的版本如下：（在建站模板中选择）</li>
 	<li><img class="alignnone wp-image-28" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170319225402-300x72.jpg" alt="" width="505" height="121" /></li>
 	<li>后面就是按照默认配置+个人信息填写+用券付款的路子完成云主机的购买。</li>
 	<li>购买成功之后需要1~2分钟的时间安装，安装成功之后如下：<img class="alignnone wp-image-29 " src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170319230427.jpg" alt="" width="712" height="136" /></li>
 	<li>这个时候就可以用建立好的公网IP直接访问，这个IP就归你啦！如果发现一直访问不了的，可能是主机的<strong>安全组配置</strong>出了问题，解决方法如下：在左侧功能栏中选择安全组。新建一个放通全部端口的安全组<img class="wp-image-31 alignright" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170319230817-268x300.jpg" alt="" width="299" height="334" />然后回到主机页面，在<strong>&lt;更多&gt;</strong>中选择<strong>&lt;配置安全组&gt;</strong>并且选择刚刚新建好的安全组就OK了。</li>
 	<li>或者在新建好的安全组页面中通过<strong>&lt;加入实例&gt;</strong>将云主机添加入这个安全组规则中。</li>
 	<li>此时用浏览器输入公网IP查看能否正常访问，如果进入WordPress的配置页面那就对啦。不过先不急，我们先把<strong>域名注册好并解析我们的公网IP</strong>，这样以后我们就可以通过炫酷的自定义专属域名访问我们的博客了，比如我自己的就是将www.cqkwithcoding.club解析入118.xx.xxx.xx</li>
 	<li>首先还是在云产品的列表中选择<strong>&lt;域名管理&gt;</strong>进入域名注册页面。拟定一个自己心水的域名并查重，选后缀，建议用.cn，不过只要年价小于25都相当于免费。之后的流程与购买主机的步骤并无二致就不再赘述了。<img class="alignnone wp-image-32" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170319231402-300x135.jpg" alt="" width="385" height="173" /></li>
 	<li>成功之后需要一段时间注册域名，之后可以在<strong>&lt;云解析&gt;</strong>中看到自己的域名。在<strong>&lt;更多&gt;</strong>中<strong>勾选开启CNAME加速，开启搜索引擎推送。</strong>然后下一步就是激动人心的解析了！</li>
 	<li>对于www服务，一般我们还需要添加一条A记录，即<strong>记录类型为A</strong>（将域名指向一个IPv4地址）<strong>主机记录为www</strong>（就是域名的前缀）线路类型默认即可，记录值填写你购买的<strong>云主机的公网IP，</strong>TTL选择1小时即可（即在DNS服务器缓存中的刷新时间）。</li>
 	<li>这样子我们就完成了域名注册与解析工作，等待<strong>大约10分钟</strong>后，我们可以进行测试。在Windows下ping该域名，看看是否能够ping通，并且查看返回的IP地址是否是云主机的公网IP。</li>
 	<li><img class="alignnone wp-image-33" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170319232243-300x147.jpg" alt="" width="482" height="236" />这样就大功告成咯。除此之外以后用ssh远程登录的时候也只要直接使用域名登录就可以了。</li>
</ul>
<h5>到这里博客的环境搭建基本完成。接下来就是最重要的获取权限与安装了。</h5>
<h1>WordPress获取权限与安装</h1>
<ul>
 	<li>经过云主机的注册和域名解析，我们可以用公网IP/注册域名访问自己的地址，进入后的页面如下：<img class="alignnone wp-image-37" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170320132547-300x206.jpg" alt="" width="454" height="312" /></li>
 	<li>数据库名/用户名/密码并不是我们购买主机时填写的那些信息，这些信息需要连接到我们的OS获取。这里需要用到一个<strong>linux端的远程工具putty</strong>来远程登录到服务器。</li>
 	<li>下载安装运行后在主界面填上<strong>服务器IP及端口，端口一般默认为22</strong>。Connection type默认勾选<strong>ssh</strong>，然后Open，进入登录界面。</li>
</ul>
<img class="alignnone size-medium wp-image-38" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170320133001-300x289.jpg" alt="" width="300" height="289" /><img class="alignnone wp-image-39" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170320133110-300x189.jpg" alt="" width="356" height="224" />
<ul>
 	<li>login as：   之后输入的是用户名，<strong>默认root</strong> 。然后回车，提示输入密码，就是你自己设置的主机密码，<strong>填写的时候光标不会动，不要认为是无效的输入，直接输好然后一个回车就连接成功了</strong>，进去之后其实就是一个shell，通过命令行来控制。</li>
</ul>
<img class="alignnone wp-image-40" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170320133450-300x189.jpg" alt="" width="490" height="309" />
<ul>
 	<li>这里我们需要键入<strong>cat default.pass</strong>命令查看FTP权限及数据库权限。可以看到MySQL的密码。</li>
</ul>
<img class="alignnone wp-image-41" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170320133537-300x189.jpg" alt="" width="453" height="285" />
<ul>
 	<li>获取密码等信息后就可在WordPress页面连接到数据库了，提交之后的页面如下：</li>
</ul>
<img class="alignnone wp-image-42 size-full" src="http://118.89.113.34/wp-content/uploads/2017/03/QQ截图20170320152529.jpg" alt="" width="889" height="312" />
<ul>
 	<li>进行安装...之后就不用我说咯，安装好之后就进入了Wordpress的后台模式，至此所有搭建过程完成，顺利的话其实不用1个小时。然后就可以开始DIY自己个人博客了，功能方面的都需要你们自己去摸索体验，一个好的博客也是需要走心经营出来的。</li>
</ul>
配置的过程有其他问题的也可以参考第三方镜像模板的对应文档，如果选择了我这套模板的话可以参考：<a href="http://market.qcloud.com/products/1434?productId=1434">WordPress 开源博客系统（Centos 7.2 64位）</a>
<h3>最后写给自己：</h3>
每一个领域想要有所建树肯定是要付出代价的，比你有天赋还比你更勤奋的人，一定比你过的更好，轰轰烈烈的信息革命让学习的成本变得从未如此之低，但大多数人还是因为缺少一种叫不可替代性的东西被时代淘汰了。私以为&lt;不可替代性&gt;=&lt;足够坚实的基础&gt;+&lt;不断点亮的技能树&gt;+&lt;解决问题的能力&gt;，写博客是提高自我的有效途径之一，希望自己能坚持下去，