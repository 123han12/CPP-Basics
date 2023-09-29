#### `bind`绑定器的基本使用

```C++

using namespace placeholders;  // _1 占位符所在的命名空间
/*
C++11 bind绑定器 => 返回的结果还是一个函数对象
*/
void hello(string str) { cout << str << endl; }
int sum(int a, int b) { return a + b; }

class Test
{
public:
	int sum(int a, int b) { return a + b; }
};
int main()
{
	// bind是函数模板 可以自动推演模板类型参数
	bind(hello, "hello bind!")();
	cout << bind(sum, 10, 20)() << endl;
	cout << bind(&Test::sum, Test(), 20, 30)() << endl;

	// 参数占位符  绑定器出了语句，无法继续使用
	bind(hello, _1)("hello bind 2!") ; // _1 表示将传入的第一个参数放入_1的位置。
	
   cout << bind(sum, _1, _2)(200, 300) << endl ; // 调用sum(200 , 300) 
	cout << bind(sum, _2 , _1)(200 , 300) << endl ; // 调用sum(300 , 200)
   
	// 此处把bind返回的绑定器binder就复用起来了,延长了binder的生存期
   
	function<void(string)> func1 = bind(hello, _1);
	
   func1("hello china!");
	
   func1("hello shan xi!");
	
   func1("hello si chuan!");

	return 0;
}

```



####通过`bind和function`实现小型线程池

```C++
#include <iostream>
#include <vector>
#include <functional>
#include <thread>

using namespace std ; 
class Thread{
public:
	Thread(function<void () > func ) : _func(func) {} 

	thread start()
	{
		return thread(_func) ; // 在线程库的底层直接调用_func对象 
	}

private:
	function<void() > _func ; 
} ;
 
class ThreadPool {

public:
	ThreadPool() {}
	~ThreadPool() {
		for(int i = 0; i < _Pool.size() ; ++i ) 
		{
			delete _Pool[i] ; 
		}
	}
	
	// 启动所有线程
	void startPool(int size)
	{
		for(int i = 0 ; i < size ; i ++ )
		{
			_Pool.push_back(new Thread(
				bind(&ThreadPool::runInThread , this , i ) 
            // 如果参数都绑定之后，注意Thread的参数的类型是function<void()>
			 ) ) ;  
		}

		for(int i = 0 ; i < size ; ++i)
		{
			_handler.push_back(_Pool[i]->start() ) ; 
		}	
		for(thread &t : _handler )
		{
			t.join() ;  // 等待子线程执行完毕
		}
	}
private:

	vector<Thread*> _Pool ; 
	vector<thread> _handler ; 
	// 这个成员方法为线程函数 
	void runInThread(int id )
	{
		cout << "call runInThread" << id << endl ; 
	}

} ; 
int main()
{
	ThreadPool pool ;
	pool.startPool(10) ; 
	return 0 ; 
}

```

