# 语言联邦

## C

C 的局限：没有模板、没有异常、没有重载

## Object-Oriented C++

class、封装、继承、多态、动态绑定

## Template C++

——

## STL

容器、迭代器、算法、函数对象

# 构造函数

## 1. **声明为 `explicit` 的构造函数不能用于隐式转换**
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

# 智能指针
## unique_ptr

### 1. 独占式拥有

`unique_ptr` 实现独占式拥有或严格拥有概念。

```c++
#include <iostream>
#include <string>
#include <memory>
    
using namespace std;

int main() {
    unique_ptr<std::string> p1(new std::string("unique"));
    unique_ptr<std::string> p2;
    p1 = p2;
    
    return 0;
}

输出结果为：
Desktop % g++ main.cpp -o main -std=c++11
main.cpp:10:8: error: object of type 'unique_ptr<std::string>' 
(aka 'unique_ptr<basic_string<char, char_traits<char>, allocator<char> > >') 
cannot be assigned because its copy assignment operator is implicitly deleted
```

### 2. 临时右值

当程序试图将一个 `unique_ptr` 赋值给另一个时，如果源 `unique_ptr` 是个临时右值，编译器允许这么做。

```c++
#include <iostream>
#include <string>
#include <memory>

using namespace std;

int main() {
    unique_ptr<string> pu;
    pu = unique_ptr<string>(new string("You"));

    return 0;
}
```

### 3. move()

C++ 有一个标准库函数 `std::move()`，让你能够将一个 `unique_ptr` 赋给另一个。

```c++
#include <iostream>
#include <string>
#include <memory>

using namespace std;

int main() {
    unique_ptr<std::string> ps1, ps2;
    ps1 = unique_ptr<std::string>(new std::string("world"));
    ps2 = std::move(ps1);
    ps1 = unique_ptr<std::string>(new std::string("hello"));
    cout << "ps1: " << *ps1 << endl;
    cout << "ps2: " << *ps2 << endl;

    return 0;
}

输出结果为：
Desktop % ./main              
ps1: hello
ps2: world
```
### 4. get()

```
pointer get() const noexcept;
```

Get pointer: Returns the stored pointer.

The stored pointer points to the object managed by the unique_ptr, if any, or to nullptr if the unique_ptr is empty.

**Notice that a call to this function does not make unique_ptr release ownership of the pointer** (i.e., it is still responsible for deleting the managed data at some point). Therefore, the value returned by this function shall not be used to construct a new managed pointer.

In order to obtain the stored pointer and release ownership over it, call unique_ptr::release instead.

### 5. reset()

销毁正在持有的对象，注意和 release() 的区别。

```
void reset (pointer p = pointer()) noexcept;
```

Reset pointer

Destroys the object currently managed by the unique_ptr (if any) and takes ownership of p.

If p is a null pointer (such as a default-initialized pointer), the unique_ptr becomes empty, managing no object after the call.

To release the ownership of the stored pointer without destroying it, use member function release instead.

### 6. release()

调用 `release()` 会切断 `unique_ptr` 和它原来管理的对象的联系。
`release()` 返回的指针通常被用来初始化另一个智能指针或给另一个智能指针赋值。

> `release()` 之后返回裸指针。

```c++
#include <iostream>
#include <string>
#include <memory>

using namespace std;

int main() {
    unique_ptr<std::string> ps1, ps2;
    ps1 = unique_ptr<std::string>(new std::string("hello"));
    // ps2 = ps1.release(); // 编译不通过
    std::string * ptr = ps1.release();
    cout << "ps1: " << ps1 << endl;
    cout << "ptr: " << *ptr << endl;

    return 0;
}

输出结果为：
hupeidong@ZBMac-C02F31WYM Desktop % ./main              
ps1: 0x0
ptr: hello
```

#### 写在后面

`unique_ptr` 如果是一个局部对象，在生命周期结束时，如果之前通过 `get()` 得到的原始指针还在使用，就会出现崩溃的情况。所以智能指针和原始指针不要混用。

## shared_ptr

### 1. 基本方式

```c++
std::shared_ptr<TestClass> p(new TestClass("hello world", 123));
```

这句话调用了两次内存管理器：
1. 创建 TestClass 实例
2. 创建引用计数对象

### 2. make_shared方式

