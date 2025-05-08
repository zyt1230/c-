# `explicit` 关键字在 C++ 中的作用

`explicit` 是 C++ 中的一个关键字，主要用于防止构造函数的隐式类型转换。

## 主要用途

1. **用于构造函数**：防止编译器进行隐式的类型转换
2. **用于转换运算符** (C++11 起)：防止隐式的类型转换

## 基本用法

### 1. 用于单参数构造函数

```cpp
class MyClass {
public:
    explicit MyClass(int x) { /*...*/ }
};

void func(MyClass obj) {}

int main() {
    MyClass obj1(10);     // 正确 - 显式调用
    MyClass obj2 = 10;    // 错误 - 不能隐式转换
    func(10);             // 错误 - 不能隐式转换
    func(MyClass(10));    // 正确 - 显式转换
}
```

### 2. 用于多参数构造函数 (C++11 起)

```cpp
class MyClass {
public:
    explicit MyClass(int x, int y) { /*...*/ }
};

MyClass obj1(1, 2);      // 正确
MyClass obj2 = {1, 2};   // 错误 - 不能隐式转换
```

### 3. 用于转换运算符 (C++11 起)

```cpp
class MyBool {
public:
    explicit operator bool() const { return true; }
};

MyBool b;
if (b) { /*...*/ }       // 正确 - 在布尔上下文中允许
bool x = b;              // 错误 - 不能隐式转换
bool y = static_cast<bool>(b); // 正确 - 显式转换
```

## 为什么要使用 explicit

1. **避免意外的类型转换**：防止编译器在你不注意的情况下进行转换
2. **提高代码清晰度**：明确表明转换是设计者有意为之的
3. **减少错误**：避免潜在的逻辑错误和歧义

在大多数情况下，除非你确实需要隐式转换，否则将单参数构造函数声明为 `explicit` 是一个好习惯。
