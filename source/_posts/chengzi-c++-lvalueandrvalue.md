---
title: 【c++】左值与右值
date: 2023-10-06 14:51:12
cover: 'https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/c++.png'
tags:
- c++
- 左值右值
categories:
- c/c++
ai:  true
---


# :sunny:什么是左值？什么是右值？

左值（L-value）是可以`标识内存位置`的表达式，通常是变量或者对象的名称。右值（R-value）是指`表达式的值`，但不能标识内存位置。在Cpp中，左值可以出现在赋值语句的左边，而右值则通常出现在赋值语句的右边。

>常见左值：变量名或解引用的指针
常见右值：字面常量、表达式返回值

简单的举几个例子：
```cpp
int x = 5;  // 'x' 是左值，因为它标识了内存位置
int y = x;  // 'x' 在赋值语句的右边，是右值

int* ptr = &x;  // 'ptr' 是左值，因为它存储了 'x' 的地址
int z = *ptr;   // '*ptr' 是右值，因为它是 'ptr' 指向的内存位置的值

int getResult() {
    return 42;
}
int result = getResult();  // 'getResult()' 是右值，因为它是函数调用的结果
```

在这个例子中，变量（如 'x'、'ptr'）和表达式（如 '*ptr'、'getResult()'）展示了左值和右值的概念。

# :umbrella:左值引用与右值引用

>无论左值引用还是右值引用，都是给对象取别名。

左值引用就是给左值取别名，右值引用就是给右值取别名。

```cpp
//左值
int* a = new int(10);
int b = 1;
//一些左值引用
int*& ra = a;
int& rb = b;
int& avalue = *a;
// 右值
1;
x+y;
max(x, y);
//一些右值引用
int&& r1 = 10;
double&& r2 = x + y;
double&& r3 = max(x, y);
```

>右值`不能取地址`，但是给右值取别名后，会使得右值被存储到一个特定的位置，并且可以取到该位置的地址。

拿上面代码来说，虽然不能取1的地址，但是当r1引用后，可以对r1取地址，也可以修改r1。如果不想被修改，可以用const int&& r1 去引用。

左值引用只能引用左值，不能引用右值。但是`const`左值引用既可引用左值，也可引用右值。
```cpp
int main()
{
    int& a = 1;   // 编译失败，因为1是右值
    // const左值引用既可引用左值，也可引用右值。
    const int& b = 1;
    const int& c = a;
    return 0;
}
```

右值引用只能右值，不能引用左值。但是右值引用可以`move`以后的左值。
```cpp
int main()
{

 int a = 10;
 int&& b = a;  // message : 无法将左值绑定到右值引用
 // 右值引用可以引用move以后的左值
 int&& c = move(a);
 return 0;   
}
```


# :snowflake:c++11引入右值引用的意义


## :cat:移动构造
左值可以做`参数`，也可以做`返回值`，但是如果函数的返回对象是一个`局部变量`，那么出了函数作用域就会被销毁，此时使用左值引用返回，就会出现问题。那么就只能传值返回。传值返回会导致至少一次拷贝构造。

例如：
```cpp
string returnstring(){
    string str("111");
    return str;
}
```
稍微旧一些的编译器，会先将str拷贝一份，再将拷贝的那一份再拷贝一次给返回后接收的变量。而新的编译器大多只拷贝一次，将str直接拷贝给需要接收的那个变量。如果拷贝是深拷贝，那必然会降低效率。而c\+\+11解决了这一点。

```cpp
#include <iostream>
#include <vector>

class MyString {
private:
    char* data;
public:
    MyString(const char* str) {
        // 构造函数从传入的C字符串中分配内存
        // 复制数据到内部缓冲区
    }

    // 此时可以新增一个移动构造函数，使用右值引用
    MyString(MyString&& other) {
        data = other.data;  // 直接复制指针，而不是复制数据
        other.data = nullptr;  // 清空原对象的指针，避免重复释放内存
    }
    
    // ...
};

int main() {
    MyString str1("Hello, World!");

    // 使用移动构造函数，将str1的内容转移到str2，避免了数据的不必要复制
    MyString str2(std::move(str1));

    // 此时，str1不再持有数据，避免了内存泄漏
    std::cout << "str1: " << str1 << std::endl;  // 这里会输出空字符串

    return 0;
}

```

str1原本是左值，通过move操作以后就变成了右值，进而可以实现移动构造，提高效率。
这种场景，就是需要用右值去引用左值实现移动语义。当需要用右值引用引用一个左值时，可以通过move函数将左值转化为右值。
move函数虽然名字是move，但它不会移动任何东西，它唯一的功能就是`将一个左值强制转化为右值引用`，然后实现移动语义。

```cpp
list<string> lt;
 string str("abc");
// 这里调用的是拷贝构造
 lt.push_back(str);
// 这里调用的是移动构造
 lt.push_back("abc");
 lt.push_back(move(str));
```
上面这串代码也说明了，STL容器插入接口也新增了右值引用的版本。


## :dog:完美转发

>完美转发指能够将参数以`原始的值类别`（左值或右值）传递给其他函数，而不会失去参数的原始属性。这在泛型编程、函数模板和构造函数中较为常见，因为它允许编写通用代码，同时保持参数的值类别。

为了实现完美转发，C\+\+11引入了`右值引用`（&&）和`std::forward`函数。

```cpp
void processValue(int& x) {
    std::cout << "L-value reference: " << x << std::endl;
}

void processValue(int&& x) {
    std::cout << "R-value reference: " << x << std::endl;
}

// 使用完美转发的模板函数
template <typename T>
void forwardValue(T&& x) {
    processValue(std::forward<T>(x));  // 使用std::forward来保持参数的值类别
}

int main() {
    int a = 42;

    forwardValue(a);      // 调用 processValue(int&)
    forwardValue(123);    // 调用 processValue(int&&)

    return 0;
}
```

无论传递给forwardValue的参数是左值还是右值，它都会将参数原封不动地传递给processValue函数，保持参数的原始值类别。

这样就可以在不复制参数的情况下将参数传递给其他函数，从而提高性能，并保持参数的语义。完美转发是实现泛型函数和类的重要技术，特别是在STL（标准模板库）中的容器和算法中经常用到。