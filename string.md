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
