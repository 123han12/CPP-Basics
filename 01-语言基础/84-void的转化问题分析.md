

* * *

今天看到一段代码，觉得非常有意思。

```c++
void* say_hello(void* args)
{
    cout << "Hello World！" << endl;
    return 0;
}
```

这里的返回类型竟然是`void*`。一般来说如果没有返回值，那么写一个`void`就行了，`void*`到底是什么样的存在？所以做了一些测试，总结了一些`void*`指针的用法。

**1`void *`是什么？**
-------------------

C语言中，`*`类型就是指针类型。比如 `int *p`，`double *q`，虽然是不一样的指针，但是大小却一样`sizeof(p) == sizeof(q)`，其实很容易理解，因为他们都是同一种类型`*`类型的。C语言是强类型的语言。对类型的区分十分严格。那这两个有什么不同点吗？有，+1就不同了，看下面的图：

![](https://img-blog.csdnimg.cn/img_convert/b0c4a1d07ca401008676cb3ca8d7c98c.png)


也就是对于一个指针而言，如果我们在前面规定了它的类型。那就相当于决定了它的“跳跃力”。“跳跃力”就比如说上面图中`int`跳了4个字节，但是`double`跳了8个字节。

基于这样的理解，我要对`void *`下定义了：**`void *` 是一个跳跃力未定的指针**

### 1.1 跳跃力什么时候定？

这就是它的神奇之处了，我们可以自己控制在需要的时候将它实现为需要的类型。这样的好处是：编程时候节约代码，实现泛型编程。（见下文）

2\. `void*`详解
------------

### 2.1 `void*`可以指向任何类型的地址，但是带类型的指针不能指向`void*`的地址

正常来说如果两个指针类型不一样的话，两个指针变量是不可以直接相等的。例如`int* a, float* b`，假如令`a = b`，会直接发生编译错误，而`void*`指针可以等于任何类型的指针。但是反过来不可以，也就是说一个有类型的指针不能指向一个`void*`类型的变量（哪怕此时`void*`变量已经指向了一个有类型的地址）。

```c++
float f = 5.5;
float* pf = &f;
void* pv = pf;
float* pf2 = pv;//编译错误，有类型的指针变量不能指向void*变量
```

### 2.2 `void*`指针只有强制类型转换以后才可以正常取值

```c++
int main(int argc, const char * argv[]) {
    
    float f = 5.5;
    float* pf = &f;
    void* pv;
    
    pv = pf; //这句是可以的
    
    cout<<*pv<<endl;  //编译错误，这样直接对pv取值是错误的
    cout<<*(float*)pv<<endl;  //强制类型转换后可以取值
    
    return 0;
}
```

在令`pv = pf`后，此时`pv`和`pf`指向的是同一个地址，值相同，但是两者的类型是不一样的。`pf`作为浮点型指针，是可以直接取到浮点数的，**但是`pv`必须要强制类型转换以后才可以取值，也就是说一个`void*`的指针必须要经过强制类型转换以后才有意义。**

```c++
int main(int argc, const char * argv[]) {
    
    float f = 5.5;
    float* pf = &f;
    void* pv;
 
    pv = pf;
   
    cout<<*(float*)pv<<endl;  //强制类型转换后可以取值,值为5.5
    cout<<*(int*)pv<<endl; //强制类型转换，值为1085276160
    cout<<(int)(*(float*)pv)<<endl;//取值后再次类型转换，值为5
    
    return 0;
}
```

如果把一个指向`float*`的值的`void*`指针，强制转换成`int*`也是不对的。也就是说地址**保存了什么样的变量，就要转化成哪种类型的指针**，否则就会出错。

### 2.3 `void*`指针变量和普通指针一样可以通过等于0或者NULL来初始化，表示一个空指针

```c++
void* pv = 0; 
void* pv2 = NULL;
cout<<pv <<endl; //值为0x0
cout<<pv2<<endl; //值为0x0
```

### **2.4 当`void*`指针作为函数的输入和输出时，表示可以接受任意类型的输入指针和输出任意类型的指针**

```c++
void* test(void* a)
{
    return a;
}
 
int main() {
 
    static int a = 5;
    int* pi = &a;
    
    cout<<pi<<endl;              //值为0x100001060
    cout<<test(pi)<<endl;        //值为0x100001060
    cout<<test((void*)pi)<<endl; //值为0x100001060
}
```

如果函数的输入类型为void\*，在调用时由于是值传递，所以函数实际接收到的应该就是一个地址值。这个值可以是任意类型。

```c++
int a = 5;
int* pi = &a;
 
void* test()
{
    return pi; 
}
 
int main() {
    cout<<test()<<endl;        //值为0x100001060
}
```

输出时同样也是值传递，因此可以输出任意类型指针指向的地址。

### 2.5 和`void`的区别

再让我们回头看初始的那段函数：

```c++
//返回了一个空指针
void* say_hello(void* args)
{
    cout << "Hello World！" << endl;
    return 0;
}
 
//没有返回值
void say_hello(void* args)
{
    cout << "Hello World！" << endl;
 　　return;
}
```

其实两个函数实现的内容是一样的。但是`void*`返回类型的函数返回了一个空指针，而void型没有返回值。

3. 作用及典型应用
----------

### 3.1 函数传参时不确定类型，或者要支持多类型的传参；

```c++
void function(int dataType, void* data) {
 
    // 根据dataType的不同值，进行不同的转换
    switch (dataType) {
 
        case 0:
            int* a = (int*)data;
 
        case 1:
            char* a = (char*)data;
 
        ...
    }
}
```

### 3.2 当函数的返回值不考虑类型指关心大小的时候

```c++
void * memcpy(void *dest, const void *src, size_t len);
void * memset ( void * buffer, int c, size_t num );
```

`memcpy`和`memset`对外接收任何类型的指针，这样是合理并且必要的，因为这是内存操作函数，是对bit进行操作的，考虑数据类型是没有任何意义的。

```c++
int *a=NULL；
 
a=(int*)malloc(sizeof(int));//返回的是void*，所以赋值给其他指针类型要强转一下
```

同样的，`malloc`函数只关注你要多大的内存，你需要把它怎么划分是你的事情，但是你需要显式的表明你是怎么划分的。这里语法要求是必须的，`void *`类型转为其他类型必须[强制类型转换](https://so.csdn.net/so/search?q=%E5%BC%BA%E5%88%B6%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2&spm=1001.2101.3001.7020)。

4\. 总结：
-------

#### 4.1 `void*`类型的指针其实本质就是一个过渡型的指针状态，必须要赋予类型（强制类型转换）才能正常使用。

`void *`的范围较大，所以强制转换，使其进行范围缩小。

![](https://img-blog.csdnimg.cn/img_convert/073091e717e7a3d6ed2ad5b9770cf841.png)

#### 4.2 只能单向类型转换。`void*`可以转化成其他类型，但是有类型的不能转化成`void*`。

任何类型的指针都可以直接赋值给`void *`型指针，无需进行强制类型转换，相当于`void *`包含了其他类型的指针。

#### 4.3 在函数调用过程中的使用作为输入输出参数也非常好用，可以灵活使用任意类型的指针，避免只能使用固定类型的指针。