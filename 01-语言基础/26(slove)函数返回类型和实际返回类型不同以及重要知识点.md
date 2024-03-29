### 函数返回类型和实际返回类型有事不是精准匹配的疑惑



**前置知识**

> **求值顺序**
>
> 优先级规定了运算对象的**组合方式**，但是没有说明运算对象按照什么顺序求值。在大多数情况下，不会明确指定求值的顺序。对于如下的表达式
>
> ```C++
> int i = f1() * f2() ; 
> ```
>
> ​	我们知道 `f1` 和`f2`一定会在执行乘法之前被调用，因为毕竟相乘的是这两个函数的返回值。但是我们无法知道到底`f1`在 `f2`之前调用还是`f2`在`f1`之前调用。
>  	对于那些**没有指定执行顺序的运算符**来说，如果**表达式指向并修改了同一个对象**，将会引发错误并产生未定义的行为。举个例子:  `<<` 运算符没有明确规定何时以及如何对运算对象求值，因此下面的输出表达式是未定义的:
>
> ```C++
> int i = 0;
> cout << i << " "<< ++i << endl ;//未定义的
> ```
>
> 因为**程序是未定义的，所以我们无法推断它的行为**。编译器可能先求`++i` 的值再求`i`的值，此时输出结果是`1 1`;也可能先求`i`的值再求`++i` 的值，输出结果是`0 1` :**甚至编译器还可能做完全不同的操作**。因为此表达式的行为不可预知，因此不论编译器生成什么样的代码程序都是错误的。
>
> + **有4种运算符明确规定了运算对象的求值顺序**
>   + `&&`运算符：先求左侧运算对象的值，只有当左侧运算对象的值为`true`时才继续求右侧运算对象的值。
>   + `|` 运算符：先求左侧运算对象的值，只有当左侧运算对象的值为`false`时才继续求右侧运算对象的值。
>   + `?:`运算符：相当于按照`if else`的顺序求值
>   + `,` 运算符： 从左到右依次求值。



+ 函数调用需要完成的工作
  + 用实参初始化函数对应的形参
  + 将控制权转移给被调用函数，此时**主调函数**的执行被暂时中断，**被调函数**开始执行。



+ `return`语句完成的工作
  + :一是返回 return 语句中的值(如果有的话) ， 返回值用于**初始化调用表达式**的结果
  + 二是将控制权从被调函数转移回主调函数



==**尽管实参与形参存在对应关系，但是并没有规定实参的求值顺序。编译器能以任意可行的顺序对实参求值**==



+ 函数的形参列表为空的两种形式

  ```C++
  void f1() {}   // 隐式地定义空形参列表
  void f2(void) {} // 显式地定义空形参列表
  ```

+ 形参是一种自动对象，函数开始时为形参申请存储空间，因为形参定义在函数体作用域内，所以函数一旦终止，形参也就被销毁了





### 局部静态对象

> 某些时候，**有必要令局部变量的生命周期贯穿函数调用及之后的时间**。可以将局部变量定义成`static`类型从而获得这样的对象。局部静态对象`(localstatic object)`在程序的执行路径**第一次经过对象定义语句时初始化**，并且**直到程序终止才被销毁**，==在此期间即使对象所在的函数结束执行也不会对它有影响。==
> 举个例子:
>
> ```C++
> size_t count_calls()
> {
>    static size_t ctr = 0 ;//调用结束后，这个值仍然有效
>    
>    return ++ctr;
> }
> int main()
> {
>    for (size_t i = 0 ; i != 10 ; ++i )
>    {
>       cout << count_calls() << endl ;
>    } 
>    return 0 ; 
> }
>    
> ```
>
> 在控制流第一次经过 `ctr` 的定义之前，`ctr` 被创建并初始化为 0。每次调用将 ctr加1并返回新值。每次执行` count_calls` 函数时，变量 `ctr` 的值都已经存在并且等于函数上一次退出时 `ctr`的值。因此，第二次调用时 `ctr` 的值是 1，第三次调用时 `ctr`的值是 2，以此类推。
> 如果局部静态变量没有显式的初始值，它将执行**值初始化**(参见 3.3.1 节，第88 页),==内置类型的局部静态变量初始化为 0==。













