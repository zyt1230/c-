 

一、tuple
-------

### 1、了解

*   **作用**：包裹多个**不同类型的值**，实现轻量级数据组合。

```
头文件
#include<tuple>
```

### 2、基本操作

#### 2.1、构造和初始化

*   类型：
    *   直接构造
    *   辅助函数构造
    *   类模版参数推导
    *   tie()构造
    *   forward\_as\_tuple

> 注意:  
> ①tuple 的c++17起，可以**自己推导出来类型**。  
> ②std::**forward\_as\_tuple**的初始化方式是**无法使用结构化绑定访问元素的**。

```cpp
#include <tuple>
#include <string>

// 构造方式
std::tuple<int, std::string, double> t1(42, "hello", 3.14);  // 直接构造
auto t2 = std::make_tuple(100, "world", 2.718);              // 使用辅助函数

// C++17起支持类模板参数推导（CTAD）
std::tuple t3(3.14f, "PI");  // 自动推导为 tuple<float, const char*>
//tie()
	int a;
    double b;
    string c;
    tie(a,b,c)=make_tuple(1,2.33,"3");
//forward_as_tuple
auto tuple4=forward_as_tuple(1,2,"2222");

```

#### 2.2、元素访问

*   get<索引>(tuple\_name)

```cpp
int i =get<0>(tuple1)
```

*   tie(变量名)访问

```cpp
#include <iostream>
#include<tuple>
#include<string>
using namespace std;
int main(){
    int a;
    int b;
    string c;
    tie(a,b,c)=make_tuple(1,2,"2222");
    cout<<a<<endl;//1
    cout<<b<<endl;//2
    cout<<c<<endl;//2222
}
```

*   结构化绑定访问

```cpp
int main(){
    auto [i,j,k] = make_tuple(1,2,"2");
    cout<<i<<endl;//1
    cout<<j<<endl;//2
    cout<<k<<endl;//3
}
```

#### 2.3、tuple的交换

*   current\_tuple.swap(other\_tuple)
    *   交换2个tuple

```cpp
int main() {
    // 创建一个包含int和string的元组tuple1，初始值为1和"hello"
    tuple tuple1(1, "hello");

    // 使用C++17的类模板参数推导，创建元组tuple2(2,"world")
    // 编译器会自动推导类型为tuple<int, string>
    tuple tuple2(2, "world");

    // 交换tuple1和tuple2的内容
    tuple1.swap(tuple2);
    cout<<get<0>(tuple1)<<" "<<get<1>(tuple1)<<endl;
    cout<<get<0>(tuple2)<<" "<<get<1>(tuple2)<<endl;
/*
2 world
1 hello
 * */

    return 0;  // 程序正常结束
}
```

#### 2.4、tuple的拼接和递归打印tuple元素

*   tuple\_cat(其他tuple)

```cpp
#include <iostream>    // 包含标准输入输出流库
#include <tuple>       // 包含元组库
#include <string>      // 包含字符串库
using namespace std;   // 使用标准命名空间
template <size_t I=0,class ... Ts>
void print_tuple(const tuple<Ts...>&t){
    if constexpr (sizeof...(Ts)>I){
        cout<<get<I>(t)<<" ";
        print_tuple<I+1>(t);// 递归打印下一个元素
    }
}

int main() {
    // 创建一个包含int和string的元组tuple1，初始值为1和"hello"
    tuple tuple1(1, "hello");

    tuple tuple2(2, " world");
    tuple tuple3(3, " you");

    // 拼接tuple1和tuple2的内容
    auto new_tuple = tuple_cat(tuple1,tuple2,tuple3);
    print_tuple(new_tuple);
    return 0;

/*
1 hello 2  world 3  you
 * */
}
```

#### 2.5、大小

*   tuple\_size

```cpp
    // 获取元组大小
    constexpr size_t size1 = std::tuple_size<decltype(tuple1)>::value;
    constexpr size_t size2 = std::tuple_size_v<decltype(tuple2)>;
```

#### 2.6、获取元素类型

*   #include <type\_traits> // 需要包含此头文件
*   typeid(tuple\_name).name()获取名字
*   static\_assert(is\_same\_v(type，类型))判断是否正确
*   tuple\_emlement\_t<index，decltype>获取类型

