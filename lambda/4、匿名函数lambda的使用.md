 

一、格式
----

```cpp
[捕获列表](参数列表)mutable(可选) 异常属性->返回类型{
//函数体
}

```

*   捕获列表是可以自动捕获参数的。
*   一般编译器可以自己推断出返回类型，所以可以不用写。 如下:

```cpp
[捕获列表](参数列表){函数体}
```

二、捕获列表
------

### 1、按值和引用传递

*   捕获列表是可以自动捕获参数的。

```cpp
void test(){
    int c =12;
    int d =30;
    int e = 10;
    auto add =[c,d,&e](int a,int b)->int{
        cout<<"d="<<d<<endl;
        cout<<"e="<<e<<endl;
        return c;
    };
    d=20;
    e=20;
    cout<<add(1,2)<<endl;
}
int main(){
    test();
    //d=30
    //e=20
    //12
}

```

### 2、隐式捕获

*   **手动写捕获列表有时候是非常复杂的**，这种机械的工作可以交给编译器来处理，这时候可以在捕获列表中写一个\*\*&或=**向编译器声明采用**引用捕获或者值捕获\*\*。
*   \[=\]就可以使用外面的变量了，但是**是const类型**。 \[&\]也可以使用外面的变量。

```cpp
void test(){
    int c =12;
    int d =30;
    auto add =[&](int a,int b)->int{
        c=a;
        cout<<"d="<<d<<endl;
        return c;
    };
    d=20;
    cout<<add(1,2)<<endl;
    cout<<"c="<<c<<endl;
}
int main(){
    test();
    //d=20
    //1
    //c=1
}

```

### 3、空捕获列表

捕获列表\[\]中为空，表示**lambda不能使用所在函数中的变量**。

```cpp
void test(){
    int c =12;
    int d =30;
    auto add =[](int a,int b)->int{
        c=a;//编译错误
        cout<<"d="<<d<<endl;//编译错误
        return c;//编译错误
    };
    d=20;
    cout<<add(1,2)<<endl;
    cout<<"c="<<c<<endl;
}

```

### 4、表达式捕获

以上3种捕获都是**左值捕获**，如果要**右值捕获**使用的是 **c++14** 。

```cpp
void test{
    auto important = make_unique<int>(1);
    auto add=[v1=1,v2=std::move(important)](int x,int y)->int{
        return x+y+v1+(*v2);
    };
}

```

### 5、[泛型](https://so.csdn.net/so/search?q=%E6%B3%9B%E5%9E%8B&spm=1001.2101.3001.7020)lambda

在**C++14之前**，**lambda表示的形参只能指定具体的类型，没法泛型化**。C++14后，[lambda函数](https://so.csdn.net/so/search?q=lambda%E5%87%BD%E6%95%B0&spm=1001.2101.3001.7020)的形式参数可以使用**auto关键字**来产生意义上的泛型：

```cpp
void test(){
    auto add = [](auto x,auto y){
        return x+y;
    };
}

```

### 6、可变lambda

*   采用值捕获的方式，lambda不能修改其值，如果要修改，使用mutable修饰。
*   采用引用捕获的方式，lambda可以直接修改其值。

本文转自 <https://blog.csdn.net/2401_82911768/article/details/146418811?spm=1001.2014.3001.5501>，如有侵权，请联系删除。