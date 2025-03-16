 

一、[命名空间](https://so.csdn.net/so/search?q=%E5%91%BD%E5%90%8D%E7%A9%BA%E9%97%B4&spm=1001.2101.3001.7020)  
命名空间只能在全局范围内使用。  
它可以避免命名冲突的问题。  
1.使用方法  
( : : 符号为作用域限定符)  
方法一：指定命名空间

```cpp

namespace N1{
    int n;
    int add(int left,int right){
        return left+right;
    }
}
//也可以嵌套
namespace N2{
    int a;
    int b;
    int add(int left,int right){
        return left+right;
    }
    namespace N3{
        int c;
        int a;
        int sum(int left,int right){
            return left+right;
        }
    }
}
//使用可以指定它来自哪个命名空间
int main(){
    cin>>N1::n;
    cin>>N2::N3::a;
}
```

方法二：using引入成员

```cpp
namespace N1{
    int n;
    int add(int left,int right){
        return left+right;
    }
}
namespace N2{
    int a;
    int b;
    int add(int left,int right){
        return left+right;
    }
    namespace N3{
        int c;
        int a;
        int sum(int left,int right){
            return left+right;
        }
    }
}

int main(){
    using N1::n;
    cin>>n;//n就不用指定了
    cin>>N2::N3::a;

}
```

方法三：using引入空间全部

```cpp
namespace N1{
    int n;
    int add(int left,int right){
        return left+right;
    }
}
namespace N2{
    int a;
    int b;
    int add(int left,int right){
        return left+right;
    }
    namespace N3{
        int c;
        int a;
        int sum(int left,int right){
            return left+right;
        }
    }
}

int main(){
    using namespace N1;
    cin>>n;//n就不用指定了
    add(1,3);
}
```

本文转自 <https://blog.csdn.net/2401_82911768/article/details/145694083?spm=1001.2014.3001.5502>，如有侵权，请联系删除。