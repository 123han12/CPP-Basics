### `function`函数的底层实现原理



#### 普通的`function`模版

```C++
#include <iostream>
#include <typeinfo>
#include <string>
#include <functional>

template<typename Fty>
class myfunction {};

// 函数类型的偏特化
template<typename R, typename A1>
class myfunction<R(A1)>
{
public:
	using PFUNC = R(*)(A1);
	myfunction(PFUNC pfunc) :_pfunc(pfunc) {}
	R operator()(A1 arg)
	{
		return _pfunc(arg); // hello(arg)
	}
private:
	PFUNC _pfunc ; // 函数指针类型
};


template<typename R, typename A1, typename A2>
class myfunction<R(A1, A2)>
{
public:
	using PFUNC = R(*)(A1, A2);
	myfunction(PFUNC pfunc) :_pfunc(pfunc) {}
	R operator()(A1 arg1, A2 arg2)
	{
		return _pfunc(arg1, arg2); // hello(arg)
	}
private:
	PFUNC _pfunc; // 函数指针类型
};

int main()
{
	function<void(string)> func1(hello);
	func1("hello world!"); // func1.operator()("hello world!")

	myfunction<int(int, int)> func2(sum) ; 
	cout << func2(10, 20) << endl; 
	return 0;
}

```

**可以理解`function`是一种可调用对象的封装器，内部封装一个可调用对象，通过调用`function`模版的`operator()`重载运算符内部调用封装的可调用对象。**

####通过模版参数包和函数参数包，将`function`模版的个数进行统一。

```C++
#include <iostream>
using namespace std;

template<typename T>
class myfunction
{};

template<typename R, typename... Arg> // 前面跟三个点表示其为一个参数包
class myfunction<R(Arg...)>
{
public:
	using PFUNC = R(*)(Arg...);  // Arg是一个模版参数包 后面跟... 表示对参数包的展开
	myfunction(PFUNC pfunc) : _pfunc(pfunc) {}

	R operator()(Arg ... arg)   // arg是一个函数参数包 
	{
		return _pfunc(arg...);
	}
private:
	PFUNC _pfunc;

};

int sum(int a, int b)
{
	return a + b;
}

int main()
{

	myfunction<int(int , int) > obj = sum ; 
	cout << sum(10, 20);
	return 0;
}
```

> ==在进行模版的特例化的时候，必须存在一个主模版，如果将上述代码中的[4,6]之间的主模版去掉，这里会报错。== 