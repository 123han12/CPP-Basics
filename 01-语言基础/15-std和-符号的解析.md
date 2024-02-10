`->`的返回值的限定:
和大多数其他运算符一样(尽管这么做不太好)，我们能令`operator*`完成任何我们指定的操作。换句话说，我们可以让 `operator*`返回一个固定值 42，或者打印对象的内容，或者其他。==箭头运算符则不是这样==，==它永远不能丢掉成员访问这个最基本的含义。==当我们重载箭头时，**可以改变的是箭头从哪个对象当中获取成员，而箭头获取成员这一事实则永远不变**。
对于形如 `point->mem` 的表达式来说，**point 必须是指向类对象的指针或者是一个重载了 operator->的类的对象**。根据`point`类型的不同，`point->mem` 分别等价于

```C++
(*point).mem;  // point 是一个内置的指针类型
point.operator()->mem; // point 是类的一个对象  [注:这个是C++ primer中的写法，有点奇怪]
```

除此之外，代码都将发生错误。`point->mem` 的执行过程如下所示:

+ 如果 `point` 是指针,则我们应用内置的箭头运算符,表达式等价于`(*point).mem`。首先解引用该指针，然后从所得的对象中获取指定的成员。如果 `point` 所指的类型没有名为`mem`的成员，程序会发生错误。
+ 如果`point `是定义了`operator->`的类的一个对象,则我们使用`point.operator->()`的结果来获取 mem。其中，如果该结果是一个指针，则执行第1步;如果该结果本身含有重载的` operator->()`，则重复调用当前步骤。最终，当这一过程结束时程序或者返回了所需的内容，或者返回一些表示程序错误的信息。

**重载的箭头运算符必须返回类的指针或者自定义了箭头运算符的某个类的对象**

```C++
#include <iostream>

using namespace std;
namespace A {
    int a = 100;
    namespace B            //嵌套一个命名空间B
    {
        int a = 20;
    }
}

int a = 200;//定义一个全局变量


int main(int argc, char *argv[]) {
    cout << "A::a =" << A::a << endl;        //A::a =100
    cout << "A::B::a =" << A::B::a << endl;  //A::B::a =20
    cout << "a =" << a << endl;              //a =200
    cout << "::a =" << ::a << endl;          //::a =200

    using namespace A;
    cout << "a =" << a << endl;     // Reference to 'a' is ambiguous // 命名空间冲突，编译期错误
    cout << "::a =" << ::a << endl; //::a =200

    int a = 30;
    cout << "a =" << a << endl;     //a =30
    cout << "::a =" << ::a << endl; //::a =200

    //即：全局变量 a 表达为 ::a，用于当有同名的局部变量时来区别两者。

    using namespace A;
    cout << "a =" << a << endl;     // a =30  // 当有本地同名变量后，优先使用本地，冲突解除
    cout << "::a =" << ::a << endl; //::a =200


    return 0;
}
```









