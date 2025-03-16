 

一、为什么需要[智能指针](https://so.csdn.net/so/search?q=%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88&spm=1001.2101.3001.7020)
-----------------------------------------------------------------------------------------------------------

智能指针主要解决的问题有以下内容：  
1.**内存泄漏**：内存手动释放，使用智能指针可以**自动释放**。 （自动[释放内存](https://so.csdn.net/so/search?q=%E9%87%8A%E6%94%BE%E5%86%85%E5%AD%98&spm=1001.2101.3001.7020)：智能指针退出作用域，就会在栈上自动析构。**先声明的后析构**）  
2.**共享所有权指针的传播和释放**，比如多线程使用同一个对象时析构问题。

> 智能指针线程安全的问题  
> 1、ref count是安全的。  
> 2、指向对象数据，如果修改，他是不安全的。（如果要数据安全，要加锁机制）

智能指针是属于栈上的。它的对象是堆上分配的。

*   **几个智能指针的特点**：
    *   unique\_ptr:独占对象的所有权，由于没有引用计数，因此性能较好。 它的删除器在编译时绑定。
    *   shared\_ptr:共享对象的所有权，但性能较差。它的删除器在运行时绑定。
    *   weak\_ptr:配合shared\_ptr，解决循环引用的问题。

二、shared\_ptr
-------------

### 1、理解

shared\_ptr 使用引用计数，每个shared\_ptr的拷贝都指向相同的内存。当**最后一个shared\_ptr析构**的时候，**内存才会被释放**。  
shared\_ptr共享管理对象，同一时刻可以多个shared\_ptr拥有对象的所有权，当最后一个shared\_ptr对象销毁时，被管理对象自动销毁。

*   简单来说,shared\_ptr包含2个部分，
    *   一个指向堆上创建的对象的裸指针raw\_ptr
    *   一个指向内部隐藏的、共享的管理对象。 share\_count\_object  
        第一部分没什么好说的，第二部分是重点。
*   use\_count,当前这个堆上对象被多少对象引用了，简单来说就是引用计数。**只能count=0才能被析构**。

### 2、基本用法和常用函数

s.get():返回shared\_ptr中保存的裸指针  
s.reset():用于**重置智能指针的管理对象**。通过 reset()，可以**释放当前管理的对象**（如果需要），并将指针指向新的对象或置**为空**。

*   reset不带参数时，若智能指针s是**唯一指向该对象的指针**，则释放，**并置空**。若智能指针p**不是唯一指向该对象的指针**，则引用计数**减少1**，同时**p置空**。
*   reset带参数时，若智能指针s是唯一指向对象的指针，则释放**并指向新的对象**。若p不是唯一的指针，则只减少引用计数，并指向新的对象。

#### 2.1、创建shared\_ptr方法

我们应该优先使用make\_shared来构造智能指针，因为他更高效。

```cpp
    
    auto sp1 = make_shared<int>(100);   
    或   
    shared_ptr<int>sp1 = make_shared<int>(100);
   //相当于  
	shared_ptr<int> sp1(new int(100)); 

```

```cpp
#include <memory>

// 方法1: 直接构造
std::shared_ptr<int> p1(new int(42));

// 方法2: 推荐使用 make_shared（更高效）
auto p2 = std::make_shared<int>(42);

// 方法3: 从 unique_ptr 转换
std::unique_ptr<int> uptr(new int(42));
std::shared_ptr<int> p3 = std::move(uptr);
```

#### 2.2、观察use\_count()和使用unique()

s.use\_count():返回shared\_ptr的强引用计数。  
s.unique():若use\_count()为1，返回true，否则false。

```cpp
#include <iostream>
#include <memory> // 包含智能指针的头文件
class MyClass {
public:
    MyClass(int value) : value(value) {
        std::cout << "MyClass constructed with value: " << value << std::endl;
    }

    ~MyClass() {
        std::cout << "MyClass destroyed with value: " << value << std::endl;
    }

    void Print() const {
        std::cout << "Value: " << value << std::endl;
    }
private:
    int value;
};
int main() {
    // 创建一个 shared_ptr
    std::shared_ptr<MyClass> s = std::make_shared<MyClass>(10);

    // 1. 测试不带参数的 reset
    std::cout << "--- Testing reset() without arguments ---" << std::endl;
    {
        std::shared_ptr<MyClass> p = s; // p 和 s 共享所有权
        std::cout << "Before reset:" << std::endl;
        std::cout << "s use_count: " << s.use_count() << std::endl; // 输出 2
        std::cout << "p use_count: " << p.use_count() << std::endl; // 输出 2

        p.reset(); // p 释放所有权，s 仍然持有对象
        std::cout << "After reset:" << std::endl;
        std::cout << "s use_count: " << s.use_count() << std::endl; // 输出 1
        std::cout << "p use_count: " << p.use_count() << std::endl; // 输出 0
    }

    // 2. 测试带参数的 reset
    std::cout << "--- Testing reset() with arguments ---" << std::endl;
    {
        std::shared_ptr<MyClass> p = s; // p 和 s 共享所有权
        std::cout << "Before reset:" << std::endl;
        std::cout << "s use_count: " << s.use_count() << std::endl; // 输出 2
        std::cout << "p use_count: " << p.use_count() << std::endl; // 输出 2

        p.reset(new MyClass(20)); // p 指向新对象，s 仍然持有原对象
        std::cout << "After reset:" << std::endl;
        std::cout << "s use_count: " << s.use_count() << std::endl; // 输出 1
        std::cout << "p use_count: " << p.use_count() << std::endl; // 输出 1

        p->Print(); // 输出新对象的值
        s->Print(); // 输出原对象的值
    }

    // 3. 测试唯一指针的 reset
    std::cout << "--- Testing reset() with unique ownership ---" << std::endl;
    {
        std::shared_ptr<MyClass> p = std::make_shared<MyClass>(30); // p 是唯一所有者
        std::cout << "Before reset:" << std::endl;
        std::cout << "p use_count: " << p.use_count() << std::endl; // 输出 1

        p.reset(new MyClass(40)); // p 释放原对象，指向新对象
        std::cout << "After reset:" << std::endl;
        std::cout << "p use_count: " << p.use_count() << std::endl; // 输出 1

        p->Print(); // 输出新对象的值
    }

    return 0;
}
```

#### 2.3、reset

使用reset构造shared\_ptr

```cpp
    shared_ptr<int> p1(new int(1));
    shared_ptr<int> p2=p1;
    shared_ptr<int> p3;
    p3.reset(new int(1));
    if(p3){
        cout<<"p3 is not null"<<endl;
    }
```

reset含有参数和无参数的构建

```cpp
auto p1 = std::make_shared<int>(42);  // 引用计数为 1
auto p2 = p1;                         // 引用计数为 2

p1.reset();  // p1 释放对象，引用计数减为 1
p2.reset();  // p2 释放对象，引用计数归零，对象被销毁

p1.reset(new int(100));  // p1 指向新对象
```

#### 2.4、当需要获取原始指针时，可以用get()来返回原始指针

```c++
    std::shared_ptr<int>p(new int(1));
    int* ptr = p.get();
```

> **get()会带来的坏处 1**  
> 1、生命周期管理问题：`std::shared_ptr` 的主要作用是自动管理对象的生命周期。当最后一个 `std::shared_ptr` 被销毁或重置时，它所管理的对象会被自动释放。然而，`p.get()` 返回的原始指针并**不参与引用计数**，因此无法保证对象的生命周期。

```c++
int main() {
    std::shared_ptr<MyClass> p = std::make_shared<MyClass>();
    MyClass* rawPtr = p.get(); // 获取原始指针

    // 假设在某个地方 p 被重置或销毁
    p.reset();

    // 此时 rawPtr 指向的对象已经被释放
    UseRawPointer(rawPtr); // 未定义行为（悬空指针）
    return 0;
}
```

> **get()会带来的坏处 2**  
> 2、无法保证线程安全：`std::shared_ptr` 的引用计数是线程安全的，但 `p.get()` 返回的原始指针并不提供任何线程安全保障。如果多个线程同时访问或修改原始指针指向的对象，可能会导致数据竞争或未定义行为。

```cpp
    
#include <iostream>
#include <memory>
#include <thread>

class MyClass {
public:
    int value = 0;
};

void ModifyValue(MyClass* ptr) {
    for (int i = 0; i < 100000; ++i) {
        ptr->value++; // 数据竞争
    }
}

int main() {
    std::shared_ptr<MyClass> p = std::make_shared<MyClass>();
    MyClass* rawPtr = p.get();

    std::thread t1(ModifyValue, rawPtr);
    std::thread t2(ModifyValue, rawPtr);

    t1.join();
    t2.join();

    std::cout << "Final value: " << rawPtr->value << std::endl; // 结果不确定
    return 0;
}
    
```

> **get()会带来的坏处 3**  
> 3、无法扩展对象的生命周期：`.get()` 返回的原始指针不会增加引用计数，因此无法延长对象的生命周期。如果所有 `std::shared_ptr` 都被销毁，即使原始指针仍然存在，对象也会被释放。

```cpp
#include <iostream>
#include <memory>

class MyClass {
public:
    ~MyClass() {
        std::cout << "MyClass destroyed!" << std::endl;
    }
};

int main() {
    MyClass* rawPtr = nullptr;
    {
        std::shared_ptr<MyClass> p = std::make_shared<MyClass>();
        rawPtr = p.get(); // 获取原始指针
    } // p 超出作用域，对象被释放

    // 此时 rawPtr 是悬空指针
    std::cout << "Raw pointer still exists, but object is destroyed." << std::endl;
    return 0;
}   
    
```

#### 2.5、指定删除器

如果使用shared\_ptr管理**非new对象或是没有析构函数的类**时，应当为其传递合适的删除器。

```cpp
   
void DeleteIntPtr(int* p){
    cout<<"call DeleteIntPtr"<<endl;
    delete p;
}
int main(){
    shared_ptr<int>p (new int(1), DeleteIntPtr);
    return 0;
}
```

也可以用**lambda函数**写。

```cpp
    
int main(){
    shared_ptr<int>p (new int(1), [](int* p){
    cout<<"call lambda delete p"<<endl;
    delete p;});
    return 0;
}
    
```

### 3、需要注意的问题

#### 3.1、不要用原始指针，初始化多个shared\_ptr

```cpp
    int * ptr= new int;
    shared_ptr<int>p1(ptr);
    shared_ptr<int>p2(ptr);//逻辑错误
```

#### 3.2、不要在函数参数列表中用shared\_ptr

因为在不同编译器中，计算顺序是不同的，一般从右到左。如果从左到右，右边出错，而左边写的shared\_ptr初始化，就会报错。

```cpp
//不能写成
func(shared_ptr<int>,g());//在func函数里写shared_ptr

//可以写成
class MyClass {
public:
    void do_something() const {
        std::cout << "Doing something!\n";
    }
};

void process_object(const MyClass* obj) {
    if (obj) {
        obj->do_something();
    }
}

int main() {
    auto ptr = std::make_shared<MyClass>();
    process_object(ptr.get());  // 传递裸指针
    return 0;
}
    
```

#### 3.3、通过shared\_from\_this()返回指针

**不要将this指针作为shared\_ptr返回出来。因为this指针本质是一个裸指针**，因此，这样可能会导致重复析构。

```cpp
class A{
public:
    shared_ptr<A> getSelf(){
        return shared_ptr<A>(this);//不能这么做
    }
    ~A(){
        cout<<"Destructor A"<<endl;
    };
};
int main(){
   shared_ptr<A> sp1(new A);
   shared_ptr<A>sp2 = sp1->getSelf();
    //会打印2次
    //Destructor A
    //Destructor A
    return 0;
}
```

正确改法：

```cpp
    class A:public std::enable_shared_from_this<A>{
    public:
        shared_ptr<A> GetSelf(){
        return shared_from_this;
    }
}
```

#### 3.4、避免循环引用

循环引用指的是两个或多个对象通过 `std::shared_ptr` 相互持有对方的引用，形成一个环状依赖关系。由于 `std::shared_ptr` 的引用计数机制，这些对象的引用计数永远不会降为 0，导致它们无法被正确释放。

```cpp
#include <iostream>
#include <memory>

class B; // 前向声明

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed!" << std::endl; }
};

class B {
public:
    std::weak_ptr<A> a_ptr; // 使用 weak_ptr 代替 shared_ptr
    ~B() { std::cout << "B destroyed!" << std::endl; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();

    a->b_ptr = b; // A 持有 B 的 shared_ptr
    b->a_ptr = a; // B 持有 A 的 weak_ptr

    // 引用计数：
    // a 的引用计数：1（只有 a）
    // b 的引用计数：2（b 和 a->b_ptr）

    // 退出作用域时：
    // a 的引用计数降为 0，A 被释放
    // b 的引用计数降为 1，但由于 A 被释放，B 的引用计数最终降为 0，B 被释放
    return 0;
}  
```

三、unique\_ptr
-------------

unique\_ptr:是一个独占型智能指针，它不允许其他的智能指针共享其内部的指针，不允许通过赋值将一个unique\_ptr赋值给另一个unique\_ptr。

```cpp
unque_ptr<int>p1(new int);  
unque_ptr<int>p2 = p1;  
```

unique\_ptr不能复制，但是可以通过函数返回给其他的unique\_ptr，还可以通过std::move来转移到其他函数的unique\_ptr,这样它本身不再拥有原来指针的所有权了。例如：

```cpp
    unique_ptr<T> my_ptr(new T);//
    unique_ptr<T>my_other_ptr= std::move(my_ptr);
    
```

**make\_unique是14的性质，不属于11。而make\_shared是11**。  
使用new的版本重复了被创建对象的键入，但是make\_unique函数则没有。重复类型违背了软件工程的一个重要原则：应避免重复会引起编译次数增加，导致目标代码膨胀。

*   除了unique\_ptr的独占性，与shared\_ptr还有一些区别。  
    **可以指向数组，但是shared\_ptr不能。并且数组必须指定大小**。

```cpp
    unique_ptr<int []>ptr(new int[9]);//必须指定大小
    ptr[9]=8;
    shared_ptr<int []>ptr (new int[]);//报错
    
```

*   unique\_ptr指定删除器和shared\_ptr有区别。

```cpp
    shared_ptr<int>ptr1 (new int(1),[](int* p){delete p;});//正确
    unique_ptr<int>ptr2 (new int(1),[](int* p){delete p;});//错误
```

*   错误原因
    *   std::unique\_ptr 的删除器是类型的一部分，**需要在模板参数中指定删除器的类型**。
    *   **直接传递删除器会导致编译错误**，因为**编译器无法推断删除器的类型**
    *   如下是修改方法

（1）使用 decltype 推断删除器类型

```cpp
auto deleter = [](int* p) { delete p; };
std::unique_ptr<int, decltype(deleter)> ptr2(new int(1), deleter);
```

(2) 直接指定删除器类型

```cpp
// 使用函数指针
void custom_deleter(int* p) { delete p; }
std::unique_ptr<int, void(*)(int*)> ptr2(new int(1), custom_deleter);

// 使用 lambda（需要指定类型）
std::unique_ptr<int, std::function<void(int*)>> ptr2(new int(1), [](int* p) { delete p; });
```

shared\_ptr和unique\_ptr的使用场景是要根据实际应用需求来选择。

> 如果**只希望使用一个智能指针管理资源或者数组用unique\_ptr**，如果希望多个智能指针管理同一个资源就使用shared\_ptr。

四、weak\_ptr
-----------

shared\_ptr虽然很好了，但是有[内存泄漏](https://so.csdn.net/so/search?q=%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F&spm=1001.2101.3001.7020)问题。当两个对象相互使用一个shared\_ptr成员变量指向对方，会造成循环引用，使引用计数失效，从而内存泄漏。  
weak\_ptr是一种不控制对象生命周期的智能指针，**它指向一个shared\_ptr管理的对象**，进行该对象的内存管理的是那个强引用的shared\_ptr，weak\_ptr只是提供了对管理对象的一个访问手段。  
weak\_ptr（弱引用的智能指针）是用来辅助shared\_ptr的。

### 4.1、weak\_ptr的基本用法

1、可以使用use\_count()获取当前资源的引用计数。

```cpp
    shared_ptr<int> sp(new int(10));  
    weak_ptr<int>wp(sp);  
    cout<<wp.use_count<<endl;//结果输出1  
```

2、使用`expired()`方法判断所观察资源是否已经释放，如下：

```cpp
    shared_ptr<int>sp(new int(10));
    weak_ptr<int>wp(sp);
    if(wp.expired()){
        cout<<"weak_ptr无效，资源已经释放";
    }else{
        cout<<"weak_ptr有效";
    }
```

3、通过lock获取监视的shared\_ptr,如下所示：

```cpp
void f(){
    std::weak_ptr<int>gw;
    auto spt = gw.lock();
    if(gw.expired()){
        cout<<"gw无效，资源已经释放";
    }else{
        cout<<"gw有效";
    }
}
int main(){

    f();

    return 0;
}
```

#### 4.2、weak\_ptr返回this指针

在C++中，`std::weak_ptr` 是一种弱引用智能指针，它不会增加所指向对象的引用计数。`std::weak_ptr` 通常用于解决 `std::shared_ptr` 的循环引用问题。如果你需要从一个类的成员函数中返回一个指向当前对象的 `std::weak_ptr`，可以通过 `std::shared_from_this` 来实现。

使用 `std::enable_shared_from_this`

为了让一个类能够安全地返回 `std::weak_ptr` 或 `std::shared_ptr` 指向自身，该类需要继承 `std::enable_shared_from_this`。以下是一个示例：

```cpp
#include <iostream>
#include <memory>

class MyClass : public std::enable_shared_from_this<MyClass> {
public:
    std::weak_ptr<MyClass> getWeakPtr() {
        // 使用 shared_from_this() 获取当前对象的 shared_ptr，然后转换为 weak_ptr
        return weak_from_this();
    }

    void doSomething() {
        std::cout << "Doing something!" << std::endl;
    }
};

int main() {
    // 创建一个 shared_ptr 指向 MyClass 对象
    auto sharedPtr = std::make_shared<MyClass>();

    // 获取一个 weak_ptr 指向同一个对象
    std::weak_ptr<MyClass> weakPtr = sharedPtr->getWeakPtr();

    // 使用 weak_ptr
    if (auto lockedPtr = weakPtr.lock()) {  // 尝试将 weak_ptr 提升为 shared_ptr
        lockedPtr->doSomething();  // 如果对象仍然存在，调用成员函数
    } else {
        std::cout << "Object has been destroyed!" << std::endl;
    }

    return 0;
}
```

#### 4.3、weak\_ptr 解决循环引用计数

`std::weak_ptr` 是一种弱引用，它不会增加对象的引用计数。通过将其中一个 `std::shared_ptr` 替换为 `std::weak_ptr`，可以打破循环引用。

```cpp
#include <iostream>
#include <memory>

class B;  // 前向声明

class A {
public:
    std::shared_ptr<B> bPtr;
    ~A() { std::cout << "A destroyed" << std::endl; }
};

class B {
public:
    std::weak_ptr<A> aWeakPtr;  // 使用 weak_ptr 代替 shared_ptr
    ~B() { std::cout << "B destroyed" << std::endl; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();

    a->bPtr = b;        // A 持有 B 的 shared_ptr
    b->aWeakPtr = a;    // B 持有 A 的 weak_ptr

    // 当 main 函数结束时，a 和 b 的引用计数会归零，对象会被正确销毁
    return 0;
}
```

*   当 `main` 函数结束时，`a` 的引用计数变为 0，`A` 对象被销毁。
*   `A` 对象销毁后，`b` 的引用计数也变为 0，`B` 对象被销毁。
*   如果需要访问 `std::weak_ptr` 所指向的对象，可以调用 `lock()` 方法将其提升为 `std::shared_ptr`。

本文转自 <https://blog.csdn.net/2401_82911768/article/details/145694145?spm=1001.2014.3001.5502>，如有侵权，请联系删除。