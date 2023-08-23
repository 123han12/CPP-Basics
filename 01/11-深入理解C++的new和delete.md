### 深入理解`C++`的new和delete 。 



+ **`malloc`和`free` 称作`C`的库函数 ， `new` 和`delete` 称作运算符。**

+ **涉及到动态内存的分配和释放一律使用`new` 和`delete`** . 

+ 开辟变量：

```C++
int main()
{
    int *p = (int*)malloc(sizeof(int)) ;   // 返回类型为void(*) ，开辟内存的单位为字节。 不会初始化
    if(p == nullptr ) return -1 ;  // 判断是否开辟失败。
    *p = 20 ; 
    free(p) ;    // 释放指定的空间的地址。
    
    
    // 对于可能抛出异常的代码，放入try语句块中
    try{
    	int *p1 = new int(20) ;  // 指定类型为int 初始值为20 。初始化了内存。
    }
    catch(const bad_alloc &e ) //  对应的异常 ， 使用的时候需要包含指定的头文件。
    {
        // 处理方式
    }
    
    delete p1 ; // 释放内存。
    
    return 0 ; 
}
```



+ 开辟数组内存：

```C++
int main()
{
    int *q = (int* ) malloc(sizeof(int) * 20) ; 
    if(q == nullptr ) return -1 ; // 数组内存开辟失败。
    free(q) ; // 释放数组内存。
    
    int* q1 = new int[20]  ; // 开辟20个整型,不负责初始化。
    int* q2 = new int[20]() ; // 开辟20个整型,负责初始化所有元素为0
    
    
    // 释放数组内存。 
    delete []q1 ; 
    
    
    
    
    return 0 ; 
}
```













+ `new`和`malloc` 的区别

  > + new不仅可以做内存开辟，还可以做内存初始化操作
  > + malloc开辟内存失败，是通过返回值和nullptr做比较；而new开辟内存失败，是通过抛出**bad_alloc**类型的异常来判断的。



+ `delete` 和`free`的区别

  > 删除内存的形式不同。





+ 问： `new` 的类型有哪几种。

  ```C++
  // 主要有四种。
  int main()
  {
      // 普通 new 
      int *p1 = new int(20) ;       
      
      int *p2 = new (nothrow) int ;  // 不抛出异常的new 
      
      // 常量new
      const int *p3 = new const int(40) ;  // 在堆上开辟一个常量内存。
      
      // 定位new , 在指定地址上开辟内存,并进行初始化。
      int data = 0 ; 
      int *p4 = new (&data) int(50) ; 
      
      
      return 0 ; 
      
  }
  ```

  