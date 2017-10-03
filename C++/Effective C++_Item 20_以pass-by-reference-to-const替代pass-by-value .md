> 我们经常在函数传参的时候看到`const &`这样的形式，而不是简单的`&`或传值，这里面肯定是有大大的学问的

如果一个函数是`pass-by-value`，那么传入函数内部时，编译器会调用**copy构造函数**构造一份实参的副本，执行函数内部的逻辑，然后将这个副本返回，所以我们只是传值的话，是不会对原实参作改动的。
但是更严重的是，`pass-by-value`还有可能产生严重的性能浪费，以一个简单的继承体系为例：
```cpp
class Person{
public:
	Person()=default;
	virtual ~Person()=default; //注意声明析构函数为virtual
	...
private:
	string name;
	string number;
};
class Student:public Person{
public:
	Student()=default;
	~Student()=default;
private:
	string stu_name;
	string stu_number;
};
```
接下来调用函数分析：
```c++
bool testFunc(Student item);
Student s;
testFunc(s);
```
传递参数时，`Student`调用一次copy构造函数，以及脱离作用域的一次析构调用，但是！`Student`继承自`Person`类，而且私有域内还有`string`类型的类成员，所以每次构造一个`Student`类，还会构造一个`Person`基类，加上私有域中`string`类也要调用构造函数...那么这么一个函数调用，其不可见部分的函数调用，可以说是非常影响性能的，最后一共要调用**“六次构造函数和六次析构函数”**！
而如果我们使用`pass-by-reference-to-const`调用函数：
```cpp
testFunc(const Student& item);
```
因为是`&`，所以传入实参时不会构造实参副本，也就免去了大量的构造-析构的开销，声明为`const`，说明实参是**只读**的，在函数内部不能对实参进行`写`操作，在不改变语义的情况下，通过参数的传递方式，很自然的完成了成功的性能优化。
如果设计的基类中有`virtual`函数，那么`pass-by-reference`还避免了对象切割（slicing）问题，对象切割不是多态，多态是通过指针或者引用实现的，而对象切割是**派生类直接向基类传递或者强制类型转换**的时候，自己（基类）的部分在转换时丢失了，就像被切割了一样。
```cpp
Person* item=new Person();
Student test;
...
item=&test;  //这是多态
*item=test;  //这是对象切割
```
所以如果我们在`Person`类中加入一个`virtual`函数，并且在`Student`中重写：
```cpp
class Person{
...
	virtual void show(){
		cout<<name<<number<<endl;
	}
}

class Student{
...
	void show() override{
		cout<<stu_name<<stu_number<<endl;
	}
}
```
如果在某个函数中，我们需要调用其中的`virtual`函数，且需要避免对象切割问题，那么我们就应该使用`pass-by-reference`：
```cpp
void testSilce(Person item){
	item.show();
	...
	}
void testSlice2(const Person& item){
	item.show();
	}
Student test;
testSlice(test);  //强制类型转换，发生对象切割
testSlice2(test);  //多态
```
在C++中，`reference`的底层实现其实还是靠指针的，所以如果函数传递的对象是`built-in types` **[同时也包括了STL对象和函数对象！]**，那么其实传值也不是一个那么糟糕的方案，至少对于`built-in types`，传值是有性能优势的，毕竟不像我们上面举的例子，内置类型都是非常轻量化的类型，甚至效率要来的更加高。
但是我们并不能认为自定义类型的看似小型的类，也可以用`pass-by-value`的方法提高性能，这是不对的，copy构造函数的代价往往不是我们能够轻易准确估量的，甚至还和具体编译器的具体实现有关。
所以记住，除了`built-in types , STL迭代器对象 , 函数对象`,其他实践中，尽量使用`pass-by-reference-to-const`是一个良好的编程习惯！