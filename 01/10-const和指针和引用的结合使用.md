### `const `， 一级指针，引用的结合应用

+ 问题：写一句代码，在内存`0x0018ff44` 处写一个4字节的10。

  > ```C++
  > int main()
  > {
  >     int* ptr = (int*)0x0018ff44 ; // 先将数字转换为地址，之后将这个地址给指针p。
  >     
  >     
  >     //int* &p = (int*)0x0018ff4 ; // 错误的。
  > 
  >     int* &&p = (int*)0x0018ff44 ; // 是一个右值引用，正确。 
  >     
  >     int* const &p1 = (int*)0x0018ff44  // 正确，可以使用一个 常引用  来用  立即数 初始化。
  >     
  >     // 不要写成： const int* &p1 = (int*)0x0018ff44 ， 这个时候 const修饰的是 *p1 ，引用不是常引用会引用。 
  >     
  >     
  >     return 0 ; 
  > }
  > ```



+ 问题：如何快速辨别下列代码是否存在问题

  > ```C++
  > int main()
  > {
  >     int a = 10 ; 
  >     int *p = &a ; 
  >     int *&q = p ; 
  > 	return 0; 
  > }
  > ```
  >
  > **可以将引用还原为指针进行辨析[规则：左边的`&`变为`*`,右边添加`*`],上述代码相当于：**
  >
  > ```C++
  > int main()
  > {
  >     int a = 10 ; 
  >     
  >     int *p = &a ; 
  >     
  >     int **p = &p ;     // int** <= int** 没毛病。
  > 	
  >     return 0; 
  > }
  > ```
  >
  >  

+ 问题：如何判断下列代码的对错

  > ```C++
  > int main()
  > {
  >     int a = 10 ; 
  >     int *p = &a ; 
  >     const int *&q = p ; 
  > 	return 0; 
  > }
  > ```
  >
  > **可以将引用还原为指针进行辨析[规则：左边的`&`变为`*`,右边添加`*`],上述代码相当于：**
  >
  > ```C++
  > int main()
  > {
  >     int a = 10 ; 
  >     int *p = &a ; 
  >     const int **q = &p ;     //   const int** <== int** , 错误。 
  > 	return 0; 
  > }
  > ```









+ 三者结合的错误写法：

  ```C++
  // 错误1
  int a = 10 ; 
  int *const p = &a ; 
  int *&q = p ;     // 相当于 int**q  <= int *const* , 将一个 常量指针的地址泄漏给了普通的二级指针，如果 *q被修改
  // 就是违法修改内存了，错误。
  
  
  // 错误2
  int a = 10 ;
  const int * p = &a ; 
  int *&q = p ;    
  /*
  最后一句相当于：
  int **q = &p ;     int** <= const int**    错误。
  */
  
  
  // 错误3
  int a = 10 ; 
  int *p = &a ; 
  const int *&q = p ;      //  const int ** <= int** 错误
  const int **q = &p ;     // const int** <= int ** 错误
  ```

  





