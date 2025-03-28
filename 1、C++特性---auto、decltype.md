 

一、什么是右值、左值？
-----------

*   左值可以取地址，一般位于等号左边。
*   右值没法取地址，位于等号右边。

```cpp
int a = 8;
```

*   a可以通过&取地址，位于等号左边，所以a是左值。
*   8没法通过&取地址，位于等号右边，所以8是右值。

再举个复杂的例子：

```cpp
struct A{
    A(int a=0){
        a_ = a;
    }
    int a_;
};
A a =A();
```

*   同样的，a可以通过&取地址，位于等号左边，所以a是左值。
*   A（）是个临时值，没办法取地址，是右值。  
    可见，**有地址的变量就是左值，没有地址的字面值、临时值就是右值**。

二、什么是左值应用、[右值引用](https://so.csdn.net/so/search?q=%E5%8F%B3%E5%80%BC%E5%BC%95%E7%94%A8&spm=1001.2101.3001.7020)
--------------------------------------------------------------------------------------------------------------

### 2.1、左值引用

左值引用：能指向左值，不能指向右值的就是左值引用。

```cpp
int a = 5;
int &ref_a=a;//左值引用指向左值，编译通过
int &ref_a=5;//左值引用指向右值，编译失败
```

引用是变量的别名，**由于右值没有地址，没法被修改**，所以**左值引用无法指向右值**。  
但是const左值引用可以指向右值：

```cpp
const int& ref_a = 5;
```

const 左值引用不会修改指向值，因此可以指向右值，这也是为什么要使用const &作为[函数参数](https://so.csdn.net/so/search?q=%E5%87%BD%E6%95%B0%E5%8F%82%E6%95%B0&spm=1001.2101.3001.7020)的原因之一，如 std::vector的push\_back()：

```cpp
void push_back(const value_type& val);
```

**如果没有const，vec.push\_back(5)这样的代码就无法编译成功**。

### 2.2、右值引用

右值引用的标志是&&，顾名思义，右值引用为右值而生，可以指向右值，不能指向左值。

```cpp
int&& ref_a_right=7;//ok
int a = 8;
int &&ref_a_right = a;//编译失败，右值引用不能指向左值
ref_a_right = 2;//右值引用的用途，可以修改右值
```

### 2.3、对左右值引用本质的讨论

#### 2.3.1、右值引用有办法指向左值吗？

可以使用std::move：

```cpp
int a = 5;//a是个左值
int &ref_a_left = a;//左值引用指向左值
int &&ref_a_right = std::move(a);//通过std::move将左值转为右值，可以被右值引用指向

```

std::move是一个非常迷惑性的函数：

*   不理解左右值概念的人们往往以为它能把一个变量里的内容移动到另一个变量。**右值引用也是引用**。
*   **但事实上std::move移动不了什么**，唯一的功能是**把左值强制转换成右值**，让右值引用可以指向左值。其实现**等同于类型转换：static\_cast<T&&>（value）。所以move不会有性能提升**。
*   同样的，右值引用能指向右值，本质上也是把左值变右值了，并定义一个右值引用通过std::move()指向左值。

```cpp
int &&ref_a = 5;
ref_a = 6;
//等同于
int temp =5;
int &&ref_a = std::move(temp);
ref_a = 6;
//temp等于6
```

```cpp
int temp = 5;
int &ref_t = temp;
int &&ref_a = std::move(temp);
ref_a = 6;
cout<<&temp<<" ,"<<&ref_t<<" ,"<<&ref_a<<endl;
//一样的地址
cout<<temp<<endl;//6
```

#### 2.3.2、总结

1.  从性能上看，左右值没区别，传参使用左右值引用都可以**避免拷贝**。
2.  右值引用可以直接指向右值，也可以通过std::move指向左值；而左值引用只能指向左值（const左值引用也能指向右值）。
3.  **作为函数形参时，右值引用更灵活。虽然const左值引用也可以做到左右值都接收，但是无法修改值**。

三、右值引用和std::move使用场景
--------------------

### 3.1、右值引用优化性能，避免深拷贝

浅拷贝重复释放

```cpp
class ShallowCopy {
public:
    int* data;

    ShallowCopy(int value) : data(new int(value)) {}
    ~ShallowCopy() { delete data; }
};

int main() {
    ShallowCopy obj1(10);
    ShallowCopy obj2 = obj1;  // 浅拷贝，obj2.data 和 obj1.data 指向同一块内存

    // obj1 和 obj2 析构时都会尝试释放同一块内存，导致重复释放错误
    return 0;
}
```

深拷贝

```cpp
class DeepCopy {
public:
    int* data;

    DeepCopy(int value) : data(new int(value)) {}
    DeepCopy(const DeepCopy& other) : data(new int(*other.data)) {}  //左值引用内部 深拷贝
    ~DeepCopy() { delete data; }
};

int main() {
    DeepCopy obj1(10);
    DeepCopy obj2 = obj1;  // 深拷贝，obj2.data 是 obj1.data 的副本

    // obj1 和 obj2 析构时各自释放自己的内存，没有重复释放问题
    return 0;
}
```

移动构造函数

```cpp
class MyClass {
public:
    int* data;

    // 构造函数
    MyClass(int value) : data(new int(value)) {}

    // 移动构造函数
    MyClass(MyClass&& other) noexcept : data(other.data) {
        other.data = nullptr;  // 将源对象的指针置空
    }

    // 析构函数
    ~MyClass() { delete data; }
};

int main() {
    MyClass obj1(10);
    MyClass obj2 = std::move(obj1);  // 调用移动构造函数，obj1 的资源被移动到 obj2

    // obj1.data 现在是 nullptr，obj2.data 持有资源
    return 0;
}
```

*   **浅拷贝**：**直接复制指针**，导致重复释放问题。
*   **深拷贝**：**复制资源**，**避免重复释放**，但带来性能开销。
*   **移动构造函数**：**通过右值引用 “窃取” 资源** ，避免深拷贝，优化性能。
*   **使用move要写&&的构造**。

### 3.2、forward完美转发

forward完美转发实现了**参数在传递过程中保持其值属性的功能**，即若是左值，则传递之后仍是左值，若是右值，传递之后仍是右值。

```c++
Template<class T>
void func(T&& val);
```

根据前面所描述的，这种引用类型**既可以对左值引用，也可以对右值引用**。  
**但注意，引用之后，这个val值本质是一个左值**！  
看下面例子：

```c++
int &&a = 10;
int &&b = a;//错误
```

注意这里，a是一个右值引用，**但其本身a也有内存名字**，所以a本身是一个左值，再右值引用引用a这是不对的。  
因此我们有了std::forward()完美转发，这种T&& val中的val是左值，但如果我们用std::forward(val),就会按照参数原来的类型转发；

```cpp
int &&a =10;
int &&b = std::forward<int>(a);
```

这样是正确的。  
通过范例巩固下知识：

```cpp
#include <iostream>
#include <utility>

void target_function(int& value) {
    std::cout << "Lvalue: " << value << std::endl;
}

void target_function(int&& value) {
    std::cout << "Rvalue: " << value << std::endl;
}

template <typename T>
void wrapper(T&& arg) {
    // 使用 std::forward 保持 arg 的原始值类别
    target_function(std::forward<T>(arg));
}

int main() {
    int value = 42;

    wrapper(value);       // 调用 target_function(int&)
    wrapper(123);         // 调用 target_function(int&&)
    wrapper(std::move(value));  // 调用 target_function(int&&)

    return 0;
}
/*输出
Lvalue: 42
Rvalue: 123
Rvalue: 42
*/
```

### 3.3、emplace\_back减少内存拷贝和移动

对于STL容器，c++11后引入了emplace\_back()接口。  
emplace\_back是**就地构造**，**不用构造后再次复制到容器中**。因此效率更高。  
考虑这样的语句：

```c++
vector<string>testVec;
testVec.push_back(string(16,'a'));
```

上述语句足够简单易懂，将一个string对象添加到testVec中。底层实现：

*   首先，string(16,‘a’)会创建一个string类型的**临时对象**，这涉及到string构造过程。
*   其次，vector会**创建一个新的string对象**，这是第二次构造。
*   最后push\_back()结束时，最开始的临时对象会被析构。加在一起，这2行代码会涉及到2次string构造和一次析构。  
    C++11可以使用emplace\_back代替push\_back，emplace\_back可以直接在vector中创建一个对象，而非创建一个临时对象，再放进vector，再销毁。emplace\_back可以**省略一次构建和一次析构**，达到优化目的。

本文转自 <https://blog.csdn.net/2401_82911768/article/details/145717129?spm=1001.2014.3001.5502>，如有侵权，请联系删除。