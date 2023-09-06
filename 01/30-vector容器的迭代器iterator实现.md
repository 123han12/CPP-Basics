### `vector`容器的迭代器`iterator` 的实现



### 迭代器实现思路

```C++
class iterator{
public:
    iterator(T* ptr = nullptr )
    : _ptr(ptr ){}
    bool operator!= (const iterator&  it ) const 
    {
        return _ptr == it._ptr ; 
    }
    void operator++()
    {
        ++_ptr ; 
    }

    T& operator*() {return *_ptr ; } 
    const T& operator*() const {return *_ptr ; } // 只读
private:
    T* _ptr ; 

} ;

iterator begin() {
    return iterator(_first) ; 
}
iterator end()
{
    return iterator(_last) ; 
}
```

> 对于底层结构是数组形式的容器我们提供`[]`运算符的重载从而实现**随机存取**效率是比较高的，但有的容器却没有提供`[]`重载运算符是因为其底层数据结构**随机存取**的效率比较低，所以直接砍掉了这种访问方式。

`vectorHead.h`

```C++
#ifndef MY_STRING
#define MY_VECTOR
#include <iostream>
#include <cstring> 
template<typename T> 
struct Allocator{
   // 分配内存
   T* allocate( size_t n ) 
   {
      T* ptr = (T*)malloc( n * sizeof(T) ) ; 
      return ptr ; 
   }
   // 构造对象 , 使用  定位new
   void construct(T *p , const T& val)
   {
      new (p)T(val) ;     
   }
   
   // 析构对象
   void destroy(T *p ) 
   {
      p->~T(); 
   }

  	// 释放内存 
	void deallocate(void *p ) // 这里为什么要使用  void*  
   {
      free(p) ; 
   }
   
} ;
template<typename T , typename Alloc = Allocator<T> >
class vector {
public:
   vector(int size = 10 )
   {
      _first = alloc.allocate(size) ; 
      _last = _first ;
      _end_of_storage = _first + size ; 
   }
   ~vector()
   {
      for(T* iter = _first ; iter != _last ; ++iter )
      {
         alloc.destroy(iter ) ; 
      }
      alloc.deallocate(_first )  ;

   }

   vector(const vector &rhs )
   {
      int size = rhs._end_of_storage - rhs._first ;  // ****
     
      _first = alloc.allocate(size) ; 
      int len = rhs._last - rhs._first ; // 有效元素的个数
      for(int i = 0 ; i < len ; ++ i )
      {
         alloc.construct(_first + i , rhs[i] ) ; 
      }
      _last = _first + len ; 
      _end_of_storage = _first + size ; 
   
   }

   vector& operator=(const vector& rhs )
   {
      if(this == &rhs ) return *this ; 

      //delete[]_first;
		for (T *p = _first; p != _last; ++p)
		{
			alloc.destroy(p); // 把_first指针指向的数组的有效元素进行析构操作
		}
		alloc.deallocate(_first) ;  

       int size = rhs._end_of_storage - rhs._first ;  // ****
     
      _first = alloc.allocate(size) ; 
      int len = rhs._last - rhs._first ; // 有效元素的个数
      for(int i = 0 ; i < len ; ++ i )
      {
         alloc.construct(_first + i , rhs[i] ) ; 
      }
      _last = _first + len ; 
      _end_of_storage = _first + size ; 
   }
   void push_back(const T &val )
   {
      if(full() ) expand() ;
      alloc.construct(_last , val ) ; 
      _last ++ ; 
   }
   void pop_back() // 从容器末尾删除元素
   {
      if(empty() ) return ; 
      alloc.destroy(--_last ) ; 
   }

   T back() const {
      return *(_last - 1 ) ; 
   }

   bool full() const { return _last == _end_of_storage ; }
   bool empty() const {return _last == _first ; }
   int size() const {return _last - _first ;} 


   T& operator[](int index )
   {
        if(index < 0 || index >= size() ) 
        {
            throw index + "OutOfRangeException" ; 
        }
        return _first[index] ; 
   }

   //迭代器一般实现为容器的嵌套类型
class iterator{
public:
    iterator(T* ptr = nullptr )
    : _ptr(ptr ){}
    bool operator!= (const iterator&  it ) const 
    {
        return _ptr == it._ptr ; 
    }
    void operator++()
    {
        ++_ptr ; 
    }

    T& operator*() {return *_ptr ; } 
    const T& operator*() const {return *_ptr ; } // 只读
private:
    T* _ptr ; 

} ;

iterator begin() {
    return iterator(_first) ; 
}
iterator end()
{
    return iterator(_last) ; 
}
private:   
   T* _first ;
   T* _last ; 
   T* _end_of_storage ; 
   Alloc alloc ; 
   void expand() 
   {
      int size = _end_of_storage - _first ; 
      T* newptr = alloc.allocate(size * 2 ) ; 
      int index = 0 ; 
      for(T* iter = _first ; iter != _last ; ++iter , ++index )
      {
         alloc.construct(iter , newptr[index] )   ; 
      }
      
      //delete [] _first ;
      for(T* iter = _first ; iter != _last ; ++iter ) 
      {
         alloc.destroy(iter ) ; 
      }

      _first = newptr ; 
      _last = _first + index ; 
      _end_of_storage = _first + 2 * size ; 
   }
} ;  
class Test{
public:
   Test() {std::cout << "Test()" << std::endl ; }
   Test(const Test& val ) {
      std::cout << "copy construct" << std::endl ; 
   }
   ~Test() {std::cout << "~Test()" << std::endl ; }
} ; 
```



`main.cc`

```C++
#include <iostream>
#include <string>
#include "vectorHead.h"

int main()
{
   vector<int> vec(200);
	for (int i = 0; i < 20 ; ++i)
	{
		vec.push_back(rand() % 100 + 1);
	}
   
   int size = vec.size();
	for (int i = 0; i < size; ++i)
	{
		cout << vec[i] << " ";  // 有的容器不支持这种写法。
	}
	cout << endl ; 

	auto it = vec.begin();
	for (; it != vec.end(); ++it)
	{
		cout << *it << " ";
	}
	cout << endl;

	for (int v : vec)
	{
		cout << v << " ";
	}
	cout << endl;


   return 0 ;  
}
```

 