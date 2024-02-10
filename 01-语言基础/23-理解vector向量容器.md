### 理解`vector`容器



**这一节直接是算法的理解**

**模拟简易的`STL vector` 容器**

```C++
#include <iostream>
using namespace std ;

template<typename T>
class vector {
public:
   vector(int size = 10 )
   :  _first(new T[size]) ,
      _last(_first)  , 
      _end_of_storage(_first + size )  
   {}
   
   ~vector()
   {
      delete[] _first ; 
      _first = _last = _end_of_storage = nullptr ; 
   }

   vector(const vector &rhs )
   {
      int size = rhs._end_of_storage - rhs._first ;  // ****
      _first = new T[size] ; 
      int len = rhs._last - rhs._first ; // 有效元素的个数
      for(int i = 0 ; i < len ; ++ i )
      {
         _first[i] = rhs._first[i] ; 
      }
      _last = _first + len ; 
      _end_of_storage = _first + size ; 
   
   }

   vector& operator=(const vector& rhs )
   {
      if(this == &rhs ) return *this ; 
      delete []_first ; 

      int size = rhs._end_of_storage - rhs._first ;  // ****
      _first = new T[size] ; 
      int len = rhs._last - rhs._first ; // 有效元素的个数
      for(int i = 0 ; i < len ; ++ i )
      {
         _first[i] = rhs._first[i] ; 
      }
      _last = _first + len ; 
      _end_of_storage = _first + size ; 

   }
   void push_back(const T &val )
   {
      if(full() ) expand() ;
      *_last++ = val ; 
   }
   void pop_back() // 从容器末尾删除元素
   {
      if(empty() ) return ; 
      --_last ; 
   }

   T back() const {
      return *(_last - 1 ) ; 
   }

   bool full() const { return _last == _end_of_storage ; }
   bool empty() const {return _last == _first ; }

   int size() const {return _last - _first ;} ; 

private:   
   T* _first ;
   T* _last ; 
   T* _end_of_storage ; 
   void expand() 
   {
      int size = _end_of_storage - _first ; 
      T* newptr = new T[size * 2 ] ;
      int index = 0 ; 
      for(T* iter = _first ; iter != _last ; ++iter , ++index )
      {
         newptr[index] = *iter ;  
      }
      delete [] _first ; 
      _first = newptr ; 
      _last = _first + index ; 
      _end_of_storage = _first + 2 * size ; 
   }
} ;  
int main()
{
   vector<int> vec ; 
   for(int i = 0 ; i < 100 ; ++ i)
   {
      vec.push_back(rand() % 100 ) ; 
   }
   while(!vec.empty())
   {
      cout << vec.back() << endl ; 
      vec.pop_back() ; 
   }
   return 0 ;  
}
```

**上述版本还有很大的问题，具体分析请看下一篇问章**



