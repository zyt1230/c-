![0ad92d92-0b05-4cab-9fd7-6dfebf1a2b29](file:///C:/Users/LEGION/Pictures/0ad92d92-0b05-4cab-9fd7-6dfebf1a2b29.png)

const写在成员函数后面，只能被const 对象使用。

```cpp
size_t size() const {
    return _size;
}
const string& s;只能被这种const对象使用。
```
