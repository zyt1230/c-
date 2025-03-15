# 一、auto

## 1、作用

auto 用于**自动推导`变量`的类型**，编译器会根据初始化表达式推断变量的类型。

- **核心规则**
  - **忽略顶层 const 和引用**（**除非显式声明为 const 或引用，就是在auto前或auto后写const或引用**）。
  - 类型推导基于初始化表达式的值类型（**即表达式本身的类型，不含引用和 const**）。

> **那么什么是显示声明**？
> 例如：
> 
> ```cpp
> int x = 10;
> const int y = 20;
> int& ref = x;
> 
> auto a = x;       // a 的类型是 int（忽略顶层 const 和引用）
> auto b = y;       // b 的类型是 int（忽略 const）
> auto c = ref;     // c 的类型是 int（忽略引用）
> auto& d = ref;    // d 的类型是 int&（显式保留引用）
> const auto e = y; // e 的类型是 const int（显式声明 const）
> 
> ```

## 2、区分const，&

举例说明，加const 或&的情况下，auto的变量是什么类型。**通过这个例子，大家就能理解到auto的转换了**。常见的写法，例如：

```cpp
#include<iostream>
#include<cstdio>

using namespace std;
int main(int argv,char* argc[]){
    int x =0;
    int & y =x;
    const int z = 10;
    auto x1= x;//x1是int
    auto x2 = y;//x2是int
    auto &x3= x;//x3是int &
    auto &x4 = y;//x4是int &
    const auto x5= x;//x5是const int
    const auto x6 = z;//x6是const int
    auto x7=z;//x7是int
    auto &x8 =z;//x8是const int &
}
```

## 3、auto的高级使用

- 指针与引用【如果第2点能懂，那么这里稍微看看就能明白了】
  
  ```cpp
  #include<iostream>
  #include<cstdio>
  using namespace std;
  int main(int argv,char* argc[]){
      int x =0;
      auto * x1=&x;//x1为int*  也就是说auto是int
      auto x2 =&x;//x2为int*   auto推导为int*
      auto& x3 = x;//x3为int& auto推导为int
      auto x4 = x;//x4是int     auto推导为int
  }
  ```
  
  

- **C++11** lambda中使用

```cpp
    auto func = [](int a, int b) { return a + b; };
```

   

- **C++14** lambda**支持auto参数**
  
  ```cpp
  auto lambda = [](auto a, auto b) { return a + b; };
  
  ```

## 4、auto的4个限制

- 不能在**函数的参数**中使用

- 不能作用于**类的非静态成员变量**

- 不能作用于**模版参数**

- 不能推导**数组**类型

# 二、decltype

## 1、作用

decltype 用于**推导`表达式`的类型**，保留表达式的顶层 const 和引用。

- **核心规则**
  - **保留表达式的完整类型（包括 const 和引用）**。
  - **变量名**：推导声明类型；**表达式**：推导计算结果的类型。

例如：

```cpp
int x = 10;
const int y = 20;
int& ref = x;

decltype(x) a = x;        // a 的类型是 int
decltype(y) b = y;        // b 的类型是 const int
decltype(ref) c = ref;    // c 的类型是 int&
decltype(x + y) d = x;    // d 的类型是 int（表达式 x+y 的类型是 int）
decltype((x)) e = x;      // e 的类型是 int&（括号强制为表达式，推导出引用）


```

> **注意**：**decltype((x))这里有强制转换成引用**

## 2、decltype的高级使用

- 模板编程中推导**返回值类型**
  
  - **C++11**中template**需要decltype**来得add的**返回**类型。**C++14可以自动推出**，不用写decltype。
    
    ```cpp
    #include<iostream>
    #include<cstdio>
    #include<typeinfo>
    using namespace std;
    template <class R,class T,class U>
    R add(T t,U u){
    return t+u;
    }
    int main(int argv,char* argc[]){
    int x = 10;
    double y = 20.0;
    auto result = add<decltype(x+y)> (x,y);
    }
    ```
    
    或写成
    
    ```cpp
    #include<iostream>
    #include<cstdio>
    #include<typeinfo>
    using namespace std;
    template <class T,class U>
    auto add(T t,U u)->decltype(t+u){//可能大家会发现这里是auto而不是class R
    //之所以这样是因为，如果使用class R，就会冗余，无法推导了，需要强制转换才能改回来。强制转换就是上面的写法
    return t+u;
    }
    int main(int argv,char* argc[]){
    int x = 10;
    double y = 20.0;
    auto result = add(x,y);
    }
    ```

- 保留引用和 const 属性
  
  ```cpp
  const std::vector<int> vec = {1, 2, 3};
  decltype(vec[0]) elem = vec[0]; // elem 的类型是 const int&
  
  ```

```

- **C++14**中auto和decltype相结合了
```cpp
int main(int argv,char* argc[]){
    int x = 10;
    int y =10;
    decltype(auto) res =  x+y;
}
```

# 三、类型检查工具

1. 运行时类型信息（RTTI）
   使用 typeid 配合 typeinfo 获取类型名称（仅作调试）：

```cpp
#include <typeinfo>
auto x = 42;
std::cout << typeid(x).name();  // 输出 "i"（表示 int）
```

> 注意
> 
> - **typeid 无法区分引用/const**（需结合 decltype）。
> - 编译器输出名称可能非标准化（**如 "i" 表示 int**）。

# 四、综合对比表

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4c3584a54a9b4f208315a74f28edb868.png)
