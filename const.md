# const

## 成员函数

编译器强制实施 bitwise constness，但你编写程序时应该使用概念上的常量性。
当 const 和 non-const 成员函数有着实质等价的实现时，令 non-const 版本调用 const 版本可避免重复。

### 1. 概念：bitwise constness

不更改对象的任一 bit。

### 2. 概念：logical constness

一个 const 成员函数可以修改它所处理的对象的某些 bits，但要保证在用户使用中侦测不出。

### 3. mutable 成员变量

mutable 成员变量，即使在 const 成员函数内也可被修改。