```c++
std::shared_ptr<TestClass> p = std::make_shared<TestClass>("hello world", 123);
```

#### 2.1 缺点-低效率

```c++
void fiddle(std::shared_ptr<Foo> f);

shared_ptr<Foo> myFoo = make_shared<Foo>();

fiddle(myFoo);
```

函数执行的时候，存在无谓的原子性增加和减小操作，而且操作都使用了完整的内存屏障。使用普通指针可以避免无谓的原子操作。

# 禁止复制

## private

将复制构造函数和赋值运算符的可见性声明为 private，可以防止它们被外部调用。

```c++
class TestClass {
private:
    TestClass(const TestClass &);              // 复制构造函数
    TestClass & operator=(const TestClass &);  // 赋值运算符
 
public:
    ...
};
```

## delete

在 C++11 中，我们可以在复制构造函数和赋值运算符后面加上 delete 关键字来达到这个目的。将带有 delete 关键字的复制构造函数的可见性设为 public 更好，因为在这种情况下调用复制构造函数的话，编译器会给出明确的错误消息。

```c++
class TestClass {
public:
    TestClass(const TestClass &) = delete;              // 复制构造函数
    TestClass & operator=(const TestClass &) = delete;  // 赋值运算符
    ...
};
```

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

## 函数声明后面添加 `throw()`

**异常安全，不允许抛出任何异常**

```
void fun() throw();
```

**可以抛出任何形式的异常**

```
void fun() throw(...);
```

**只能抛出特定类型的异常**

```
void fun() throw(exceptionType);
```

# std::function

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

# string

## 1 字符串拼接

```c++
result = result + s[i];
```

调用内存管理器，分配临时字符串对象保存连接结果。赋值的时候会复制临时字符串的内容，并且最终要删除临时字符串。

相比来说，下面的方式会更好：

```c++
result += s[i];
```

## 2 字符串预留空间

```c++
result.reserve(size);
```

## 3 使用字符串迭代器

```c++
for (auto it = s.begin(), end = s.end(); it != end; ++it)
```

## 4 避免使用 stringstream

`stringstream` 有锁，多线程下性能极差。

`<sstream>` 库定义了三种类：`istringstream`、`ostringstream` 和 `stringstream`，分别用来进行流的输入、输出和输入输出操作。

```c++
#include <iostream>
#include <sstream>

using namespace std;

int main(){
    string s = "1 23 # 4";
    stringstream ss;
    ss << s;
    while (ss >> s){
        cout << s << endl;
    }
    return 0;
}

输出结果为：
hupeidong@ZBMac-C02F31WYM Desktop % ./main                         
1
23
#
4
```

**解决**：`std::to_string` / `base::string_printf` / `base::string_appendf` 等函数。

## 5 避免使用 boost::lexical_cast

`boost::lexical_cast`，容易忘记 `catch`，且大量抛异常有性能开销。

## 6 避免使用 std::stoi

`std::stoi` 会抛异常，容易忘记 `catch`，且大量抛异常有性能开销。

## 7 使用 std::strtol

不抛异常。

**Convert string to long integer**

### Parameters

- str

  C-string beginning with the representation of an integral number.

- endptr

  Reference to an object of type `char*`, whose value is set by the function to the next character in str after the numerical value. This parameter can also be a null pointer, in which case it is not used.

- base

  Numerical base (radix) that determines the valid characters and their interpretation. If this is `0`, the base used is determined by the format in the sequence (see above).

```c++
/* strtol example */

#include <iostream>

using namespace std;

int main() {
    char szNumbers[] = "2001 60c0c0 -1101110100110100100000 0x6fffff";
    char* pEnd;
    long int li1, li2, li3, li4;
    li1 = std::strtol(szNumbers, &pEnd, 10);
    li2 = std::strtol(pEnd, &pEnd, 16);
    li3 = std::strtol(pEnd, &pEnd, 2);
    li4 = std::strtol(pEnd, NULL, 0);
    printf("The decimal equivalents are: %ld, %ld, %ld and %ld.\n", li1, li2,
           li3, li4);
    return 0;
}

输出结果为：
hupeidong@ZBMac-C02F31WYM Desktop % ./main                         
The decimal equivalents are: 2001, 6340800, -3624224 and 7340031.
```

