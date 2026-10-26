# C++ 基础知识整理｜秋招复习笔记（扩展版）

> 适用：C++ / 嵌入式软件开发笔试、面试；示例建议使用 C++17。
> 复习路线：const、static、指针与基础运算 → 类与对象 → 类型与作用域 → 引用和移动语义 → 内存管理与智能指针 → 继承多态 → 模板。

## 目录

1. [const：常量与只读限定](#1-const常量与只读限定)
2. [static：静态存储与类共享成员](#2-static静态存储与类共享成员)
3. [指针：地址、解引用、数组、空指针](#3-指针地址解引用数组空指针)
4. [inline：内联函数和 ODR](#4-inline内联函数和-odr)
5. [volatile：易变对象和硬件寄存器](#5-volatile易变对象和硬件寄存器)
6. [assert：调试断言](#6-assert调试断言)
7. [sizeof：对象及类型占用的字节数](#7-sizeof对象及类型占用的字节数)
8. [strlen：C 风格字符串长度](#8-strlenc-风格字符串长度)
9. [联合体 union 与匿名联合体](#9-联合体-union-与匿名联合体)
10. [构造函数、析构函数与初始化列表](#10-构造函数析构函数与初始化列表)
11. [explicit 与自定义类型转换](#11-explicit-与自定义类型转换)
12. [友元函数与友元类](#12-友元函数与友元类)
13. [using、命名空间与作用域解析符](#13-using命名空间与作用域解析符)
14. [enum 与 enum class](#14-enum-与-enum-class)
15. [decltype 与 auto](#15-decltype-与-auto)
16. [左值、右值与右值引用](#16-左值右值与右值引用)
17. [拷贝构造与移动构造](#17-拷贝构造与移动构造)
18. [浅拷贝、深拷贝与重复释放](#18-浅拷贝深拷贝与重复释放)
19. [成员初始化列表与 std::initializer_list](#19-成员初始化列表与-stdinitializer_list)
20. [封装、继承与多态](#20-封装继承与多态)
21. [虚继承与纯虚函数](#21-虚继承与纯虚函数)
22. [类模板与成员模板](#22-类模板与成员模板)
23. [malloc、calloc、realloc、alloca 与 new/delete](#23-malloccallocreallocalloca-与-newdelete)
24. [智能指针](#24-智能指针)
25. [秋招高频对照与自测](#25-秋招高频对照与自测)

---

## 1. const：常量与只读限定

**核心：** `const` 表示不能通过被 const 限定的对象或访问路径修改对应值；它并不意味着数据在任何情况下都绝对不会发生变化。

```cpp
const int a = 10;
// a = 20;                  // 错误：不能修改 const 对象
int n = 5;
const int& r = n;           // 通过 r 不能修改 n
n = 8;                     // 通过原变量仍可以修改
```

### 1.1 const 与指针（重点）

读法：**从变量名向外读，重点看 const 修饰的是指针本身还是指向的内容。**

```cpp
int a = 10, b = 20;
const int* p1 = &a;         // 指向 const int 的指针；不能通过 p1 改值
int const* p2 = &a;         // 与 p1 一样
int* const p3 = &a;         // 常量指针；p3 不能指向别处
const int* const p4 = &a;   // 指向常量的常量指针

p1 = &b;                   // 正确：p1 可改指向
// *p1 = 30;               // 错误：不能通过 p1 修改
*p3 = 30;                  // 正确：能通过 p3 修改 a
// p3 = &b;                // 错误：p3 自身不可重新赋值
```

| 写法 | 能修改指向的值？ | 能修改指向？ |
|---|---|---|
| `const int* p` | 不可以通过 p | 可以 |
| `int* const p` | 可以 | 不可以 |
| `const int* const p` | 不可以通过 p | 不可以 |

### 1.2 const 成员函数

```cpp
class Sensor {
    int value_ = 42;
public:
    int read() const { return value_; }  // 不允许修改普通非 mutable 成员
};
const Sensor s;
int result = s.read();               // 正确
```

`int read() const` 中的 `const` 修饰隐式 `this` 指向的对象，允许从 const 对象调用。`mutable` 成员是例外；`const` 成员函数也不能简单视为线程安全保证。

### 1.3 const 与引用、形参

```cpp
void print(const int& x);           // 不复制大对象时尤其常用
int x = 10;
const int& cr = x;                  // 可绑定左值
const int& temp = 42;               // 也可绑定临时值
```

**面试易错：** `const int*` ≠ `int* const`；成员函数末尾的 `const` 与返回类型前面的 `const` 作用不同。

---

## 2. static：静态存储与类共享成员

`static` 的含义取决于使用位置：可以改变对象的**存储期、链接属性**，也可以声明类的**静态成员**。

### 2.1 函数内的 static 局部变量

```cpp
#include <iostream>
void count() {
    static int n = 0;       // 只初始化一次，具有静态存储期
    ++n;
    std::cout << n << ' ';
}
int main() {
    count(); count(); count();  // 输出：1 2 3
}
```

普通局部变量每次进入函数通常重新创建；静态局部变量在整个程序运行期间存在，**作用域仍然是函数内部**。C++11 起局部静态变量首次初始化具有线程安全保证（但对它后续的无同步读写不自动线程安全）。

### 2.2 文件作用域的 static

```cpp
static int counter = 0;      // 当前翻译单元内的内部链接
static void helper() {}      // 此处函数也只有内部链接
```

在现代 C++ 中，也常用**匿名命名空间**让名称仅在当前翻译单元可见。

### 2.3 类中的 static 成员

```cpp
#include <iostream>
class Device {
public:
    static int total;
    Device() { ++total; }
};
int Device::total = 0;  // C++17 之前普通静态数据成员常用类外定义
int main() {
    Device a, b;
    std::cout << Device::total; // 2
}
```

静态数据成员由该类对象**共享**，不是每个对象各一份。静态成员函数不具有 `this` 指针，因此不能在没有对象的情况下直接访问非静态成员。

```cpp
class A {
    int x = 3;
public:
    static void show() {
        // std::cout << x; // 错误：没有具体对象
    }
};
```

C++17 可以写 `inline static int total = 0;`，通常不需要另外写类外定义。

**面试易错：** `static` 局部变量的**作用域仍是局部**；全局 `static` 和类内 `static` 不是同一种用途。

---

## 3. 指针：地址、解引用、数组、空指针

**核心：** 指针是保存地址的对象，`&` 取地址，`*` 解引用；使用指针必须注意有效性、生命周期和内存所有权。

```cpp
#include <iostream>
int main() {
    int a = 10;
    int* p = &a;            // p 存 a 的地址
    std::cout << *p;        // 10
    *p = 20;                // 修改 a
    std::cout << a;         // 20
}
```

### 3.1 指针与引用对照

| 对比 | 指针 | 引用 |
|---|---|---|
| 声明 | `int* p = &a;` | `int& r = a;` |
| 能否为空 | 可以，如 `nullptr` | 正常引用必须绑定有效对象 |
| 能否重新指向 | 可以 | 不可以重新绑定 |
| 访问对象 | `*p`、`p->field` | `r`、`r.field` |

### 3.2 nullptr 与悬空指针

```cpp
int* p = nullptr;          // 空指针，解引用前要检查
int* q = new int(7);
delete q;
q = nullptr;               // 避免 q 自身成为悬空指针
```

注意：`delete` 后再把 **q** 置空，不会自动修复任何其他别名指针。不要对悬空指针解引用、重复 `delete`，也不要返回局部变量的地址。

### 3.3 数组和指针

```cpp
int arr[3] = {10, 20, 30};
int* p = arr;             // 此表达式上下文中 arr 转换为首元素指针
// p[1] 与 *(p + 1) 都为 20
```

数组和指针**并非同一类型**。例如 `sizeof(arr)` 是整个数组占用的字节数，`sizeof(p)` 是指针对象本身的字节数。

### 3.4 指针运算与二级指针

```cpp
int a = 5;
int* p = &a;
int** pp = &p;
**pp = 9;                 // a 变为 9
```

`p + 1` 根据所指向的元素类型移动一个元素的距离，不是简单增加 1 字节；只应在合法数组范围内做指针运算，不能越界解引用。

### 3.5 函数传指针

```cpp
void increase(int* p) {
    if (p != nullptr) ++(*p);
}
int x = 3;
increase(&x);              // x 为 4
```

**面试重点：** `*p` 与 `&a`、指针与数组、`sizeof(数组)` 与 `sizeof(指针)`、野指针与悬空指针、`const int*` 与 `int* const`、`new/delete` 配对。

---

## 4. inline：内联函数和 ODR

**核心：** `inline` 最初用于建议编译器内联展开，但**不保证**机器码一定展开。现代 C++ 中它还影响函数/变量可以跨翻译单元定义的规则（ODR）。

```cpp
inline int add(int a, int b) {
    return a + b;
}
int x = add(2, 3); // 5
```

### 4.1 类内定义成员函数

```cpp
class A {
public:
    int square(int x) const { return x * x; } // 通常隐式 inline
};
```

通常把内联函数定义放在头文件，让包含它的翻译单元看到一致的定义；函数是否真的展开取决于编译器优化策略。`inline` 不等于速度一定更快，过度展开可能使代码体积变大。

### 4.2 inline 变量（C++17）

```cpp
struct Config {
    inline static int baud = 115200;
};
```

**面试易错：** `inline` 不等于宏 `#define`；前者有类型检查和正常作用域，后者是预处理文本替换。对于 MCU，需兼顾代码大小与性能。

---

## 5. volatile：易变对象和硬件寄存器

`volatile` 告诉编译器，对这个对象的访问具有需要保留的可观察效果，不能仅按普通变量那样消除或合并某些访问。常见场景是**内存映射硬件寄存器**。

```cpp
#include <cstdint>
// 示例地址纯属演示；必须以芯片参考手册为准
constexpr std::uintptr_t REG_ADDR = 0x40000000u;
volatile std::uint32_t* reg =
    reinterpret_cast<volatile std::uint32_t*>(REG_ADDR);
// 在确认地址、访问权限及平台映射有效后，才可使用 *reg
```

### 常见误区

- `volatile` **不等于线程安全**，不保证原子性，也不能替代互斥锁或 `std::atomic`。
- `volatile int count; count++;` **不是原子操作**，可能包含多步读改写。
- 对于中断和主循环共享数据，`volatile` 是否适用、是否还需要临界区或原子操作，取决于硬件和平台内存模型。
- 在现代 C++ 多线程代码中，跨线程同步主要用 `std::atomic` 和其他同步原语。

```cpp
#include <atomic>
std::atomic<int> counter{0};
// 不同线程可以安全使用 counter.fetch_add(1);
```

**面试高频问法：** `volatile` 能保证原子性吗？**不能。** 能否保证多线程同步？**不能。**

---

## 6. assert：调试断言

`assert` 是 `<cassert>` 中的宏，用于检查程序员认为**必须成立的内部条件**。

```cpp
#include <cassert>
int divide_positive(int a, int b) {
    assert(b > 0);          // 调试期检查前置假设
    return a / b;
}
```

如果断言表达式求值为 false，通常会报告诊断信息并终止程序。定义 `NDEBUG` 后，标准 `assert` 检查将被禁用，其表达式**也不会执行**。

```cpp
int x = 0;
// assert(++x == 1);       // 不要依赖断言中的副作用：NDEBUG 时不会自增
```

### 适用与不适用

- ✅ 检查内部不变量、算法预期和调试假设。
- ❌ 不要用它代替用户输入校验、运行时错误处理或可靠性检查。
- ❌ 不要将必要的设备初始化、计数更新等操作写入 `assert` 表达式。

对于嵌入式产品，断言触发后的处理方式也需要结合日志、看门狗和故障恢复策略设计。

---

## 7. sizeof：对象及类型占用的字节数

**核心：** `sizeof` 是 C++ 运算符，结果类型为 `std::size_t`，单位是**字节**；一般不求值其操作数（变长数组不是标准 C++ 特性）。

```cpp
#include <iostream>
int main() {
    int arr[5] = {1, 2, 3, 4, 5};
    int* p = arr;
    std::cout << sizeof(arr) << '\n'; // 5 * sizeof(int)
    std::cout << sizeof(p) << '\n';   // 指针本身大小，常见 64 位环境为 8
    std::cout << sizeof(arr) / sizeof(arr[0]); // 5
}
```

### 7.1 sizeof 不是计算字符串长度

```cpp
char s[] = "abc";
// sizeof(s) == 4：含末尾 '\0'
// strlen(s) == 3：不含末尾 '\0'
```

### 7.2 结构体对齐

```cpp
struct S {
    char c;
    int n;
};
// sizeof(S) 通常为 8，但由目标 ABI 的类型大小和对齐规则决定
```

### 7.3 数组作为函数形参会退化

```cpp
void f(int arr[]) {
    // 这里 arr 实际是 int* 形参，sizeof(arr) 是指针大小
}
```

**面试易错：** `sizeof(char) == 1` 按定义成立；`sizeof(int)`、指针大小、结构体大小均不应脱离具体平台假定。

---

## 8. strlen：C 风格字符串长度

**核心：** `std::strlen` 在 `<cstring>` 中，统计从指针位置开始到首个 `\0` **之前**的字符数，**不包含终止零字节**。

```cpp
#include <iostream>
#include <cstring>
int main() {
    char s[] = "hello";
    std::cout << sizeof(s) << '\n';        // 6
    std::cout << std::strlen(s) << '\n';    // 5
}
```

### 8.1 遇到中间的零字符就停止

```cpp
char s[] = {'A', 'B', '\0', 'C', '\0'};
// sizeof(s) == 5
// std::strlen(s) == 2
```

### 8.2 典型危险写法

```cpp
char bytes[3] = {'a', 'b', 'c'};
// std::strlen(bytes);  // 错误：没有 '\0'，可能越界读取，未定义行为
```

- `strlen` 需要遍历直到 `\0`，时间复杂度 O(n)。
- 对 C 风格字符串应确保缓冲区中确实存在可访问的终止零字符。
- `std::string::size()` 则返回字符串对象管理的长度，可包含中间 `\0`，通常不需要遍历查找。
- `strlen` 返回类型是 `std::size_t`，不是必然为 `int`。

### 8.3 sizeof 与 strlen 速记表

| 对比 | `sizeof` | `strlen` |
|---|---|---|
| 本质 | 运算符 | 标准库函数 |
| 参数 | 类型或表达式 | 指向以 `\0` 结尾的字符序列的指针 |
| 返回意义 | 字节数 | `\0` 之前的字符个数 |
| 是否包括末尾 `\0` | 对字符数组会计入整个数组 | 不包括 |
| 是否需要遍历内容 | 一般不需要 | 需要 |

**秋招高频：** `char s[] = "abc"; sizeof(s)` 为 4，`strlen(s)` 为 3；但 `const char* p = "abc"; sizeof(p)` 是指针大小。

---

## 9. 联合体 union 与匿名联合体

**核心：** `union` 的非静态数据成员共用同一段存储，同一时刻通常只有一个活动成员。其大小至少要容纳最大的成员，并满足对齐需求。

```cpp
union Data {
    Data() : i(10) {}  // 初始化活动成员 i
    int i;
    double d;
};

static union {         // 命名空间作用域匿名联合体，要求 static
    int globalValue;
    double globalDouble;
};

int main() {
    Data u;
    // u.i 为 10
    ::globalValue = 20;

    union {            // 块作用域匿名联合体
        int localValue;
        double localDouble;
    };
    localValue = 30;
}
```

- 有名联合体通过 `u.i` 访问。
- 匿名联合体成员名称直接进入其外围作用域；同名时局部名称会隐藏全局名称。
- `::x` 从全局命名空间查找 `x`。
- **陷阱：** 不要随意写入一个成员后读取另一不相关成员来做类型转换；这通常不具可移植性。

## 10. 构造函数、析构函数与初始化列表

**构造函数**与类同名，无返回类型，创建对象时执行；**析构函数**写作 `~类名()`，对象生命周期结束时用于清理资源，不接收参数。

```cpp
#include <iostream>
class Student {
    int age;
public:
    Student() : age(18) { std::cout << "无参构造\n"; }
    Student(int a) : age(a) { std::cout << "有参构造\n"; }
    ~Student() { std::cout << "析构\n"; }
    int getAge() const { return age; }
};

int main() {
    Student s1;      // 无参构造
    Student s2(20);  // 有参构造
} // 先析构 s2，再析构 s1
```

### `A(int a) : _a(a) {}` 逐项理解

```cpp
class A {
    int _a;
public:
    A(int a) : _a(a) {} // 参数 a 直接初始化成员 _a
};
```

- `A(int a)` 是有参构造函数；冒号后是**成员初始化列表**；`{}` 是构造函数体。
- `class` 默认成员访问权限为 `private`；对外构造通常放在 `public:` 下。
- 若用户声明了有参构造函数，编译器通常不再隐式声明无参构造函数。
- 类成员真正的初始化顺序是**声明顺序**，与初始化列表的书写顺序无关。
- `const` 成员、引用成员和没有可用默认构造函数的类成员，必须在初始化阶段正确初始化。

## 11. explicit 与自定义类型转换

`explicit` 限制构造函数及转换函数参与隐式转换。

```cpp
struct A {
    A(int) {}
    operator bool() const { return true; }
};
struct B {
    explicit B(int) {}
    explicit operator bool() const { return true; }
};
```

`operator bool() const` 是**类型转换成员函数**：将当前对象转换成 `bool`；没有单独的返回类型，末尾 `const` 表示不修改当前对象。

| 写法 | A | B | 原因 |
|---|---|---|---|
| `T x(1);` | ✅ | ✅ | 直接初始化 |
| `T x = 1;` | ✅ | ❌ | 复制初始化不能使用 explicit 构造转换 |
| `T x{1};` | ✅ | ✅ | 直接列表初始化 |
| `T x = {1};` | ✅ | ❌ | 复制列表初始化选择 explicit 构造函数则报错 |
| `T x = (T)1;` | ✅ | ✅ | 显式转换 |
| `doT(1);` | ✅ | ❌ | 按值实参需要隐式转换 |
| `if (x) {}` | ✅ | ✅ | 条件上下文允许 explicit bool 转换 |
| `bool v(x);` | ✅ | ✅ | 直接初始化 |
| `bool v = x;` | ✅ | ❌ | 普通隐式转换受限 |
| `bool v = static_cast<bool>(x);` | ✅ | ✅ | 显式转换 |

**特别记忆：** `if (b)` 可以，而 `bool x = b;` 不可以，原因是 **按语境转换为 bool** 与普通隐式转换的规则不同。

## 12. 友元函数与友元类

`friend` 是有针对性地授权外部函数或其他类访问当前类的 `private/protected` 成员。

```cpp
#include <iostream>
class Student {
    int age = 20;
public:
    friend void print(const Student& s); // 友元函数
    friend class Teacher;                // 友元类
};
void print(const Student& s) {
    std::cout << s.age << '\n';
}
class Teacher {
public:
    void show(const Student& s) { std::cout << s.age << '\n'; }
};
```

- 友元函数**不是**该类的成员函数；调用 `print(s)`，不是 `s.print()`。
- 友元类的成员函数可访问授权类的非公有成员。
- 友元关系**不对称、不传递、不被继承**。
- 常见用途：`operator<<` 等需要访问内部状态的非成员运算符函数；但不是所有运算符重载都必须使用友元。

## 13. using、命名空间与作用域解析符

```cpp
#include <iostream>
namespace A { int x = 10; }
int x = 100;

using namespace A;     // using 指令：A 中名称参与未限定名称查找
using std::cout;       // using 声明：引入指定名称
using Integer = int;   // 类型别名声明（不是 using 指令）

int main() {
    int x = 20;
    cout << x << ' ';    // 20：局部 x
    cout << ::x << ' ';  // 100：全局 x
    cout << A::x;       // 10：A 命名空间的 x
}
```

- `::x`：全局命名空间里的 `x`。
- `A::x`：命名空间或类 `A` 中的 `x`。
- `obj.x`：通过对象访问成员；`ptr->x`：通过指针访问成员。
- `void A::show() {}`：在类外定义 `A` 的成员函数。
- `using namespace A` 不会把所有成员真的复制进当前作用域。多命名空间同名成员可能产生歧义。
- **工程建议：** 头文件中避免写 `using namespace std;`。

## 14. enum 与 enum class

```cpp
#include <cstdint>
enum Color { RED, GREEN = 5, BLUE }; // 值为 0、5、6
enum class MotorState : std::uint8_t {
    STOP = 0, FORWARD = 1, REVERSE = 2
};

int main() {
    int a = GREEN; // 普通 enum 可以隐式转整数
    MotorState s = MotorState::FORWARD;
    auto value = static_cast<std::uint8_t>(s);
    // int b = MotorState::STOP; // 错误：enum class 不隐式转 int
}
```

| 项目 | enum | enum class |
|---|---|---|
| 枚举项所在作用域 | 所在外围作用域 | 枚举自身作用域 |
| 隐式转换为整数 | 通常允许 | 不允许 |
| 调用示例 | `RED` 或 `Color::RED` | `MotorState::STOP` |
| 指定底层类型 | 支持（C++11） | 支持，默认 `int` |

**嵌入式应用：** 用 `enum class Command : std::uint8_t` 定义 UART 命令字、设备状态，增强类型安全。

## 15. decltype 与 auto

`auto` 从初始化表达式推导类型；`decltype(expr)` 按特定规则确定表达式的类型，**不会求值**。

```cpp
#include <utility>
int main() {
    const int a = 10;
    auto b = a;       // int：通常丢弃顶层 const
    decltype(a) c = 20; // const int

    int x = 5;
    decltype(x) y = 10;   // int
    decltype((x)) z = x;  // int&：括号内是左值表达式
    z = 30;               // x 也变成 30

    decltype(x++) k = 1;  // 不执行 x++
    decltype(std::move(x)) r = std::move(x); // int&&
}
```

最重要的规则：

- `decltype(变量名)`：若是未加额外括号的 id-expression，则得到声明类型。
- 一般表达式若是**左值**，得到 `T&`；**将亡值**，得到 `T&&`；**纯右值**，得到 `T`。
- `decltype(x)` 与 `decltype((x))` 可能不同，后者对普通变量 `x` 通常是 `T&`。

## 16. 左值、右值与右值引用

```cpp
#include <utility>
int main() {
    int a = 10;           // 表达式 a 是左值；10 是纯右值
    int& lref = a;        // 左值引用
    const int& cr = 20;   // const 左值引用可绑定右值
    int&& rref = 30;      // 右值引用
    // int&& bad = a;     // 错误：a 是左值
    int&& moved = std::move(a); // std::move(a) 是将亡值
    int& alsoLeft = rref; // 注意：有名字的 rref 表达式是左值
}
```

- `T&` 通常绑定左值；`T&&` 通常绑定右值。
- **右值引用类型的有名变量，其名字表达式仍是左值。**
- `std::move(x)` 本身**不会移动任何数据**，只是将表达式转换为适当的右值表达式；之后是否真的移动资源取决于被调用的函数。
- 模板类型推导语境中的某些 `T&&` 是**转发引用**，不应与普通右值引用混淆。

## 17. 拷贝构造与移动构造

```cpp
#include <iostream>
#include <utility>
class A {
public:
    A() { std::cout << "默认构造\n"; }
    A(const A&) { std::cout << "拷贝构造\n"; }
    A(A&&) noexcept { std::cout << "移动构造\n"; }
    A& operator=(const A&) { std::cout << "拷贝赋值\n"; return *this; }
    A& operator=(A&&) noexcept { std::cout << "移动赋值\n"; return *this; }
};
int main() {
    A a;
    A b = a;              // 拷贝构造
    A c = std::move(a);   // 移动构造
    b = c;                // 拷贝赋值
    b = std::move(c);     // 移动赋值
}
```

**判断口诀：** 是不是在**创建新对象**？是则可能构造；若对象已存在且执行 `=`，通常是赋值。

- 拷贝构造：`A(const A&)`，按类的实现复制成员/资源。
- 移动构造：`A(A&&)`，可以接管源对象的资源，避免大量数据复制。
- `noexcept` 对标准容器进行扩容搬迁时能否优先移动常常很重要。
- `std::move` 并不保证会调用移动构造；如不存在可用的移动构造，则可能选择拷贝。
- C++17 起某些纯右值初始化有**保证的拷贝消除**，不能假设每个看起来像“复制”的写法都会打印拷贝/移动日志。

## 18. 浅拷贝、深拷贝与重复释放

### 浅拷贝的风险

```cpp
struct Bad {
    int* p;
    Bad(int v) : p(new int(v)) {}
    ~Bad() { delete p; }
};

// Bad a(10);
// Bad b = a; // 默认成员拷贝只复制地址，两个对象都会 delete 同一指针：UB
```

- **浅拷贝**：成员指针只复制地址，新旧指针指向同一资源。
- **深拷贝**：新对象独立申请资源，再复制实际数据。
- **Double Free**：同一动态内存重复释放，属**未定义行为**。
- **悬空指针**：资源已经释放，指针仍存着旧地址；读取或再次释放都危险。
- `delete p; p = nullptr;` 只会清空 `p`，**不会自动清空别的别名指针**。

### 深拷贝的核心形式

```cpp
struct Deep {
    int* p;
    explicit Deep(int v) : p(new int(v)) {}
    Deep(const Deep& other) : p(new int(*other.p)) {}
    Deep& operator=(const Deep& other) {
        if (this != &other) {
            int* next = new int(*other.p); // 先分配，成功再替换
            delete p;
            p = next;
        }
        return *this;
    }
    ~Deep() { delete p; }
};
```

以上是教学例子，未提供移动操作。实际项目优先考虑 `std::vector`、`std::string`、`std::unique_ptr`。

**三/五/零法则：** 手动管理资源的类若需要自定义析构、拷贝构造或拷贝赋值，应考虑三法则；加入移动构造和移动赋值称五法则；更好的设计常是用 RAII 成员实现**零法则**。

## 19. 成员初始化列表与 std::initializer_list

名字类似，本质不同：

| 成员初始化列表 | `std::initializer_list<T>` |
|---|---|
| 构造函数语法 `A(int n):x(n){}` | 标准库只读元素视图类型 |
| 用来初始化成员/基类 | 用来接收 `{1,2,3}` 形式的一组元素 |
| 不需额外头文件 | 需要 `<initializer_list>` |

```cpp
#include <initializer_list>
#include <vector>
class Numbers {
    const int id;
    std::vector<int> data;
public:
    Numbers(int i, std::initializer_list<int> values)
        : id(i), data(values) {} // 两个概念同时出现
};

int main() {
    std::vector<int> a(3, 10); // [10,10,10]：数量 + 初始值
    std::vector<int> b{3, 10}; // [3,10]：initializer_list 优先匹配
    Numbers nums(1, {1, 2, 3});
}
```

- 直接初始化：`A a(1)`；直接列表初始化：`A a{1}`。
- 复制初始化：`A a = 1`；复制列表初始化：`A a = {1}`。
- 花括号列表初始化会拒绝很多**窄化转换**，如 `int x{3.14};` 报错。
- 存在可行的 `initializer_list` 构造函数时，列表初始化通常优先选择它。

## 20. 封装、继承与多态

### 封装

通过 `public`、`protected`、`private` 控制成员访问，隐藏内部状态、通过公共接口维护不变量。

```cpp
class Account {
    int balance = 0;
public:
    void deposit(int n) { if (n > 0) balance += n; }
    int getBalance() const { return balance; }
};
```

### 继承

```cpp
class Animal {
public:
    void eat() {}
};
class Dog : public Animal {
public:
    void bark() {}
};
```

| 基类成员访问权限 | public 继承 | protected 继承 | private 继承 |
|---|---|---|---|
| public | public | protected | private |
| protected | protected | protected | private |
| private | 派生类不可直接访问 | 派生类不可直接访问 | 派生类不可直接访问 |

基类先构造，派生类后构造；析构顺序反过来。以上是简化情况，实际还要考虑虚基类和成员对象。

### 多态：`virtual` 与 `override`

```cpp
#include <iostream>
struct Animal {
    virtual void speak() { std::cout << "Animal\n"; }
    virtual ~Animal() = default;
};
struct Dog : Animal {
    void speak() override { std::cout << "Dog\n"; }
};
int main() {
    Dog d;
    Animal* p = &d;
    p->speak(); // Dog：动态绑定
}
```

- **静态多态**：如函数重载、模板实例化。
- **运行时多态**：通过基类指针/引用调用虚函数，由动态类型决定最终覆盖函数。
- `override` 让编译器检查是否真的重写了基类虚函数。
- 多态基类若可能通过基类指针 `delete` 派生类对象，通常应提供**虚析构函数**。基类析构非虚而直接 `delete Base*`（指向派生对象），常见情况下是未定义行为。
- vtable/vptr 是**常见编译器实现方式**，不是 C++ 标准强制规定的对象布局。

## 21. 虚继承与纯虚函数

两者都使用 `virtual`，但用途完全不同。

### 虚继承：处理菱形继承共享基类

```cpp
struct A { int value = 10; };
struct B : virtual A {};
struct C : virtual A {};
struct D : B, C {};

int main() {
    D d;
    d.value = 20;  // 只有一个共享的 A 虚基类子对象
}
```

非虚菱形继承 `D : B, C` 若 `B` 与 `C` 都普通继承 `A`，则 `D` 中有两份 A 子对象，`d.value` 会产生二义性。

**构造考点：** 虚基类由**最派生类**负责初始化；构造时虚基类先于非虚基类；基类及成员的初始化遵循规定顺序。

### 纯虚函数：抽象接口

```cpp
struct Sensor {
    virtual void read() = 0;  // 纯虚函数
    virtual ~Sensor() = default;
};
struct PressureSensor : Sensor {
    void read() override { /* 读取 I2C 压力数据 */ }
};

// Sensor s;          // 错误：Sensor 是抽象类
PressureSensor p;      // 正确
Sensor* base = &p;     // 可以通过基类指针访问
```

- 有纯虚函数且没有最终覆盖实现的类是抽象类，不可实例化。
- 派生类如果没实现纯虚函数，也仍是抽象类。
- 纯虚函数仍可**在类外提供定义**，但定义不会让它变成普通虚函数。
- **纯虚析构函数必须提供定义**，例如 `A::~A() {}`。

| 项目 | 虚继承 | 虚函数 | 纯虚函数 |
|---|---|---|---|
| 主要用途 | 共享虚基类子对象 | 运行时动态分派 | 规定抽象接口 |
| 写法 | `class B : virtual public A` | `virtual void f()` | `virtual void f() = 0` |

## 22. 类模板与成员模板

### 类模板：整个类参数化

```cpp
template <typename T>
class Box {
    T value;
public:
    explicit Box(T v) : value(v) {}
    T get() const { return value; }
};

Box<int> a(10);
Box<double> b(3.14);
```

### 成员模板：类中的某个成员参数化

```cpp
#include <iostream>
class Printer {
public:
    template <typename T>
    void print(const T& x) { std::cout << x << '\n'; }
};

// Printer p; p.print(1); p.print(3.14);
```

### 类模板中还可以定义成员模板

```cpp
template <typename T>
class Holder {
    T value;
public:
    Holder(T v) : value(v) {}
    template <typename U>
    void show(U x) const {
        // T 是类模板参数；U 是成员模板参数
    }
};
```

**类外定义双层模板：**

```cpp
template <typename T>
class A {
public:
    template <typename U> void f(U);
};

template <typename T>
template <typename U>
void A<T>::f(U) {}
```

- 函数模板常能由实参推导类型；类模板通常写 `Box<int>`，C++17 起部分情形支持 CTAD 类模板实参推导。
- **构造函数模板不等同于拷贝构造函数**；类的隐式拷贝构造可能仍存在。

## 23. malloc、calloc、realloc、alloca 与 new/delete

| 函数/操作 | 用途 | 初始化 | 释放 |
|---|---|---|---|
| `malloc(n)` | 申请 n 字节原始存储 | 不清零 | `free` |
| `calloc(count,size)` | 申请 count×size 字节 | 全部字节清零 | `free` |
| `realloc(p,n)` | 调整已有分配块大小 | 新增部分不初始化 | `free` |
| `alloca(n)` | 常见实现中分配临时栈空间 | 不初始化 | 随函数返回失效；非标准 C++ |
| `new T(...)` | 分配内存并构造 T 对象 | 依构造方式 | `delete` |
| `new T[n]` | 构造数组对象 | 依构造方式 | `delete[]` |

```cpp
#include <cstdlib>
int main() {
    int* a = static_cast<int*>(std::malloc(3 * sizeof(int)));
    if (!a) return 1;
    a[0] = 1; a[1] = 2; a[2] = 3;

    // 先使用临时指针：失败时原地址仍有效
    void* tmp = std::realloc(a, 5 * sizeof(int));
    if (!tmp) { std::free(a); return 1; }
    a = static_cast<int*>(tmp);
    a[3] = 4; a[4] = 5;
    std::free(a);
}
```

- `realloc` 成功后**旧指针不能继续使用**，要使用返回的新指针；失败则原块仍然有效。
- 不依赖 `realloc(p,0)` 的平台差异；需要释放时用 `free`。
- `malloc`/`calloc` 不会调用 C++ 类的构造函数，`free` 不会调用析构函数。
- **禁止混用** `malloc/free` 与 `new/delete`。
- `alloca` 不是标准 C++；它的栈分配可能造成栈溢出，不适合大块或不受控大小的分配。
- STM32/裸机/实时任务中注意栈大小、堆碎片、分配失败以及 DMA 缓冲区生命周期。

## 24. 智能指针

头文件：`#include <memory>`。智能指针遵循 RAII：通过对象生命周期管理所拥有的资源。

### unique_ptr：独占所有权

```cpp
#include <memory>
#include <utility>
int main() {
    auto p1 = std::make_unique<int>(10); // C++14
    // auto p2 = p1;                    // 错误：不可拷贝
    auto p2 = std::move(p1);            // 转移所有权
    // 此时 p1 == nullptr，p2 管理原 int
}
```

- `get()`：获取原始指针，不转移所有权；不要对其随意 `delete`。
- `release()`：放弃所有权并返回原指针，调用者要负责后续释放。
- `reset()`：释放或替换当前管理的资源。

### shared_ptr：共享所有权

```cpp
#include <memory>
auto p1 = std::make_shared<int>(10);
auto p2 = p1;  // 引用计数由 1 变为 2
// p1.use_count() == 2
```

- 多个 `shared_ptr` 可以拥有同一对象；最后一个共享所有者离开时，通常释放被管理对象。
- `use_count()` 是当前共享所有者数的观察值，不应用它编写并发同步逻辑。
- **不要**对同一个裸指针分别构造两个独立的 `shared_ptr`，否则可能重复释放。

### weak_ptr：弱观察者、处理循环引用

```cpp
#include <memory>
auto sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp; // 不增加共享所有者计数
if (auto locked = wp.lock()) {
    // locked 有效时可以访问 *locked
}
```

- `weak_ptr` 不拥有对象、不能直接 `*wp`；使用 `lock()` 得到临时 `shared_ptr`。
- `expired()` 可判断观察目标是否已无共享所有者。
- A 持有 B 的 `shared_ptr`，B 又持有 A 的 `shared_ptr`，会出现所有权环；把其中非拥有的关系改为 `weak_ptr` 可打破环。

| 智能指针 | 所有权 | 可复制 | 可移动 | 主要用途 |
|---|---|---|---|---|
| `unique_ptr<T>` | 独占 | 否 | 是 | 默认优选的动态资源所有者 |
| `shared_ptr<T>` | 共享 | 是 | 是 | 确有多人共享生命周期 |
| `weak_ptr<T>` | 无所有权 | 是 | 是 | 观察共享对象、打破循环引用 |

**优先级建议：** 不需要动态分配就直接使用值对象；需要独占动态资源时优先 `unique_ptr`；确有共享生命周期才使用 `shared_ptr`；无所有权的观察关系使用 `weak_ptr`。

---

## 25. 秋招高频对照与自测

### 一页速记表

| 考点 | 一句话记忆 |
|---|---|
| 构造/析构 | 创建时构造，生命周期结束时析构；局部对象通常后构造先析构 |
| 初始化列表 | 直接初始化成员；初始化顺序按**声明顺序** |
| explicit | 禁止某些隐式转换；`if (obj)` 对 explicit bool 有特殊规则 |
| 友元 | 定向授权访问私有成员，不对称、不传递、不继承 |
| using | `using namespace`、`using A::x`、`using X=T` 是不同语法 |
| `::` | 指定全局、命名空间或类作用域 |
| enum class | 作用域明确，不隐式转整数 |
| decltype | `decltype(x)` 与 `decltype((x))` 不一定相同 |
| 右值引用 | `T&&` 可绑定右值；有名右值引用表达式是左值 |
| std::move | 转换值类别，不负责真正搬运资源 |
| 拷贝/移动构造 | 新对象创建；赋值则是已有对象的状态变化 |
| 深/浅拷贝 | 复制资源内容 vs 复制地址；错误共享拥有权会 double free |
| initializer_list | `{}` 有时优先选初始化列表构造函数 |
| 封装 | `private` 数据与受控公共接口 |
| 继承 | 基类先构造、后析构 |
| 多态 | `virtual` + 基类指针/引用 + 重写函数，动态分派 |
| 虚继承 | 菱形结构共享一份虚基类子对象 |
| 纯虚函数 | `virtual f() = 0`，定义抽象接口 |
| 模板 | `template<typename T>` 使类型/成员通用 |
| malloc/calloc | 前者未清零，后者清零；都用 free |
| realloc | 可能移动，必须检查返回值 |
| 智能指针 | unique 独占、shared 共享、weak 不拥有 |

### 10 道自测题（先做题，再看答案）

1. `int x=1; decltype((x)) r=x;` 中 `r` 的类型是什么？
2. `int&& r=10;` 后 `int&& r2=r;` 是否能编译？
3. `vector<int> a(3,2)` 和 `vector<int> b{3,2}` 各包含几个元素？
4. `explicit operator bool()` 允许 `if(obj)` 吗？允许 `bool b=obj` 吗？
5. 基类析构函数非虚，使用 `Base* p=new Derived; delete p;` 有什么风险？
6. `shared_ptr` 的循环引用为什么会泄漏？
7. `realloc` 失败时原内存会被释放吗？
8. 菱形虚继承下，共享虚基类由谁负责初始化？
9. `A a2=a1` 与 `a2=a1` 的区别是什么？
10. 为什么 `Student(int a):age(a){}` 对 `const int age` 正确，但构造函数体内 `age=a;` 不正确？

<details>
<summary>查看答案</summary>

1. `int&`。
2. 不能：表达式 `r` 是左值；可以使用 `std::move(r)`。
3. `a` 有 3 个元素 `[2,2,2]`，`b` 有 2 个元素 `[3,2]`。
4. `if(obj)` 可以；`bool b=obj` 不可以（假设无其他可用转换）。
5. 常见情况下属于未定义行为；应给用于多态删除的基类虚析构函数。
6. 两边互相持有共享所有权，引用计数无法归零；使用 `weak_ptr` 打破非必要的所有权环。
7. 不会，原内存块仍然有效；不要直接覆盖原指针。
8. 最派生类。
9. 前者新建对象并拷贝构造；后者对已有对象拷贝赋值。
10. `const` 成员必须初始化，构造函数体内不能对其赋值。

</details>

### 嵌入式 C++ 额外提醒

- 固定大小/固定生命周期的缓冲区通常优先静态配置；动态内存不是默认选择。
- 若 UART/DMA/中断仍在使用缓冲区，必须确保资源没有因对象析构或 `realloc` 而失效。
- `shared_ptr` 的共享所有权管理有额外开销，硬实时路径需要评估可预测性。
- 多态和模板都可以用于设备驱动抽象，但在资源受限 MCU 上应关注代码尺寸、栈、堆、异常支持和运行时成本。

---

> **最终复习主线：** 先掌握类型、作用域、指针与对象生命周期，再理解构造/析构、资源所有权、拷贝/移动、多态与模板；遇到输出题要沿着“初始化 → 调用 → 状态变化 → 析构”逐步推导。
