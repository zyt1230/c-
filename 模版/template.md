# 一、模版template

- 模版是在**编译阶段**进行的。一般**不会自己转换类型**。

- 首先要了解

```cpp
template<class T>
//可以写成
template<typename T>
//一般使用class
```

- 模版的 2种 使用方法

```cpp
#include <iostream>
#include <string>
#include<fstream>
using namespace std;

template <typename T>//声明一个模版,告诉编译器后面代码中就跟着的T不要报错 ,T是一个通用的数据类型
void mySwap(T& a, T& b) {
	T temp = a;
	a = b;
	b = temp;
}

int main() {
	int a = 10, b = 20;
	//利用函数模版
	//方法1.自动类型推导
	mySwap(a,b);
    //方法2.显示指定类型
	mySwap<int>(a, b);
	cout<<"a="<<a<< endl;
	cout << "b=" << b << endl;
	return 0;
}
```

## 1、函数模版

- 自动推导：**必须推导出来的类型是一样的才能用**





## 2、类模版

- 必须指定类模版的类型 class < T类型 >

```cpp
#include<iostream>
#include<string>
using namespace std;
template <class T,class U>
class Test{
public:
    Test(T t,U u):t(t),u(u){
    }
public:
    T t;
    U u;
};
int main(int argv,char* argc[]){
    Test<int,int>test(1,3);
    
}
```

## 3、模版别名

- typedef

```cpp
#include<iostream>
using namespace std;
template <class T,class T1>
class Pair{
public:
Pair(){}
void a(T &x){
    cout<<x<<endl;
}
};

int main(){
    typedef Pair<int,int> t;
    t p;
    int a=0;
    p.a(a);
}
```

- using

```cpp
#include<iostream>
using namespace std;
template <class T,class T1>
class Pair{
public:
Pair(){}
void a(T &x){
    cout<<x<<endl;
}
};
template <class T,class T1>
using t = Pair<T,T1>;
int main(){

    t<int,int> p;
    int a=0;
    p.a(a);
}
```