```cpp
#include<iostream>
#include<tuple>
#include<type_traits>
using namespace std;   // 使用标准命名空间
int main() {
    // 创建一个包含int和string的元组tuple1，初始值为1和"hello"
    tuple tuple1(1, "hello");
    using type0 = tuple_element<0,decltype(tuple1)>::type;// 传统写法
    using type1 = tuple_element_t<0, decltype(tuple1)>;// C++14起更简洁的写法
    static_assert(is_same_v<type0,int>);
    static_assert(is_same_v<type1, int>);    // 验证类型是否为int
    cout<<"tuple1[0]的类型"<<typeid(type0).name()<<endl;
    cout<<"tuple1[1]的类型"<<typeid(type1).name()<<endl;
    return 0;

/*

 * */
}
```

二、arrary
--------

### 1、了解

array是序列式容器，类似于C语言的数组，是**固定大小的**，一旦定义完成后就**无法进行扩容或收缩**。

### 2、访问操作

*   begin
    
    *   返回容器中第一个元素的迭代器
*   end
    
    *   返回容器中最后一个元素**之后**的迭代器
*   rbegin
    
    *   返回指向最后一个元素的迭代器
*   rend  
    \-返回指向第一个元素之前的迭代器
    

```cpp
在这里插入代码片
#include<iostream>
#include<array>

int main()
{
	std::array<int, 4> arr = { 1, 3, 2, 4 };
	std::cout << "arr values:" << std::endl;
	for (auto it = arr.begin(); it != arr.end(); it++) {
		std::cout << *it << " ";
	}
	std::cout << std::endl;
	std::cin.get();
	return 0;
}

```

### 3、大小操作

*   size
    *   获取大小

```cpp
在这里插入代码片
#include<iostream>
#include<array>

int main()
{
	std::array<int, 4> arr = { 1, 3, 2, 4 };
	
	std::cout << "size of arr = " << arr.size() << std::endl;
	
	
	std::cin.get();
	return 0;
}

```

*   empty
    *   判断是否为空

```cpp
#include<iostream>
#include<array>

int main()
{
	std::array<int, 4> arr = { 1, 3, 2, 4 };
	
	std::cout << "empty = " << (arr.empty() ? "no" : "yes") << std::endl;
	std::cin.get();
	return 0;
}
```

### 4、元素操作

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7b3989ef545447c18bc5fb81880e4aed.png)

```cpp
在这里插入代码片
#include<iostream>
#include<array>

int main()
{
	std::array<int, 4> arr = {};
	for (int i = 0; i < arr.size(); i++)
	{
		arr.at(i) = i;
	}
	for (auto it = arr.begin(); it != arr.end(); it++)
	{
		std::cout << *it << " " ;
	}
	std::cout << std::endl;
	std::cout << arr.data() << std::endl;
	std::array<int, 4> arr1 = { 4, 5, 6,7 };
	arr1.swap(arr);
	for (auto it = arr.begin(); it != arr.end(); it++)
	{
		std::cout << *it << " ";
	}
	std::cout << std::endl;
	arr.fill(1);
	for (auto it = arr.begin(); it != arr.end(); it++)
	{
		std::cout << *it << " ";
	}
	std::cout << std::endl;
	std::cin.get();
	return 0;
}
```

*   [借用的这个知识点](https://blog.csdn.net/qq_26646107/article/details/106967633?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522b140cb70998479f7cb138081f40c0d54%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=b140cb70998479f7cb138081f40c0d54&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-106967633-null-null.142%5Ev102%5Epc_search_result_base9&utm_term=c%2b%2barray&spm=1018.2226.3001.4187)

三、结构化绑定
-------

### 1、了解

作用：解包复合类型数据（元组、结构体、数组）

### 2、使用

*   可以用在for\_each中或者获取键值对等等。

```cpp
std::map<int, string> m{{1, "a"}, {2, "b"}};
for (const auto& [key, val] : m) { // 直接解包键值对
    cout << key << ":" << val << endl;
}

auto getData() -> std::tuple<int, string, double> {
    return {42, "PI", 3.14};
}

auto [id, name, value] = getData(); // 自动解包元组

```

本文转自 <https://blog.csdn.net/2401_82911768/article/details/147018395?sharetype=blogdetail&sharerId=147018395&sharerefer=PC&sharesource=2401_82911768&spm=1011.2480.3001.8118>，如有侵权，请联系删除。