

添加`const`的限制的原因是预防因为35行错误代码所带来的潜在的危害: 如果不加上`const`  35行代码不会报错，这种错误很难排查，所以需要添加`const`限制。

```c++
// effective c++ 条款21测试代码
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <iostream>
using namespace std;
class Complex{
public:
	Complex(int f , int s ) : a(f) , b(s) {}
	Complex& operator=(const Complex&) = default ;
		Complex(const Complex&) = default ; 	
	friend const Complex operator*(const Complex& lhs , const Complex& rhs) ;
		operator bool() const {
			return true ;
		}	       
private:
		int a ;
		int b ; 
};

const  Complex operator*(const Complex& lhs , const Complex& rhs)
			{
		Complex res(0 , 0 )  ; 
					res.a = lhs.a + rhs.a ;
					res.b = lhs.b + rhs.b ;
		return res ; 
			}

int main()
{
	Complex a(1 , 2 ) , b(3 , 4 ) ;
	Complex c(1 , 2 ) ;
	a * b = c ; 
	if(  a*b = c )   // 这种代码虽然足够智障，但仍需要我们去注意
	{

	cout << "hello world " << endl ; 	

	} 
return 0 ;
}

```

