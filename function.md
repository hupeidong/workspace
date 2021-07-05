`std::function` 是函数模板类。

```c++
#include <iostream>
#include <string>
#include <functional>

using namespace std;

std::function<void(int a)> func1;
std::function<void(std::string str)> func2;
std::function<std::string(int a, int b)> func3;

void func(int a) {
    cout << "普通函数" << endl;
}

void static_func(std::string str) {
    cout << "普通静态函数" << endl;
}

auto lambda = [](int a, int b) -> std::string { return "body"; };

int main() {
    func1 = func;
    func1(3);
    func2 = static_func;
    func2("hello");
    func3 = lambda;
    cout << func3(2, 3) << endl;
    return 0;
}

输出结果为：
ZBMac-C02F31WYM Desktop % ./main
普通函数
普通静态函数
body
```
