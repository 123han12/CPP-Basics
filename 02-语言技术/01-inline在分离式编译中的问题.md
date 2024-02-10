`func.cpp`

```c++
#include "func.h"
#include <iostream>
inline void test::func()
{
    std::cout << "hello world" << std::endl ; 
} 
```

`func.h`

```c++
class test{
public:
    test(){} ; 
    inline void func() ;  
};
```

`test2Main.cpp`

```c++
#include "test2.h"

int main()
{
    test obj ; 
    obj.func() ; 
    return 0 ; 
}
```

上述文件存在的问题：

+ 有两个编译单元: `test2Main.cpp` 和`func.cpp` , 在编译前者的时候，因为`func()`是声明为内联的函数了，编译器必须在此编译单元内可以看到函数的定义，从而判断是否真正的内联，但`func`的定义在于`func.cpp`中，在编译`test2Main.cpp`的时候是不可见的，所以这里在编译的时候会出现问题。

**谨记**：

如果在类体外定义`inline`函数，==则必须将类定义和成员函数的定义都放在同一个头文件中(或者写在同一个源文件中)==，否则编译时无法进行置换(将函数代码的拷贝嵌入到函数调用点)。但是这样做，不利于类的接口与类的实现分离，不利于信息隐蔽。虽然程序的执行效率提高了，但从软件工程质量的角度来看，这样做并不是好的办法。==只有在类外定义的成员函数规模很小而调用频率较高时，才将此成员函数指定为内置函数。==

==正确写法：==

`func.hpp`

```c++
#include <iostream>
class test{
public:
    test(){} ; 
    inline void func() ;  
};
inline void test::func()
{
    std::cout << "hello world" << std::endl ; 
} 
```

`test.cpp`

```c++
#include "test2.h"

int main()
{
    test obj ; 
    obj.func() ; 
    return 0 ; 
}
```

