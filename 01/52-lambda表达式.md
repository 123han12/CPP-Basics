​	 

【注：C++14起Lambda支持默认参数】

一、[Lambda](https://so.csdn.net/so/search?q=Lambda&spm=1001.2101.3001.7020)简介
----------------------------------------------------------------------------

`lambda`表达式是`C++11`中引入的一项新技术，利用`lambda`表达式可以编写内嵌的匿名函数，用以替换独立函数或者函数对象，并且使代码更可读。

Lambda 表达式完整的格式如下：

    [捕获列表] (形参列表) mutable 异常列表-> 返回类型{    函数体}

各项的含义：

*   **捕获列表**：捕获外部变量，捕获的变量可以在函数体中使用
    *   \[\]：默认不捕获任何变量；
    *   \[=\]：默认以值捕获所有变量；
    *   \[&\]：默认以引用捕获所有变量；
    *   \[x\]：仅以值捕获x，其它变量不捕获；
    *   \[&x\]：仅以引用捕获x，其它变量不捕获；
    *   \[=, &x\]：默认以值捕获所有变量，但是x是例外，通过引用捕获；
    *   \[&, x\]：默认以引用捕获所有变量，但是x是例外，通过值捕获；
    *   \[this\]：通过引用捕获当前对象（其实是复制指针）；
    *   \[\*this\]：通过传值方式捕获当前对象；
*   **形参列表**：和普通函数的形参列表一样（可省略，即无参数列表）
*   **`mutable`**：`mutable `关键字，如果有，则表示在函数体中可以修改捕获变量
*   **异常列表**：`noexcept / throw(...)`,和普通函数的异常列表一样，可省略，即代表可能抛出任何类型的异常。
*   **返回类型**：和函数的返回类型一样。可省略，如省略，编译器将自动推导返回类型。
*   **函数体**：代码实现,不能省略。

==`lambda`表达式因为禁用了赋值操作符，所以不可以直接赋值==，即使他们形参和返回值完全一样，但是`lambda`表达式没有禁用复制构造函数，所以你仍然可以用一个`lambda`表达式去初始化另外一个`lambda`表达式而产生副本，并且`lambda`表达式也可以赋值给相对应的函数指针，这也使得你**完全可以把`lambda`表达式看成对应函数类型的指针。**

另外，lambda表达式不能有默认参数，不支持可变参数。

**使用示例：**

```C++
void LambdaDemo()
{
    int a = 1;
    int b = 2;
    auto lambda = [a, b](int x, int y) mutable throw() -> bool
    {
        return a + b > x + y;
    };
    bool ret = lambda(3, 4);
}
```

### 二、`lambda`编译器实现原理

编译器实现 **lambda 表达式**大致分为一下几个步骤：

1.  创建 **lambda匿名类**，实现构造函数，使用 lambda 表达式的函数体重载 **operator()**（所以 lambda 表达式 也叫匿名函数对象）
2.  创建 lambda 对象
3.  通过对象调用 **operator()**

所以编译器将 lambda 表达式翻译后的代码：

```C++
class lambda_xxxx
{
private:
    int a;
    int b;
public:
    lambda_xxxx(int _a, int _b) :a(_a), b(_b)
    {
    }
    bool operator()(int x, int y) throw()
    {
        return a + b > x + y;
    }
};
void LambdaDemo()
{
    int a = 1;
    int b = 2;
    lambda_xxxx lambda = lambda_xxxx(a, b);
    bool ret = lambda.operator()(3, 4);
}
```

其中，类名 `lambda_xxxx` 的 `xxxx` 是为了防止命名冲突加上的。

1.  `lambda` 表达式中的**捕获列表**，对应 `lambda_xxxx` 类的 **private 成员**
2.  `lambda`  表达式中的**形参列表**，对应 `lambda_xxxx` 类成员函数 **operator() 的形参列表**
3.  `lambda`  表达式中的 **mutable**，表明 `lambda_xxxx` 类成员函数 **operator() 的是否具有常属性 `const`**，即是否是 **常成员函数**
4.  `lambda`  表达式中的**返回类型**，对应`lambda_xxxx`类成员函数 **operator() 的返回类型**
5.  `lambda`  表达式中的**函数体**，对应`lambda_xxxx`类成员函数 **operator() 的函数体**

另外，`lambda` 表达 捕获列表的捕获方式，也影响 对应`lambda_xxxx` 类的 `private` 成员 的类型

1.  值捕获：`private `成员的类型与捕获变量的类型一致
2.  引用捕获：`private` 成员 的类型是捕获变量的引用类型

但有一种特殊情况需要说明：

如果 lambda 表达式不捕获任何外部变量，且有`lambda_xxxx `类到 **函数指针** 的类型转换，会有额外的代码生成，例如：

```C++
typedef int(_stdcall *Func)(int);
int Test(Func func)
{
	return func(1);
}
void LambdaDemo()
{
	Test([](int i) {
		return i;
	});
}
```

Test 函数接受一个函数指针作为参数，并调用这个函数指针。实际调用 Test 时，传入的参数却是一个 Lambda 表达式，所以这里有一个类型的隐式转换：`lambda_xxxx`=> 函数指针。

上面已经提到，==Lambda 表达式就是一个 `lambda_xxxx` 类的匿名对象==，与函数指针之间按理说不应该存在转换，但是上述代码却没有问题。

其问题关键在于，上述代码中，lambda 表达式没有捕获任何外部变量，即 `lambda_xxxx` 类没有任何成员变量，在 `operator()` 中也就不会用到任何成员变量，也就是说，`operator() `虽然是个成员函数，它却不依赖 this 就可以调用。

因为不依赖 `this`，所以 一个 `lambda_xxxx `类的匿名对象与函数指针之间就存在转换的可能。

大致过程如下：

1.  在 `lambda_xxxx `类中生成一个**静态函数**，静态函数的函数签名(函数原型)**与 `operator()` 一致**，在这个静态函数中，通过一个**空指针**去调用该类的 `operator()`
2.  在 `lambda_xxxx `重载与函数指针的类型转换操作符，在这个函数中，返回静态函数的地址。

```C++
typedef int(_stdcall *Func)(int);
 
class lambda_xxxx 
{
private:
	//没有捕获任何外部变量，所有没有成员
public:
        /*...省略其他代码...*/
	int operator()(int i)
	{
		return i;
	}
	static int _stdcall lambda_invoker_stdcall(int i)
	{
		return ((lambda_xxxx *)nullptr)->operator()(i);
	}
 
	operator Func() const
	{
		return &lambda_invoker_stdcall;
	}
};
 
int Test(Func func)
{
	return func(1);
}
void LambdaDemo()
{
	auto lambda = lambda_xxxx ();
	Func func = lambda.operator Func();
	Test(func);
}
```

