### `OOP`中的重载,隐藏，覆盖(覆盖的知识下一篇文章介绍)





#### 重载

> ```C++
> class Base
> {
> public:
> 	Base(int data = 10) :ma(data) {}
> 	void show() { cout << "Base::show()" << endl; } //#1
> 	void show(int) { cout << "Base::show(int)" << endl; }//#2
> protected:
> 	int ma;
> };
> class Derive : public Base 
> {
> public:
> 	Derive(int data = 20) :Base(data), mb(data) {}
> private:
> 	int mb;
> };
> int main()
> {
> 	Derive d ; 
> 	
>    d.show() ;
>    d.show(10) ; 
>    
>    return 0; 
> 
> } 
> ```
>
> ![image-20230909003946967](assets/image-20230909003946967.png)
>
> **基类中的show()方法构成了重载：** 一组函数要重载，==必须处在同一个作用域==当中；而且函数名字相同，参数列表不同。



#### 隐藏 : 主要指的是作用域的隐藏

> ```C++
> class Base
> {
> public:
> 	Base(int data = 10) :ma(data) {}
> 	void show() { cout << "Base::show()" << endl; } //#1
> 	void show(int) { cout << "Base::show(int)" << endl; }//#2
> protected:
> 	int ma;
> };
> class Derive : public Base 
> {
> public:
> 	Derive(int data = 20) :Base(data), mb(data) {}
> 	void show() { cout << "Derive::show()" << endl; } //#3
> private:
> 	int mb;
> };
> int main()
> {
> 	Derive d ; 
> 	
>    d.show() ;
>    d.show(10) ;  // 错误 "Derive::show 不接受1个"
>    
>    return 0; 
> 
> } 
> ```
>
> **隐藏**定义：**在继承结构中，派生类的同名成员，把基类的同名成员给隐藏调用了.如果一个函数名在基类和派生类相同，则在重载匹配的时候只在派生类的作用域内进行匹配，匹配不上则报错**。上述代码中的`Derive` 中的`show()`把`Base`类中的`show()`与`show(int)`隐藏了。 
>
> 当`d`调用`show()`的时候，==**只**==从派生类的作用域中进行**匹配** 。`ok` 可以匹配上
>
> 当`d`调用`show(int)`的时候，==**只**==从派生类的作用域中进行**匹配**  `no` 没有匹配的函数，报错!
>
> 如果想调用基类中的同名成员(假设`show()`的访问权限是`ok`的)，以`show`函数为例：
>
> ```C++
> class Base
> {
> public:
> 	Base(int data = 10) :ma(data) {}
> 	void show() { cout << "Base::show()" << endl; } //#1
> 	void show(int) { cout << "Base::show(int)" << endl; }//#2
> protected:
> 	int ma;
> };
> class Derive : public Base 
> {
> public:
> 	Derive(int data = 20) :Base(data), mb(data) {}
> 	void show() { cout << "Derive::show()" << endl; } //#3
> private:
> 	int mb;
> };
> int main()
> {
> 	Derive d ; 
> 	
>    d.show() ;
>    d.show(10) ;  // 错误 "Derive::show 不接受1个"
>    
>    d.Base::show() ; // ok 直接调用指定作用域内的函数。
>    return 0; 
> 
> } 
> ```





#### 继承中的类型转换问题

> 







```C++
/*
重载、隐藏、覆盖

============================================================
1.把继承结构，也说成从上（基类）到下（派生类）的结构
2.
基类对象 -> 派生类对象
派生类对象 -> 基类对象

基类指针（引用）-> 派生类对象
派生类指针（引用）-> 基类对象

总结：在继承结构中进行上下的类型转换，默认只支持从下到上的类型的转换 OK
*/
class Base
{
public:
	Base(int data = 10) :ma(data) {}
	void show() { cout << "Base::show()" << endl; } //#1
	void show(int) { cout << "Base::show(int)" << endl; }//#2
protected:
	int ma;
};
class Derive : public Base
{
public:
	Derive(int data = 20) :Base(data), mb(data) {}
	void show() { cout << "Derive::show()" << endl; } //#3
private:
	int mb;
};
int main()
{
	Base b(10);
	Derive d(20);

	// 基类对象b <- 派生类对象d   类型从下到上的转换  Y
	b = d;

	// 派生类对象d <- 基类对象b   类型从上到下的转换  N
	// d = b;

	// 基类指针（引用）<- 派生类对象  类型从下到上的转换 Y
	Base *pb = &d;
	pb->show();
	//((Derive*)pb)->show();
	pb->show(10);

	// 派生类指针（引用）<- 基类对象  类型从上到下的转换  N
	Derive *pd = (Derive*)&b; // 不安全，涉及了内存的非法访问！
	pd->show();

#if 0
	Derive d;
	d.show();
	d.Base::show();
	//“Derive::show”: 函数不接受 1 个参数
	d.Base::show(10);
	d.show(20);//优先找的是派生类自己作用域的show名字成员；没有的话，才去基类里面找
#endif
	return 0;
}
#endif
```