## 8 使用 std::strtoll

不抛异常。

**Convert string to long long integer**

### Parameters

- str

  C-string beginning with the representation of an integral number.

- endptr

  Reference to an object of type `char*`, whose value is set by the function to the next character in str after the numerical value. This parameter can also be a null pointer, in which case it is not used.

- base

  Numerical base (radix) that determines the valid characters and their interpretation. If this is `0`, the base used is determined by the format in the sequence (see [strtol](http://www.cplusplus.com/strtol) for details).

```c++
/* strtoll example */

#include <iostream>

using namespace std;

int main() {
    char szNumbers[] =
        "1856892505 17b00a12b -01100011010110000010001101100 0x6fffff";
    char* pEnd;
    long long int lli1, lli2, lli3, lli4;
    lli1 = strtoll(szNumbers, &pEnd, 10);
    lli2 = strtoll(pEnd, &pEnd, 16);
    lli3 = strtoll(pEnd, &pEnd, 2);
    lli4 = strtoll(pEnd, NULL, 0);
    printf("The decimal equivalents are: %lld, %lld, %lld and %lld.\n", lli1,
           lli2, lli3, lli4);
    return 0;
}

输出结果为：
hupeidong@ZBMac-C02F31WYM Desktop % ./main                         
The decimal equivalents are: 1856892505, 6358606123, -208340076 and 7340031.
```

## 9 避免使用 std::stirng operator +

Operator + 会生成大量的临时 `string` 对象，性能差。可以采用 `string` 的 `append` 方法，支持链式调用。

## 10 Append to string

Append to string

Extends the [string](http://www.cplusplus.com/string) by appending additional characters at the end of its current value:

```c++
// appending to string

#include <iostream>
#include <string>

using namespace std;

int main() {
    std::string str;
    std::string str2 = "Writing ";
    std::string str3 = "print 10 and then 5 more";

    // used in the same order as described above:
    str.append(str2);                          // "Writing "
    str.append(str3, 6, 3);                    // "10 "
    str.append("dots are cool", 5);            // "dots "
    str.append("here: ");                      // "here: "
    str.append(10u, '.');                      // ".........."
    str.append(str3.begin() + 8, str3.end());  // " and then 5 more"
    str.append<int>(5, 0x2E);                  // "....."

    std::cout << str << '\n';
    return 0;
}

输出结果为：
hupeidong@ZBMac-C02F31WYM Desktop % ./main                         
Writing 10 dots here: .......... and then 5 more.....
```

## 11 使用 string_view

使用 `std::string_view`、`boost::string_view` 替换 `const std::string&` 形式，避免 `char*` 转成 `string`，进行内存分配和拷贝。

C++ 中的字符串有两种风格，分别是 C 风格的字符串、`std::string` 字符串。C 风格的字符串性能更高，但是也不方便操作使用。如下示例：

```c++
#include <iostream>
#include <string>

using namespace std;

int main() {
    // C 风格字符串总是以 null 结尾
    char cstr1[] = {'y', 'a', 'n', 'g', '\0'};
    char cstr2[5];
    strcpy(cstr2, cstr1);
    cout << cstr2 << endl;

    // C++ 风格的字符串操作更方便，但是性能不如 C 风格字符串
    std::string str = "yang";
    std::string str2 = str;
    cout << str2 << endl;

    return 0;
}

输出结果为：
hupeidong@ZBMac-C02F31WYM Desktop % ./main
yang
yang
```

C++17 中我们可以使用 `std::string_view` 来获取一个字符串的视图，字符串视图并不真正创建或者拷贝字符串，而只是拥有一个字符串的查看功能。`std::string_view` 比 `std::string` 的性能要高很多，因为每个 `std::string` 都独自拥有一份字符串的拷贝，而 `std::string_view` 只是记录了自己对应的字符串的指针和偏移位置。当我们在只是查看字符串的函数中可以直接使用 `std::string_view` 来代替 `std::string`。

```c++
#include <iostream>
#include <string>
#include <string_view>

using namespace std;

int main() {
    const char* cstr = "hello world";
    std::string_view stringView1(cstr);
    std::string_view stringView2(cstr, 4);
    std::cout << "stringView1: " << stringView1
              << ", stringView2: " << stringView2 << std::endl;

    std::string str = "hello world";
    std::string_view stringView3(str.c_str());
    std::string_view stringView4(str.c_str(), 4);
    std::cout << "stringView3: " << stringView1
              << ", stringView4: " << stringView2 << std::endl;
}

输出结果为：
hupeidong@ZBMac-C02F31WYM Desktop % ./main
stringView1: hello world, stringView2: hell
stringView3: hello world, stringView4: hell
```

# 工厂模式

工厂模式一般分为三种：简单工厂模式、工厂方法模式、抽象工厂模式。

## 1 简单工厂模式

简单工厂模式，工厂类是创建产品的，它决定创建哪一种产品。例如，现有宝马和奔驰两种车需要生产，但是只有一个工厂，且只能在同一时间生产一种车，这时就由工厂决定生产哪种车了。

```c++
#include <iostream>

using namespace std;

enum CarType { BENZ, BMW };

class Car  //车类
{
public:
    virtual void createdCar(void) = 0;
};

class BenzCar : public Car  //奔驰车
{
public:
    BenzCar() {
        cout << "Benz::Benz()" << endl;
    }
    virtual void createdCar(void) {
        cout << "BenzCar::createdCar()" << endl;
    }
    ~BenzCar() {
    }
};

class BmwCar : public Car  //宝马车
{
public:
    BmwCar() {
        cout << "Bmw::Bmw()" << endl;
    }
    virtual void createdCar(void) {
        cout << "BmwCar::createdCar()" << endl;
    }
};

class CarFactory  //车厂
{
public:
    Car* createSpecificCar(CarType type) {
        switch (type) {
            case BENZ:  //生产奔驰车
                return (new BenzCar());
                break;
            case BMW:  //生产宝马车
                return (new BmwCar());
                break;
            default:
                return NULL;
                break;
        }
    }
};

int main(int argc, char** argv) {
    CarFactory carfac;

    Car* specificCarA = carfac.createSpecificCar(BENZ);
    Car* specificCarB = carfac.createSpecificCar(BMW);

    return 0;
}

hupeidong@ZBMac-C02F31WYM Desktop % ./main
Benz::Benz()
Bmw::Bmw()
```

简单工厂模式在每次增加新的车型时，需要修改工厂类，这就违反了开放封闭原则：软件实体（类、模块、函数）可以扩展，但是不可修改。于是，工厂方法模式出现了。

总结：工厂类直接去生产车。

## 2 工厂方法模式

工厂方法模式，不再只由一个工厂类决定哪一个产品类应当被实例化，这个决定权被交给子类去做。当有新的产品（新型汽车）产生时，只要按照抽象产品角色、抽象工厂角色提供的方法来生成即可。那么就可以被客户使用，而不必去修改任何已有的代码。可以看出工厂角色的结构也是符合开闭原则。

```c++
#include <iostream>

using namespace std;

class Car  //车类
{
public:
    virtual void createdCar(void) = 0;
};

class BenzCar : public Car  //奔驰车
{
public:
    BenzCar() {
        cout << "Benz::Benz()" << endl;
    }
    virtual void createdCar(void) {
        cout << "BenzCar::createdCar()" << endl;
    }
    ~BenzCar() {
    }
};

class BmwCar : public Car  //宝马车
{
public:
    BmwCar() {
        cout << "Bmw::Bmw()" << endl;
    }
    virtual void createdCar(void) {
        cout << "BmwCar::createdCar()" << endl;
    }
};

class Factory  //车厂
{
public:
    virtual Car* createSpecificCar(void) = 0;
};

class BenzFactory : public Factory  //奔驰车厂
{
public:
    virtual Car* createSpecificCar(void) {
        return (new BenzCar());
    }
};

class BmwFactory : public Factory  //宝马车厂
{
public:
    virtual Car* createSpecificCar(void) {
        return (new BmwCar());
    }
};

int main() {
    Factory* factory = new BenzFactory();
    Car* specificCarA = factory->createSpecificCar();

    factory = new BmwFactory();
    Car* specificCarB = factory->createSpecificCar();

    return 0;
}

hupeidong@ZBMac-C02F31WYM Desktop % ./main
Benz::Benz()
Bmw::Bmw()
```

总结：抽象的工厂类派生出奔驰车场类，去生产奔驰车。不直接修改抽象的工厂类，这是与简单工厂模式的区别。

## 3 抽象工厂

在工厂方法模式基础上，如果需要生产高配版的奔驰和宝马，那工厂方法模式就有点鞭长莫及了，这就又有抽象工厂模式。

```c++
#include <iostream>

using namespace std;

class Car  //车类
{
public:
    virtual void createdCar(void) = 0;
};

class BenzCar : public Car  //奔驰车
{
public:
    BenzCar() {
        cout << "Benz::Benz()" << endl;
    }
    virtual void createdCar(void) {
        cout << "BenzCar::createdCar()" << endl;
    }
    ~BenzCar() {
    }
};

class BmwCar : public Car  //宝马车
{
public:
    BmwCar() {
        cout << "Bmw::Bmw()" << endl;
    }
    virtual void createdCar(void) {
        cout << "BmwCar::createdCar()" << endl;
    }
};

class HighCar  //高配版车型
{
public:
    virtual void createdCar(void) = 0;
};

class HighBenzCar : public HighCar  //高配奔驰车
{
public:
    HighBenzCar() {
        cout << "HighBenzCarBenz::Benz()" << endl;
    }
    virtual void createdCar(void) {
        cout << "HighBenzCar::createdCar()" << endl;
    }
};

class HighBmwCar : public HighCar  //高配宝马车
{
public:
    HighBmwCar() {
        cout << "HighBmwCar::Bmw()" << endl;
    }
    virtual void createdCar(void) {
        cout << "HighBmwCar::createdCar()" << endl;
    }
};

class Factory  //车厂
{
public:
    virtual Car* createSpecificCar(void) = 0;
    virtual HighCar* createdSpecificHighCar(void) = 0;
};

class BenzFactory : public Factory  //奔驰车厂
{
public:
    virtual Car* createSpecificCar(void) {
        return (new BenzCar());
    }

    virtual HighCar* createdSpecificHighCar(void) {
        return (new HighBenzCar());
    }
};

class BmwFactory : public Factory  //宝马车厂
{
public:
    virtual Car* createSpecificCar(void) {
        return (new BmwCar());
    }
    virtual HighCar* createdSpecificHighCar(void) {
        return (new HighBmwCar());
    }
};

int main(int argc, char** argv) {
    Factory* factory = new BenzFactory();
    Car* specificCar = factory->createSpecificCar();
    HighCar* spcificHighCar = factory->createdSpecificHighCar();

    return 0;
}

hupeidong@ZBMac-C02F31WYM Desktop % ./main
Benz::Benz()
HighBenzCarBenz::Benz()
```

总结：抽象工厂中本身就包含生产普通车和高端车两种方法。

# C++通过类型的字符串名称获取该类型的对象--注册机制

## 1 从字符串获取类对象的思路

为了从字符串获取一个类的对象，一个可行的思路是建立字符串与类的一个映射关系，为了方便操作，我们可以把这种映射关系放在一个map结构中：`std::map<std::string, ?>`。一种理想化的映射是我们直接从字符串获取类型名称`std::map<std::string, CLASS> string_class`，并在后续通过类型名称来构造对象`string_class[str] object`。但是在C++对反射尚没有很好的支持的时候，这种方法貌似也很少有人去尝试。

另外一个常见的思路就是将字符串与可以获得类对象的函数进行映射`std::map<std::string, std::function<void*()>> string_objcreator_map`，如此，我们便可以通过字符串获取可以构造所需要的类对象的函数，实现从字符串到类对象的获取。

```c++
static std::shared_ptr<void> classFromName(std::string str) {
    if (class_map.find(str) == class_map.end()) return nullptr;
    return class_map[str]();
}
```

## 2 一个例子

```c++
#include <iostream>
#include <map>
#include <utility>
#include <string>
#include <functional>
#include <memory>

class Product {
public:
    using Ptr = std::shared_ptr<Product>;
    virtual void makeProduct() = 0;
};

class ProductA : public Product {
public:
    virtual void makeProduct() {
        std::cout << "Make Product A!" << std::endl;
    }
};

static std::map<std::string, std::function<std::shared_ptr<void>()>> class_map;

static std::shared_ptr<void> classFromName(std::string str) {
    if (class_map.find(str) == class_map.end()) return nullptr;
    return class_map[str]();
}

class Register {
public:
    Register(std::string str, std::function<std::shared_ptr<void>()> func) {
        class_map.insert(std::make_pair(str, func));
    }
};

#define REGISTER(classname)                            \
    class Register##classname {                        \
    public:                                            \
        static std::shared_ptr<classname> instance() { \
            return std::make_shared<classname>();      \
        }                                              \
    };                                                 \
    auto temp##classname =                             \
        Register(std::string(#classname), Register##classname::instance);

REGISTER(ProductA)

int main() {
    std::shared_ptr<ProductA> temp1 =
        std::static_pointer_cast<ProductA>(classFromName("ProductA"));
    temp1->makeProduct();
}
```

我们需要在main函数之前将字符串和函数之间的关系准备好，即在main函数之前进行注册。在这里，为了进行注册，利用了全局变量的构造函数在main函数之前调用这个特点。在C++反射机制的实现,还有更多的在main函数之前执行一些动作的方式。

另外这里REGISTER宏中并不是直接定义构造对象的函数，而是定义了一个类，以从中获取对象instance，这样在注册多个类的时候，instance函数不至于冲突。

注：不同的类的instance函数。

## 3 工厂注册

注册机制的真正的应用场景更多是结合了工厂模式。而上面例子里的宏也主要是为了工厂模式中众多的重复代码准备的。

想象一下在有些场景下，有多种类型的对象可以创建，而我们需要做的就是根据需要选择合适的类型进行对象构造。比如，A和B程序员分别写了一个工厂用于A和B产品的生产，但是老板没有拿定主意用哪一个。于是，我们不能直接在代码里面写明用的哪种，而是留一个可选的结构来根据条件来实现。

```c++
// reflection_template.h

#ifndef _REFLECTION_TEMPLATE_H_
#define _REFLECTION_TEMPLATE_H_

#include <iostream>
#include <map>
#include <utility>
#include <string>
#include <functional>
#include <memory>

class Product {
public:
    using Ptr = std::shared_ptr<Product>;
    virtual void makeProduct() = 0;
};

class ProductA : public Product {
public:
    virtual void makeProduct() {
        std::cout << "Make Product A!" << std::endl;
    }
};

class ProductB : public Product {
public:
    virtual void makeProduct() {
        std::cout << "Make Product B!" << std::endl;
    }
};

class Factory {
public:
    using Ptr = std::shared_ptr<Factory>;
    virtual Product::Ptr createProduct() = 0;
};

class ProductAFactory : public Factory {
public:
    virtual Product::Ptr createProduct(/* args */) {
        return std::make_shared<ProductA>();
    }
};

class ProductBFactory : public Factory {
public:
    virtual Product::Ptr createProduct() {
        return std::make_shared<ProductB>();
    }
};

static std::map<std::string, std::function<std::shared_ptr<void>()>> class_map;
static std::shared_ptr<void> classFromName(std::string str) {
    if (class_map.find(str) == class_map.end()) return nullptr;
    return class_map[str]();
}

class Register {
public:
    Register(std::string str, std::function<std::shared_ptr<void>()> func) {
        class_map.insert(std::make_pair(str, func));
    }
};

#define REGISTER(classname)                            \
    class Register##classname {                        \
    public:                                            \
        static std::shared_ptr<classname> instance() { \
            return std::make_shared<classname>();      \
        }                                              \
    };                                                 \
    auto temp##classname =                             \
        Register(std::string(#classname), Register##classname::instance);

REGISTER(ProductAFactory)
REGISTER(ProductBFactory)

#endif

#include "reflection_template.h"

int main() {
    std::shared_ptr<ProductAFactory> tempA =
        std::static_pointer_cast<ProductAFactory>(
            classFromName("ProductAFactory"));
    tempA->createProduct()->makeProduct();
    std::shared_ptr<ProductBFactory> tempB =
        std::static_pointer_cast<ProductBFactory>(
            classFromName("ProductBFactory"));
    tempB->createProduct()->makeProduct();
}
```

在这里实现了两个工厂，这里的工厂类可能被不同的程序员实现在不同的文件中。但是只要在实现完成之后利用REGISTER宏在全局的class_map中进行注册，即可通过classFromName得到构造对象的函数，进行对象构造。