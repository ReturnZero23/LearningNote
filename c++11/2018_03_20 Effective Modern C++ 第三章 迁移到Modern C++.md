# 迁移到Modern C++

## 一、区别圆括号和花括号在创建对象的时候

- 对象初始化的时候不使用 “=” 赋值操作符

```cpp
int x = { 0 };

int y{ 0 };

Widget w1;       // call default constructor

Widget w2 = w1;  // not an assignment; calls copy ctor

w1 = w2;         // an assignment; calls copy operator=
``` 
- 这里经常造成初学者的困扰，对于语言内置的类型没有任何的区别，对于自定义的类型，这里有赋值操作符，但是并没有调用拷贝构造函数，而是调用了默认的构造函数。

- 花括号可以用来初始化非静态数据成员，可以用来初始化不可拷贝的对象（不可以使用赋值操作符）。

- 花括号初始化禁止内置类型的“狭隘的隐式转换”（无法完全保留转换之前的数据的转换）

- 初始化一个无参数的对象，要使用花括号，而不能使用圆括号，圆括号的方式会和定义一个成员函数的形式冲突，只能使用花括号的形式。

- 花括号的初始化会导致一下令人困惑的行为：因为新的c++标准中添加了std::initializer, 当初始化的时候新的标准里会优先解析成初始化列表，而不是对应的参数，当一个类的构造函数中包含初始化列表，你使用花括号的初始化时，会优先使用初始化列表的构造函数，即使构造参数中有类型完全符合的构造函数，初始化列表页会根据参数的类型先进行隐式转换，转换成初始化列表中的类型（即使初始化列表中的类型并不是一致的），如果类型定义了转换成某种类型的操作符，并且这种操作符可以和初始化列表中的类型进行隐式转换，这种方式的调用会调用初始化列表的构造而不是拷贝构造函数。如果构造的花括号中的类型完全没有办法和初始化列表中的类型进行转换时，函数的构造会回退到构造函数的重载。编译器选择会优先选择初始化列表初始化，即使需要进行隐式转换。

- 调用空的初始化列表的构造函数需要这样调用，否则会认为是空参数的构造函数

```cpp
Widget w4({}); 
Widget w5{{}};
```

- 在有些时候，对象的初始化时使用圆括号和花括号表示的行为是完全不一样的。文章推荐当你在实现自己的构造函数时尽量避免花括号和圆括号的初始化会有不同语义的实现。

```cpp
std::vector<int> v1(10, 20); // use non-std::initializer_list // ctor: create 10-element
                                 // std::vector, all elements have
                                 // value of 20
std::vector<int> v2{10, 20}; // use std::initializer_list ctor: // create 2-element std::vector,
                                 // element values are 10 and 20
```

## 二、使用 nullptr 而不是 0 和 NULL

- 在代码中表示空指针类型，不使用 0 或者是 NULL 因为0的本质类型是 int, 而 NULL 的类型不太明确，但是NULL类型绝对不是指针类型。当一个类型定义了包含 int 类型和 void* 类型参数的函数，当你想你调用空指针类型的函数时，你使用0和 NULL 调用函数都会发生错误。

- nullptr 实际上也不是一个指针类型，nullptr 的实际类型是 std::nullptr_t ,nullptr_t 可以隐式转换为所有的原生指针类型。指向不同类型的指针也可以进行重载函数，当函数包含多个指针类型的重载，使用 nullptr 作为参数进行调用，编译会报错，

```cpp
#include <iostream>
using namespace std;
class a{
public:
    void print(int* a){
        cout << "intptr" << endl;
    };
    void print(float* b){
        cout << "floatptr" << endl;
    };
};

int main(){
    a Classa;
    int intnum = 10;
    float floatnum = 10.0;
    int * intptr = &intnum;
    float * floatptr = &floatnum;
    Classa.print(intptr);
    Classa.print(floatptr);
    //Classa.print(nullptr); // error "overloaded 'print(std::nullptr_t)' is ambiguous"
    return 0;
}
```

## 三、使用 alias 声明而不是 typedefs

- 在现代c++的语法中会经常用到 
```cpp
std::unique_ptr<std::unor dered_map<std::string,std::string>>
```
当你用到这么长的类型名的时候你应该使用类型别名，c++ 中提供了
```cpp

//c++98
typedef
     std::unique_ptr<std::unordered_map<std::string, std::string>>
     UPtrMapSS;

//c++11
using UPtrMapSS =
     std::unique_ptr<std::unordered_map<std::string, std::string>>;
```
- 当类型别名的定义中包含函数指针的时候，定义的名字通常会淹没在类型中，使用using的形式会更加的直观。
```cpp
// FP is a synonym for a pointer to a function taking an int and 
// a const std::string& and returning nothing
typedef void (*FP)(int, const std::string&);    // typedef

// same meaning as above
using FP = void (*)(int, const std::string&);   // alias
                                                // declaration
```

- 定义下列模板类型的别名时，只能使用using，而typedef的定义就显得不直观
```cpp
//using 
template<typename T>                        // MyAllocList<T> 
using MyAllocList = std::list<T, MyAlloc<T>>;
MyAllocList<Widget> lw;

////////////////////////////////////////////
template<typename T>                        // MyAllocList<T>::type 
struct MyAllocList {                        // is synonym for
    typedef std::list<T, MyAlloc<T>> type;  // std::list<T,
};
MyAllocList<Widget>::type lw;
```

- 你可以在模板中直接使用 using 定义的类型的别名，但是使用 typedef 定义的别名需要使用 typename 来标识这是一个类型。

- 去除修饰符和增加左值引用的模板
```cpp
std::remove_const<T>::type     // C++11: const T → T
std::remove_const_t<T>         // C++14 equivalent
std::remove_reference<T>::type // C++11: T&/T&& → T
std::remove_reference_t<T>     // C++14 equivalent
std::add_lvalue_reference<T>::type // C++11: T → T&
std::add_lvalue_reference_t<T> // C++14 equivalent
```
## 四、使用 delete 标识符表示哪些未定义的私有函数
- 旧的让哪些自动生成的函数不可访问的方式，需要在连接的时候才能判断出其他地方是否调用了不该调用的函数。使用delelte 能够在编译时刻就发现语法错误。

- 能够使用 delete 标识符去禁止一些类型的重载

- 能够使用 delete 标识符去禁止一些模板特化类型

- 模板特化必须在命名空间的作用域中，不能再类的作用域中！删除模板的特化类型的可以写在函数的外部
```cpp
class Widget {
   public:
    ...
    template<typename T>
    void processPointer(T* ptr) {... }
    ...
};

template<>
void Widget::processPointer<void>(void*) = delete;
```

## 五、声明重写函数使用override

- 保证函数是重写父类的函数

- 函数结尾加 & 表示是左值对象调用的，加 && 表示是右值对象调用的

- 重写父类函数的要去特别的严格，如果万一漏掉某个条件，就会变成是重载函数

