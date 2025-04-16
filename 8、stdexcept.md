在 C++ 中，`<stdexcept>` 头文件定义了标准异常类的继承体系，主要包括两类基础异常：`logic_error`（逻辑错误）和 `runtime_error`（运行时错误）。这些异常类为程序员提供了一种统一的异常处理方式。

---

### **异常类继承关系**
```cpp
exception
    ├── logic_error         // 逻辑错误（可预防的程序错误）
    │     ├── invalid_argument
    │     ├── domain_error
    │     ├── length_error
    │     └── out_of_range
    │
    └── runtime_error        // 运行时错误（不可预测的错误）
          ├── range_error
          ├── overflow_error
          ├── underflow_error
          └── system_error (C++11 起)
```

---

### **核心用法示例**
#### 1. **直接抛出标准异常**
```cpp
#include <stdexcept>
#include <iostream>
#include <vector>

void validate_input(int val) {
    if (val < 0) {
        throw std::invalid_argument("输入值不能为负");
    }
}

int main() {
    std::vector<int> vec{1, 2, 3};
    try {
        validate_input(-5);               // 触发 invalid_argument
        std::cout << vec.at(10);          // 触发 out_of_range
    }
    catch (const std::invalid_argument& e) {
        std::cerr << "参数错误: " << e.what() << "\n";
    }
    catch (const std::out_of_range& e) {
        std::cerr << "范围错误: " << e.what() << "\n";
    }
    catch (const std::exception& e) {     // 捕获所有标准异常
        std::cerr << "其他错误: " << e.what() << "\n";
    }
}
```

#### 2. **自定义派生异常**
```cpp
class NetworkError : public std::runtime_error {
public:
    NetworkError(const std::string& msg)
        : std::runtime_error("[网络错误] " + msg) {}
};

void connect_to_server() {
    if (/* 网络连接失败 */) {
        throw NetworkError("无法连接服务器");
    }
}
```

---

### **常用异常类型速查表**
| **异常类型** | **适用场景** | **示例触发条件** |
| --- | --- | --- |
| `invalid_argument` | 函数参数无效 | 传递负数给要求正数的函数 |
| `out_of_range` | 访问超出范围的索引 | `vector::at()` 越界访问 |
| `length_error` | 对象长度超过最大允许值 | `std::string` 尝试分配过大长度 |
| `overflow_error` | 算术运算结果超出类型上限 | `numeric_limits<int>::max() + 1` |
| `underflow_error` | 算术运算结果低于类型下限 | `numeric_limits<float>::min() - 1` |
| `range_error` | 内部计算范围错误 | `std::bitset` 类型转换时的值超限 |


---

### **最佳实践**
1. **优先选择标准异常**  
在自定义错误时，优先从 `logic_error` 或 `runtime_error` 继承，而非直接继承 `exception`。
2. **构造异常时提供清晰消息**  
通过构造函数传递描述性字符串（如 `throw std::runtime_error("文件未找到: " + filename)`）。
3. **捕获按派生类到基类排序**  
先捕获特定的子类异常，最后捕获通用的 `std::exception`，避免误拦截：

```cpp
try { ... }
catch (const std::invalid_argument& e) { /* 处理具体错误 */ }
catch (const std::exception& e) { /* 兜底处理 */ }
```

4. **不滥用异常替代正常逻辑**  
频繁抛出异常（如循环内部）会显著影响性能，建议只在异常情况使用。

---

### **常见问题**
#### Q1: **如何获取异常的错误信息？**
通过 `what()` 成员函数：

```cpp
try { ... }
catch (const std::exception& e) {
    std::cerr << e.what() << "\n";  // 输出类似 "输入值不能为负"
}
```

#### Q2: **空指针异常在 **`<stdexcept>`** 中吗？**
不在。空指针异常由 `std::bad_alloc`（内存分配失败）或手动检查指针触发，通常需要自行处理。

#### Q3: **为什么建议继承 **`std::exception`**？**
保持异常体系的一致性，方便通过基类统一捕获所有标准异常。

