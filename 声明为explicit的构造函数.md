## **声明为 `explicit` 的构造函数不能用于隐式转换**
只包含一个参数的构造函数，承担了两个角色：
1. 构造函数
2. 默认且隐含的类型转换操作符

比如，`A a = n`，n 的类型正好是类 A 单参数构造函数的参数类型，编译器自动调用构造函数，创建一个类型为 A 的对象。

```c++
#include <iostream>

using namespace std;

class A {
public:
    A(int n) {
        num = n;
    }

private:
    int num;
};

class B {
public:
    explicit B(int n) {
        num = n;
    }

private:
    int num;
};

int main() {
    A a = 12;  // OK
    B b = 12;  // error: no viable conversion from 'int' to 'B'
    B b(12);   // OK
    return 0;
}
```
