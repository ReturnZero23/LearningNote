# Effective Modern C++ 第一章阅读笔记

## 第一章 类型推导

### 一.类型推导的利弊：

利：在代码中使用类型推导，方便在代码中改变某个对象的类型。

弊：代码会显得难以理解。

### 二.代码推导通常发生在：
auto，decltype，decltype(auto)出现的地方。


### 三.类型T和类型ParamType的推导

通常的函数模板的形式：
```cpp
template<typename T> 
void f(ParamType param);

f(expr);
```
编译器会通过表达式expr的类型来推导ParamType 和 T的类型，通常ParamType的类型会包括一些限定符（例如：const，引用&），像这样：
```cpp
template<typename T> 
void f(const T& param);


int x = 0;

f(x);
```
在这里，ParamType 是 const int &，T 是int。
参数 T 的类型的推导，不仅仅依赖于表达式 expr 的类型，也依赖于 ParamType 的类型。

类型推导的情形主要分三种情况：

```cpp
template<typename T> 
void f(ParamType param);
```

1.ParamType 是指针或者引用类型，但不是通用引用类型
2.ParamType 是通用的引用类型
3.ParamType 既不是指针类型，又不是引用类型

(注：表格里列为ParamType 的类型，行为expr的参数的类型，表格中的内容为推导出来的param的类型(T的类型))

| expr的类型    |  int           |  const int             | const int&             | const int*     | 27         |
| :----------: | :------------: | :--------------------: | :--------------------: | :-------------:| :--------: |
| T&           | int&(int)      | const int&(const int)  | const int&(const int)  |
| const T&     | const int&(int)| const int&(int)        | const int&(int)        |
| T*           | int*(int)      |                        |                        | const int*(int)|
| T&&          | int&(int&)     | const int&(const int&) | const int&(const int&) |                | int&&(int) |
| T            | int(int)       | int(int)               | int(int)               |

当 ParamType 的类型为 T& 时忽略传入参数的引用。

当 ParamType 的类型为 const T& 时忽略传入参数的const 限定符和引用。


当 ParamType 的类型为 T&& 通用引用时，当传入参数为左值时，T 和 param 都被推导为左值引用（保留限定符），当为右值时，和前两种情况类似，T忽略&&的右值引用。

当 ParamType 的类型为 T 时，忽略传入 expr 的所有限定符和引用，表示函数 f 的参数是进行拷贝的，param 是一个全新的对象，传入的 expr 的限定符对其不起作用。

### 四.当函数的模板参数是函数名或者是数组名的时候

注：数组的名通常被认为数组元素类型的指针，因为通常两者在使用中可以互换，但是两者的类型实际上是不一样的。

```cpp
const char name[] = "J. P. Briggs";  // name's type is
                                        // const char[13]
const char * ptrToName = name;       // array decays to pointer
```

数组名：
当 ParamType 的类型为 T 时，参数的类型会退化为指针类型 (const char *),不会保留数组的长度信息
当 ParamType 的类型为 T& 时，参数的类型会变成数组的引用类型（const char (&) [13])，会保留数组的长度信息

函数名：
当 ParamType 的类型为 T 时，参数的类型会退化为函数的指针，
当 ParamType 的类型为 T& 时，参数的类型会退化为函数的引用，
```cpp
void someFunc(int, double);     // someFunc is a function;
                                // type is void(int, double)
template<typename T>
void f1(T param);               // in f1, param passed by value

template<typename T>
void f2(T& param);              // in f2, param passed by ref

f1(someFunc);                   // param deduced as ptr-to-func;
                                // type is void (*)(int, double)
f2(someFunc);                   // param deduced as ref-to-func;
                                // type is void (&)(int, double)
```

### 五.auto 类型推导和模板类型推导的一些差异

1.auto 能够推导 std::initializer 类型，但是模板推导不行.
2.auto 在函数的返回值类型和匿名函数的参数中，无法推导 std::initializer 类型。
3.其他的类型推导都和模板类型推导一致。

### 六.decltype 

1.Decltype 在c++11 中常用来表示是模板函数中返回值得类型依赖于传入参数的类型，
2.Decltype(auto) 在函数模板的推导中表示函数的返回值的类型和返回表达式的类型严格一致，
3.Decltype(auto) 也可以用在其他地方，用来表示定义的类型和要推导的表达式的类型严格一致。

```cpp
decltype(auto) f1() {
    int x = 0; 
    ...
    return x;
}                       // decltype(x) is int, so f1 returns int
decltype(auto) f2() {
    int x = 0;
    ...
    return (x);        // decltype((x)) is int&, so f2 returns int&
}

```
4.decltype((x)),x 是int类型，表达式的类型为int&，

### 七.类型推导的调试的技巧
```cpp
template<typename T>
class TD;

TD<decltype(x)> xType;
```
这种方式会打印出x推导出来的类型

注：书中说使用某些方式得到的类型不一定准确，所以理解c++的类型推导的规则还是十分重要的。
