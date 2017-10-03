## 【转】<C++ Primer 5th>Template模板笔记

<br>

1.模板实例化：C++中的模板是一个函数或者类的蓝图，编写了不局限于类型的通用代码。模板定义本身不参与编译，而是编译器根据模板的用户使用模板时提供的类型参数生成代码，再进行编译，这一过程被称为模板实例化。用户提供不同的类型参数，就会实例化出不同的代码。
2.函数模板：编译器可以根据用户使用函数模板时提供的实参，推断出函数模板的类型参数，这被称为模板实参推断。
```cpp
template<typename T>  
int compare(const T& left, const T& right) {  
    if (left < right) {  
        return -1;   
    }  
    if (right < left) {  
        return 1;   
    }  
    return 0;  
}  
  
compare(12, 13); //推断类型为int，实例化int版本  
compare(0.1, 0.2); //推断类型为double，实例化double版本 
``` 
除了类型参数之外，还可以在函数模板中定义非类型参数，我们可以在模板定义中使用一个具体类型来指定它们。
```cpp
template<unsigned N, unsigned M> //使用unsigned标记非类型参数  
int compare(const char (&left) [N], const char (&right) [M]) {  
    return compare(left, right);  
}  
  
compare("hi", "hello"); //推断出N=3，M=6  
```
一个非类型参数可以是一个整形，或者是指针或左值引用。作为整形推断的结果必须是一个常量表达式，指针和引用必须指向具有静态生存周期的对象（static或者全局）或者是nullptr或0。此外，当用户把函数模板赋值给一个指定类型的函数指针时，编译还可以根据这个指针的类型，对模板参数进行推断。
```cpp
template<typename T>  
int compare(const T&, const T&){...}  
  
int (*pf) (const int&, const int&) = compare; //ok，推断T的类型为int  
```
另外，函数模板可以声明为inline或者constexpr的，将它们放在template之后，返回值之前即可。
3.类模板：与函数模板不同，类模板不能推断实例化，只能在使用时由用户显示制定类型参数来实例化。
```cpp
template<typename T>  
class Foo {  
...  
};  
Foo f1; //错误，必须指定类型  
Foo<int> f2; //ok  
```
类模板的成员函数既可以定义在内部，也可以定义在外部。定义在内部的被隐式声明为inline，定义在外部的类名之前必须加上template。
```cpp
template<typename T>  
class Foo{  
public:  
    void f1() {...} //ok，内部定义  
    void f2();  
};  
  
template<typename T>  
void Foo<T>::f2 {...} //注意外部定义要加上template<>  
```
类模板中既可以把非模板函数、类声明为自己的友元，也可以将模板函数、类声明为自己的友元。如果一个类模板包含一个非模板友元，那么这个友元可以访问此模板的所有实例。如果一个类模板包含一个模板友元，那么根据实现，既可以授权给这个模板友元的一个实例，也可以授权给这个模板友元的所有实例。在C++11中，我们还可以将模板类型参数声明为友元。
```cpp
class A{...};  
template<typename T>  
class B{...};  
template<typename T>  
class C{...};  
  
template<typename T>  
class D {  
    friend class A; //A是D所有实例的友元  
    friend class B<T>; //B<T>是D<T>的友元  
    template<typename X> friend class C; //C的所有实例都是D的所有实例的友元  
    friend T; //T是D<T>的友元  
};  
```
类模板中可以声明static成员，它必须有且仅有一个定义。但是，每个不同的模板实例都会有一个独有的static成员对象。还有一点需要注意的是，类模板的成员函数并非全部实例化的，而是哪个成员函数被使用到了，哪个成员函数才会被实例化，而未被使用到的成员函数是不会被实例化的（静态成员例外，一定会被实例化）。
4.类模板别名：我们可以使用typedef为类模板定义个别名。在C++11中，使用using语句可以固定一个或多个类型参数（partial模板？）。
```cpp
template <typename T> using WithNum = std::pair<T, int>;  
WithNum<std::string> strs; //实际类型，pair<string, int>  
WithNum<int> ints; //实际类型，pair<int, int>  
```
5.模板中使用类的类型成员：在C++中，假定通过域操作符访问的是名称而不是类型。所以在模板中，如果想使用一个模板类型参数的类型成员，就要通过使用typename关键字显式的通知编译器，域操作符后面的是一个类型。
```cpp
template <typename containerT, typename elemT>  
typename containerT::difference_type count(const containerT& c, const elemT& e) {  
    typename containerT::difference_type ret = 0;  
    typename containerT::const_iterator iter = c.cbegin();  
    while (iter != c.cend()) {  
        if (*iter == e) {  
            ret++;  
        }  
        iter++;  
    }  
    return ret;  
}  
```
6.默认模板实参：我们可以为类模板指定默认实参，在C++11中还可以为函数模板指定默认实参。
```cpp
template<typename T, typename F = std::less<T>> //指定F的默认值是std::less<T>  
int compare(const T& left, const T& right, F f = F()) {  
    if (f(left, right)) {  
        return -1;  
    }  
    if (f(right, left)) {  
        return 1;  
    }  
    return 0;  
}  
compare(1, 2); //未指定第三个参数，使用默认值std::less<int>  
  
template<typename T = int>  
class A {  
    ...  
};  
A<> a; //a的类型为A<int>，注意空的<>必须写上  
```
7.成员模板：模板类和普通类都可以包含模板成员函数，模板函数成员支持实参推断，但是要注意这些模板函数不能是虚函数(virtual)。另外，如果在模板类外定义一个模板成员函数需要写两次template.
```cpp
class A {  
public:  
    template<typename T>  
    void my_func(const T&){...}  
};  
A a;  
a.my_func(12); //实例化A::my_func(const int &)  
  
template<typename T>  
class B {  
public:  
    template<typename U>  
    void my_func(const U&);  
}  
template<typename T>  
template<typename U>  
void B<T>::my_func(const U&) {...} //注意这里的两层template  
```
8.模板显式实例化：模板被用到时才实例化生成代码这意味着，在不同的文件中有各自的实例化代码。在大型的工程中，这会是一个极大的浪费。为了避免这种情况，在C++11中，可以通过在一个文件中显示实例化模板，在其他文件中extern这个模板的实例化版本来避免重复实例化的问题。注意，因为无法预测其他文件会怎样使用这个实例化版本，显示实例化会实例化类模板的所有成员函数，而不是使用到哪个成员函数再实例化哪个。
```cpp
template<typename T>  
class A {  
    ...  
};  
template class A<std::string>; //显式实例化A，实例化了所有成员函数  
extern template class A<std::string>; //A<std::string>在别处实例化了，就用那个版本吧  
```
9.模板实参推断与类型转换：当使用实参调用模板函数时，编译器会进行模板实参推断来自动判断模板类型参数，通常不会对实参进行类型转换，只有以下几种情况例外：
顶层const会被忽略
普通对象赋值给const引用
数组名转换为头指针
函数名转换为函数指针
用户显式指定类型的模板函数与模板函数中的非类型参数不受此限制，在调用时可以进行普通的类型转换。
10.显示指定部分模板参数：使用函数模板时，可以显示指定部分模板参数，其余的参数交由编译器自动推断完成。这对于需要使用者指定返回值类型的函数模板尤其有用。但是需要值得注意的是，与函数的默认实参相同，我们必须从左向右逐一指定。
```cpp
template<typename T1, typename T2, typename T3>  
T1 sum(T2 v2, T3 v3) {  
    return static_cast<T1>(v1 + v2);  
}  
//可以指定T1, T2和T3交由编译器来推断  
auto ret = sum<long>(1L, 23);  
  
template<typename T1, typename T2, typename T3>  
T3 sum(T1 v1, T2 v2) {...}  
auto ret = sum<long>(1L, 23); //错误，只能从左向右逐一指定  
auto ret = sum<long,int,long>(1L,23); //ok, 谁叫你把最后一个T3作为返回类型的呢？全部指定吧  
```
11.尾置返回类型与模板：编写函数模板的过程中，有时需要让编译器为我们自动推断返回值的类型，这时尾置返回类型可以发挥作用。
```cpp
template<typename It>  
auto doSomething(It beg, It end) -> decltype(*beg) {...} 
``` 
12.引用折叠(reference collapsing)：虽然C++禁止我们定义引用的引用(reference to reference)，但是编译器在以下四种情况下还是会自己产生引用的引用出来：
```cpp
typedef：typedef T&& T'
auto：auto&& a = b;
decltype：decltype(a)&& c = b;
```
模板函数自动类型推断：template<typename T> void func(T&&)
为了解决这一问题，C++中引入了引用折叠(reference collapsing)规则。该规则规定，除了右值引用的右值引用(Rref to Rref)被折叠为右值引用之外，其他所有情况都会被折叠为左值引用。如下表所示：
```
折叠	Lref	Rref
Lref	Lref	Lref
Rref	Lref	Rref
```
13.模板函数的右值引用参数类型推断规则：在C++11中，如果一个模板函数的参数是一个右值引用(例如：T&&)，那么它的模板类型参数推断遵循以下规则：
如果使用一个类型X的左值(lvalue)实例化这个函数，则T的类型会被推断为X&
如果使用一个类型X的右值(rvalue)实例化这个函数，则T的类型会被推断为X
```cpp
template<typename T>  
void func(T&& t) {...}  
  
int a = 42;  
func(a);   //T被推断为int&  
func(a*1); //T被推断为int  
```
14.理解std::move：了解了引用折叠和C++11模板函数关于T&&的类型推断规则之后，我们就可以了解C++11新增的标准库函数std::move的工作原理了。std::move的作用是将传入的任何参数(lvalue或者rvalue)都转换成右值引用然后返回。std::move的代码大致如下：
```cpp
template<typename T>  
typename remove_reference<T>::type&& move(T&& t) {  
    return static_cast<typename <span style="font-family: Arial, Helvetica, sans-serif;">remove_reference<T>::type&&</span>>(t);  
}  
```
情况1：如果使用一个X的左值调用std::move，则T将会被推断为X&，于是std::move将被实例化为如下的样子：
```cpp
X&& move(X& && t){  
    return static_cast<X&&>(t);  
}  
```
再应用引用折叠规则，std::move又变成了：
```cpp
X&& move(X& t) {  
    return static_cast<X&&>(t);  
}
```  
ok，函数工作正常，将一个左值引用转化成了右值引用并返回。
情况2：如果使用一个X的右值调用std::move，则T将会被推断为X，于是std::move将被实例化为如下的样子：
```cpp
X&& move(X&& t){  
    return static_cast<X&&>(t);  
}  
```
ok，虽然函数什么都没做，但是也返回了一个右值引用。
15.理解std::forward：再说说std::forward，std::forward又被称为完美转发，它将实例化模版函数的值保持原样传递给另外一个函数。通常使用场景如下：
```cpp
template<typename F, typename T>  
function_traits<F>::RetType wrapper(F f, T&& t)  { //假设我们有一个function_traits模板库，可以提取任意类型函数f的返回值类型  
    //do something  
    return f(std::forward<T>(t));  
}  
```
为什么要用std::forward来转发？这是因为即使你传递一个右值给T&&，t也是一个左值引用(t有名字就有地址，有地址就是一个左值引用，呵呵)，所以不使用std::forward处理，你是无法将一个右值转发给f的。所以，std::forward所做的事情就是，当且仅当你传入一个右值给外层函数的时候，将t转换为右值引用并返回。std::forward的大致实现如下：
```cpp
template<typename T>  
T&& forward(typename remove_reference<T>::type& t) {  
    return static_cast<T&&>(t);  
} 
``` 
情况1：如果使用一个X的左值调用外层函数wrapper，则wrapper的T将会被推断为X&，于是std::forward将被实例化为如下的样子：
```cpp
X& && forward(X& t){  
    return static_cast<X& &&>(t);  
}  
```
再应用引用折叠规则，std::forward又变成了：
```cpp
X& forward(X& t) {  
    return static_cast<X&>(t);  
}  
```
ok，这就是我们想要的，传入一个左值引用给外层函数wrapper，std::forward转发了一个左值引用。
情况2：如果使用一个X的右值调用外层函数wrapper，则wrapper的T将会被推断为X，于是std::forward将被实例化为如下的样子：
```cpp
X&& forward(X& t){  
    return static_cast<X&&>(t);  
}  
```
ok，我们传入一个右值给外层函数wrapper，std::forward转发了一个右值引用。
16.函数模板重载：函数模板之间，函数模板与普通函数之间可以重载。编译器会根据调用时提供的函数参数，调用能够处理这一类型的最“特殊”的版本。一般的说：普通函数>特殊模板（限定了T的形式的，指针、引用、容器等）>普通模板（对T没有任何限制的）。对于如何判断某个模板更加特殊，原则如下：如果模板B的所有实例都可以实例化模板A，而反过来则不行，那么B就比A特殊。
```cpp
template<typename T>  
void func(T& t) {...} //通用模板函数  
  
template<typename T>  
void func(T* t) {...} //指针版本  
  
void func(string* s) {...} //普通函数  
  
int i = 10;  
func(i); //调用通用版本，其他函数或者无法实例化或者不匹配  
func(&i); //调用指针版本，通用版本虽然也可以用，但是编译器选择最特殊的版本  
string s = "abc";  
func(&s); //调用普通函数，通用版本和特殊版本虽然也都可以用，但是编译器选择最特化的版本  
```
17.类模板特化与偏(partial)特化：类模板无法像函数模板那样重载，但是你可以通过特化来为特定的类型指定你想要的行为。类模板的特化只需要模板名称相同并且特化列表<>中的参数个数对应上即可，模板参数列表不必相同；你既可以用另外一个模板特化基础模板，也可以全部用真实类型参数特化它，甚至还可以用部分真实类型参数部分模板参数来完成特化。编译器同样会根据上面判断特殊的原则来实例化对应的模板。
```cpp
template<typename T>                                                                                 
class S {                                                                                             
public:                                                                                               
   void info() {                                                                                     
       printf("In base template\n");                                                                 
   }                                                                                                 
};  
                                                                                                  
template<>                                                                                           
class S<int> {                                                                                       
public:                                                                                               
   void info() {                                                                                     
       printf("In int specialization\n");                                                           
   }                                                                                                 
};  
                                                                                                  
template<typename T>                                                                                 
class S<T*> {                                                                                         
public:                                                                                               
   void info() {                                                                                     
       printf("In pointer specialization\n");                                                       
    }                                                                                                 
};                                                                                                   
  
template <typename ReturnType, typename... Args>                                                     
class S<ReturnType(Args...)> {                                                                       
public:                                                                                               
   void info() {                                                                                     
       printf("In Function specialization\n");                                                       
   }                                                                                                 
};                                                                                                   
  
int func(int i) {                                                                                     
   return 2 * i;                                                                                     
}  
  
S<float> s1;                                                                                     
s1.info();     //调用base模板                                                                                      
S<int> s2;                                                                                       
s2.info();     //调用int特化版本                                                                                      
S<float*> s3;                                                                                     
s3.info();     //调用T*特化版本                                                                                      
S<decltype(func)> s4;                                                                             
s4.info();    //调用函数特化版本  
```
18.函数模板特化：函数模板不能够偏特化（这个被称为重载），只能够把所有的模版参数都替换掉。
```cpp
template<typename T>                                                                                     
void func(T t) {                                                                                         
   printf("Calling general template func\n");                                                           
}  
                                                                                                      
template<typename T>                                                                                     
void func(T* t) {                                                                                       
    printf("Calling pointer template func\n");                                                           
}  
  
template<> void func(int x) {...}    //base模板的特化  
template<>void func(int *x) {...}    //重载的指针函数模板的特化 
``` 
19.除了可以特化类模板以及函数模板之外，还可以对类模板中的成员函数和普通静态成员变量进行特化。
```cpp
template<typename T>                                                                                     
class S {                                                                                               
public:                                                                                                 
    void info() {                                                                                       
        printf("In base template\n");                                                                   
    }                                                                                                   
    static int code;                                                                                     
};                                                                                                       
  
template<typename T>                                                                                     
int S<T>::code = 10;                                                                                     
  
template<>                                                                                               
int S<int>::code = 100;    //普通静态成员变量的int特化                                                                                  
  
template<>                                                                                               
void S<int>::info() {    //成员函数的int特化                                                                                     
     printf("In int specialization\n");                                                                   
}                                                                                                       
                                                                   
S<float> s1;                                                                                         
s1.info();    //普通版本                                                                                          
printf("Code is: %d\n", s1.code);    //code = 10                                                                    
S<int> s2;                                                                                           
s2.info();   //int特化版本                                                                                          
printf("Code is: %d\n", s2.code);   //code = 100  
```
20.变参模板：C++11中新增了变参模板，一个模板可以接收可变数目(零到任意)的模板参数。这个可变数目参数被称为参数包。在template关键词后面的<>中的被称为模板参数包，在函数模板参数列表中的被称为函数参数包。我们可以通过sizeof运算符得知参数包中的参数个数。
```cpp
template<typename ... Args>  
size_t how_many(Args ... args) {  
    return size_of...(args);  
}  
```
除了可以获取参数包的大小之外，我们还可以对它做扩展(expand)操作。通过在参数包后面加上...表示扩展操作，让编译器打开这个包，同时我们还可以指定打开包时对其中的每个元素执行的额外操作。扩展这个操作通常用来做转发，将参数包转发给另外的函数。
```cpp
template<typename ... Args>  
void bad_forward(Args ... args) {  
    container c;  
    c.emplace(args...); //不附加操作的直接展开，有问题的转发  
}  
  
template<typename ... Args>  
void perfect_forward(Args&& ... args) {  
    container c;  
    c.emplace(std::forward<Args>(args)...); //完美转发，展开参数包的同时指定了std::forward操作  
}  
```
除了把参数包转发给别的函数之外，我们还可以通过模板递归自己处理参数包中的每一个成员。
```cpp
template<typename T>  
void toOneLine(strstream& ss, const T& t) { //终止递归的模板函数  
    ss<<t;  
}  
  
template<typename T, typename ... Args>  
void toOneLine(strstream& ss, const T& t, const Args& ... args) { //参数包中不止一个值的时候调用的函数  
    ss<<t<<"\t";  
    return toOneLine(ss, args...); //包扩展  
}  
  
string s = "";strstream ss;  
toOneLine(ss, 1, 2.0, true, string("abc"));  
s.assign(ss.str());  
```
上面的代码已经有些模板元编程的味道了，但是根据前面所讲的函数模板重载规则，还是能够理解它是怎样运行的。
toOneLine(1, 2.0, true, string("abc"))这个调用匹配了第二个函数模板，实例化成toOneLine(strstroeam, int, double, bool, string)。这个函数在其内部只处理了第二个参数int，之后用参数包展开的方式递归调用了这个函数模板自身(在函数模板调用的层面上来看，的确是递归调用，但是从实际实例化的函数层面上来看并不是递归)，被调用的函数实例化为toOneLine(strstroeam, double, bool, string)。然后这个过程反复执行，直至参数包里面只含有一个参数string时，两个模板都能匹配。但是根据函数模板重载规则，更特殊的那个会被调用，所以第一个函数模板被实例化为toOneLine(strstroeam, string)，执行之后终止了递归，整个处理过程完成。
总结一下执行流程：
toOneLine(strstroeam, int, double, bool, string)模板2 -> toOneLine(strstroeam, double, bool, string)模板2  -> toOneLine(strstroeam, bool, string)模板2 ->toOneLine(strstroeam, string) 模板1。