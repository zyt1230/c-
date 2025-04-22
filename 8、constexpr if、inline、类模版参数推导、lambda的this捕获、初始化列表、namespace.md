 

一、constexpr if：编译时条件分支
----------------------

*   作用：**在模板编程中**，根据**条件在编译时选择不同的代码路径**，无需特化版本或复杂SFINAE技巧\[替代SFINAE\]。\[SFINAE将在模版元编程再讲。下个月了。\]

> 注意：默认使用了隐式inline

*   基本语法

```cpp
if constexpr (condition) {
    // 如果 condition 为 true，编译这部分
} else {
    // 如果 condition 为 false，编译这部分（可选）
}
```

*   condition 必须是编译时**可求值的常量表达式**（如 constexpr 变量、模板参数、sizeof 等）
*   **关键区别**：与普通 if 不同，if constexpr **在编译时直接丢弃未选择的分支**，不会检查语法有效性。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6f552cf710ab49c3b13e1354d9d3f671.png)

二、inline变量：头文件中的全局/静态变量定义
-------------------------

*   作用：允许在**头文件**中**定义全局变量或类的静态成员变量**，避免多次定义的链接错误。
*   inline可以减少函数调用开销，提高性能。

> 注意：在**类定义内部**直接实现的**成员函数**（如头文件中的类方法）**默认是 inline** 的，无需显式声明。

*   c++17前，只能在头文件中声明后，cpp文件中使用。
    *   extern可以用来告知已经定义过这个变量了。[推荐这篇文章](https://www.zhihu.com/question/270847649)
*   例子（c++17前的static 变量，需要再类的外面定义，类里面声明）

```cpp
// MyClass.h
class MyClass {
public:
    static int sharedValue; // 头文件中声明
};

// MyClass.cpp
int MyClass::sharedValue = 10; // 必须在一个.cpp文件中定义

```

*   而c++17后，就只需要在static变量前面加inline，就可以定义了。

```cpp
// MyClass.h (C++17后)
class MyClass {
public:
    inline static int sharedValue = 10; // 直接初始化，无需外部定义
};

// 使用：多个.cpp文件可以安全包含此头文件

```

三、类模版参数推导
---------

*   作用：编译器根据构造函数参数**自动推导类模板类型**，简化代码。

```cpp
#include <vector>
#include <tuple>

// 标准库的CTAD：无需显式模板参数
std::pair p(1, 3.14);        // 推导为 std::pair<int, double>
std::vector v{1, 2, 3};      // 推导为 std::vector<int>

// 自定义类
template <typename T, typename U>
struct MyPair {
    T first;
    U second;
    MyPair(T f, U s) : first(f), second(s) {}
};

// 使用CTAD
MyPair mp(5, "hello");  // 推导为 MyPair<int, const char*>

// 若构造函数无法推断类型，可添加推导指引：
template <typename T>
MyPair(T, T) -> MyPair<T, T>; // 处理MyPair(2,3)到MyPair<int, int>的推导

```

### 补充：auto占位的非类型模版形参

*   **auto占位的非类型模版形参**

```cpp
template<auto T>
void func1(){
    cout<<T<<endl;
}
int main(){
    func1<100>();//100
    return 0;
}
```

四、lambda的this捕获\[\*this\]
-------------------------

*   作用：按值捕获当前对象的副本，避免**捕获this指针可能导致的对象销毁后的悬垂引用**。

> **悬垂引用**是指**引用了一个已经被销毁或无效的内存的引用变量**。

```cpp
// C++17后：按值捕获对象副本，安全
#include <iostream>

class Worker {
public:
    int data = 42;
    void start() {
        // C++17前：按引用捕获this，危险！（若对象销毁后lambda还在运行）
        auto lambda_old = [this]() { 
            std::cout << data << "\n"; // 可能的悬垂引用
        };

        // C++17后：按值捕获对象副本，安全
        auto lambda_new = [*this]() mutable { 
            data++; // 操作的是副本的data
            std::cout << data << "\n"; // 输出43
        };

    }
};

```

五、初始化列表
-------

### 1、c++11

*   统一初始化语法：允许用花括号 {} 初始化几乎所有类型的对象，避免旧的 () 和 {} 混乱。

```cpp
// 常见初始化方式
int arr[] = {1, 2, 3};
std::vector<int> vec = {4, 5, 6};

// 聚合类（没有自定义构造函数、无私有成员等）
struct Point { int x; int y; };
Point p = {10, 20}; // C++11允许聚合初始化

// 构造函数使用初始化列表
class Widget {
public:
    Widget(std::initializer_list<int> list) { /*...*/ }
};
Widget w{1, 2, 3}; // 调用初始化列表构造函数

```

*   注意特性：
    *   **禁止窄化转换**（如 double → int）：int x{3.14}; 会报错。
    *   **解决构造函数歧义**：优先匹配 std::initializer\_list 构造函数。
    *   **支持聚合类型初始化**：简化结构体和数组的初始化。

### 2、c++17初始化列表增强

#### 2.1、类模板参数推导（CTAD）支持初始化列表

```cpp
// C++17前：需显式指定模板参数
std::vector<int> v1 = {1, 2, 3};

// C++17允许推导
std::vector v2{1, 2, 3}; // 推导为 vector<int>
std::pair p{42, "hello"}; // 推导为 pair<int, const char*>

```

#### 2.1、聚合初始化扩展

*   允许继承的聚合初始化：基类可以是聚合类型。

```cpp
struct Base { int a; };
struct Derived : Base { int b; };
Derived d{{1}, 2}; // C++17：基类成员先初始化

```

> 注意：基类成员先初始化

#### 2.2、允许直接列表初始化枚举

```cpp
enum class Color { Red, Green, Blue };
Color c{1}; // C++17允许，对应Color::Green

```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/634bb9b6e3124c8ba44823b8607d4020.png)

六、namespace的嵌套
--------------

*   在c++17前，声明namespace嵌套，是一层一层的命令。

```cpp
//C++17之前
namespace A {
    namespace B {
        namespace C {
            void func1() {}
        } //namespace C
    } //namespace B
} //namespace A

```

*   c++17是可以使用作用域限定符方式简化。

```cpp
//C++17
namespace A::B::C {
    void func1() {}
} //namespace A::B::C
```

本文转自 <https://blog.csdn.net/2401_82911768/article/details/147366051?spm=1001.2014.3001.5501>，如有侵权，请联系删除。