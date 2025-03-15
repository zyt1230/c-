C++中的`<sstream>`库**提供了字符串流处理功能**，允许将数据转换为字符串或从字符串中解析数据。以下是其主要用法和注意事项：

# 1、将数据转换为字符串（使用ostringstream）

**示例**：将整数和浮点数格式转化为字符串。

```cpp
#include<iostream>
#include<sstream>
using namespace std;

int main(int argv,char* argc[]){

    std::ostringstream oss;
    oss << "整数：" << 42 << "，浮点数：" << 3.14;
    std::string result = oss.str(); // "整数：42，浮点数：3.14"
    cout<<result;
}3.14"


```

# 2、从字符串解析数据（使用`istringstream`）

**示例**：提取字符串中的整数、浮点数和字符串。

```cpp
#include<iostream>
#include<sstream>
using namespace std;

int main(int argv,char* argc[]){
    string input = "123 456.7 abc";
    istringstream iss(input);
    int i;
    double d;
    string s;

    iss>>i>>d>>s;
    cout<<i<<endl;//123
    cout<<d<<endl;//456.7
    cout<<s<<endl;//abc
    //input不能在123前面写字符
    //123和456.7连在一起打印的是123456和0.7
}


```

**注意**：若提取失败（如类型不匹配），流会进入错误状态，需通过`clear()`重置：

```c
if (!(iss >> i)) {
    // 处理错误
    iss.clear(); // 清除错误状态
}

```

# 3、混合读写

**示例**：写入后读取数据。

```cpp
#include<iostream>
#include<sstream>
using namespace std;

int main(int argv,char* argc[]){
    stringstream ss;
    ss<<"100 200";
    int a,b;
    ss>>a>>b;
    cout<<a<<" "<<b<<endl;
}


```

**重复使用流对象时**：

```cpp
ss.clear();  // 清除错误状态
ss.str("");  // 清空内容
ss << "新数据"; // 重新写入

```

```cpp
#include<iostream>
#include<sstream>
using namespace std;

int main(int argv,char* argc[]){
    stringstream ss;
    ss<<"100 200";
    int a,b;
    ss>>a>>b;
    cout<<a<<" "<<b<<endl;
    //不清除的话结果还是100 200而不是123 456
    ss.clear();
    ss.str("");
    
    ss<<"123 456";
    ss>>a>>b;
    cout<<a<<" "<<b<<endl;
}
```

### 4. 处理复杂格式

**按分隔符提取**（如逗号分隔）：

```cpp
#include<iostream>
#include<sstream>
using namespace std;

int main(int argv,char* argc[]){
    string input = "one,two,three";
    istringstream iss(input);
    string token;
    while(getline(iss,token,',')){
        cout<<token<<" ";//one two three
    }
}
```

**格式化输出**（需`<iomanip>`）：

```cpp
#include <iomanip>
std::ostringstream oss;
oss << std::fixed << std::setprecision(2) << 3.14159; // 输出3.14

```
