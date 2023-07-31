---
title: 【c++】关于继承的一些理解
date: 2023-07-30 15:20:45
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/c++.png
tags:
- 继承
- c++
categories:
- c/c++
ai: ture
---


>学到了cpp的继承和多态这块，感觉知识点还是很细碎繁杂的，所以就写了一篇博客先记录在继承这一块遇到的一些概念性问题。

# :rooster:三种继承

>cpp的三种继承：public（公有继承）、protected（保护继承）和private（私有继承）。

## :rabbit2:Public继承

公有继承是最常见的继承方式，基类的公有成员在派生类中仍然是公有的，保护成员在派生类中变为保护的，私有成员在派生类中是不可访问的。
使用public关键字来表示公有继承。

```cpp
#include <iostream>

// 基类
class Base {
public:
    int publicVar;
    void publicFunc() {
        std::cout << "Base::publicFunc()" << std::endl;
    }

protected:
    int protectedVar;

private:
    int privateVar;
};

// 派生类
class DerivedPublic : public Base {
    // publicVar 是公有的
    // publicFunc() 是公有的
    // protectedVar 变为保护的
    // privateVar 不可访问
};

int main() {
    DerivedPublic derived;
    derived.publicVar = 42; // 可访问
    derived.publicFunc();  // 可访问
    //derived.protectedVar = 10; // 错误，不能访问
    //derived.privateVar = 20; // 错误，不能访问
    return 0;
}
```

## :tiger:Protected继承

保护继承使得基类的公有和保护成员在派生类中变为保护的，私有成员在派生类中是不可访问的。
使用protected关键字来表示保护继承。
```cpp
#include <iostream>

// 基类
class Base {
public:
    int publicVar;
    void publicFunc() {
        std::cout << "Base::publicFunc()" << std::endl;
    }

protected:
    int protectedVar;

private:
    int privateVar;
};

// 派生类
class DerivedProtected : protected Base {
    // publicVar 变为保护的
    // publicFunc() 变为保护的
    // protectedVar 变为保护的
    // privateVar 不可访问
};

int main() {
    DerivedProtected derived;
    //derived.publicVar = 42; // 错误，不能访问
    //derived.publicFunc();  // 错误，不能访问
    //derived.protectedVar = 10; // 错误，不能访问
    //derived.privateVar = 20; // 错误，不能访问
    return 0;
}
```

## :koala:Private继承

私有继承使得基类的公有和保护成员在派生类中变为私有的，私有成员在派生类中是不可访问的。
使用private关键字来表示私有继承。

```cpp
#include <iostream>

// 基类
class Base {
public:
    int publicVar;
    void publicFunc() {
        std::cout << "Base::publicFunc()" << std::endl;
    }

protected:
    int protectedVar;

private:
    int privateVar;
};

// 派生类
class DerivedPrivate : private Base {
    // publicVar 变为私有的
    // publicFunc() 变为私有的
    // protectedVar 变为私有的
    // privateVar 不可访问
};

int main() {
    DerivedPrivate derived;
    //derived.publicVar = 42; // 错误，不能访问
    //derived.publicFunc();  // 错误，不能访问
    //derived.protectedVar = 10; // 错误，不能访问
    //derived.privateVar = 20; // 错误，不能访问
    return 0;
}
```


总结一下：基类private成员在派生类中不管怎么继承`都是不可见`的。
使用关键字`class`时默认的继承方式是`private`，使用`struct`时默认的继承方式是`public`，不过
最好显示的写出继承方式。
在实际运用中一般使用都是public继承，几乎很少使用protetced/private继承，也不提倡使用protetced/private继承，因为protetced/private继承下来的成员都只能在派生类的类里面使用，实际中扩展维护性不强。

# :pig:基类和派生类的赋值转换

>基类和派生类之间的对象赋值转换包括两种类型：向上转换（Upcasting）和向下转换（Downcasting）。

## :dog2:向上转换（Upcasting）

向上转换是将`派生类`的指针或引用赋值给基类指针或引用的过程。由于派生类`包含了基类的部分`，所以向上转换是安全的。

```cpp
#include <iostream>

// 基类
class Base {
public:
    void display() {
        std::cout << "Base class" << std::endl;
    }
};

// 派生类
class Derived : public Base {
public:
    void display() {
        std::cout << "Derived class" << std::endl;
    }
};

int main() {
    Derived derivedObj;
    Base* basePtr = &derivedObj; // 向上转换

    basePtr->display(); // 输出 "Base class"，即使是派生类对象，调用的也是基类的函数

    return 0;
}
```

不要把这个和多态搞混了，实现多态需要使用`虚函数`。这个在后面会提。
之所以会出现子类对象调用父类函数的原因是因为：在向上转换的情况下，将派生类对象的地址赋值给基类指针basePtr，调用basePtr->display()时，编译器会根据`指针的静态类型（即基类指针）`来确定调用的函数。因为静态类型是`Base*`，编译器会查找Base类中是否有名为display的成员函数。即使在派生类Derived中也存在名为display的成员函数，编译器也只会在Base类中查找。

这种行为称为`静态绑定或早绑定`，因为编译器在编译时就确定了要调用的成员函数，不考虑运行时对象的实际类型。

## :dog:向下转换（Downcasting）

向下转换是将基类的指针或引用赋值给派生类指针或引用的过程。由于基类可能不是派生类的对象，因此向下转换需要进行类型检查，确保转换是有效的。
但是实际中很少会用到向下转换。
为了进行向下转换（从基类到派生类），可以使用`dynamic_cast`运算符，但是这种转换要求基类指针指向的对象必须是`派生类的实例`，否则转换会失败并返回nullptr。

```cpp
#include <iostream>

// 基类
class Base {
public:
    virtual void display() {   //虚函数，实现了多态。
        std::cout << "Base class" << std::endl;
    }
};

// 派生类
class Derived : public Base {
public:
    void display() override {
        std::cout << "Derived class" << std::endl;
    }
};

int main() {
    Base baseObj;
    Derived derivedObj;
    Base* basePtr = &derivedObj;

    basePtr->display(); // 因为base里的displya是虚函数，所以输出 "Derived class"，通过基类指针调用派生类的函数

    // 向下转换，几乎不会使用！看看就行了
    Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);
    if (derivedPtr != nullptr) {
        derivedPtr->display(); // 输出 "Derived class"
    } else {
        std::cout << "Failed to downcast" << std::endl;
    }

    return 0;
}

```

# :pig2:重载和隐藏

>重载（Overloading）和隐藏（Hiding）是两种不同的概念，用于描述函数和成员变量在继承关系中的行为。

## :pig:函数重载（Function Overloading）
函数重载是指在`同一个作用域`内，定义了多个函数，它们具有`相同的名称但具有不同的参数列表`。函数重载的目的是为了提供一种简洁和直观的方式来处理不同类型或数量的参数，以实现相似的功能。在进行函数调用时，编译器根据调用的函数参数类型和数量来决定要调用的具体函数。


1. 函数重载发生在`同一个作用域`内。
2. 函数重载根据`函数的参数列表`来区分函数。
3. 返回值类型不会影响函数重载的判定。

```cpp
#include <iostream>

// 函数重载示例
void print(int num) {
    std::cout << "Integer: " << num << std::endl;
}

void print(double num) {
    std::cout << "Double: " << num << std::endl;
}

int main() {
    print(1);      // 调用 void print(int num)
    print(2.1);    // 调用 void print(double num)

    return 0;
}
```


## :mouse2:成员函数隐藏（Member Function Hiding）

成员函数隐藏是指在派生类中定义一个`与基类中成员函数名称相同`的函数，该派生类函数会`隐藏`基类中的同名函数，使得在派生类对象上`无法直接访问`基类的同名函数。


```cpp
#include <iostream>

// 基类
class Base {
public:
    void display() {
        std::cout << "Base class" << std::endl;
    }
};

// 派生类
class Derived : public Base {
public:
    void display() {
        std::cout << "Derived class" << std::endl;
    }
};

int main() {
    Derived derivedObj;
    derivedObj.display(); // 输出 "Derived class"

    // 隐藏了基类的 display() 函数，无法直接通过派生类对象调用基类函数
    // derivedObj.Base::display(); // 通过限定作用域可以访问基类的同名函数

    return 0;
}
```


总结一波：在继承体系中基类和派生类都有`独立`的作用域。子类和父类中有同名成员，子类成员将屏蔽父类对同名成员的直接访问，这种情况叫隐藏，也叫重定义。如果是成员函数的隐藏，`只需要函数名相同`就构成隐藏。


# :mouse:默认成员函数

默认成员函数的本意就是如果我们不写，那么编译器会自动为我们生成。
派生类的成员函数会遵循这样的规则：


>派生类的构造函数负责初始化派生类`自己的成员以及基类的成员`。在派生类的构造函数中，需要在构造函数的初始化列表中`显式调用基类的构造函数`来初始化基类的成员。如果基类没有默认构造函数（无参构造函数），则必须通过派生类构造函数的初始化列表来调用基类构造函数，并传递必要的参数。

```cpp
class Base {
public:
    Base(int x) {
        // Base类构造函数
    }
};

class Derived : public Base {
public:
    Derived(int x, int y) : Base(x) {
        // Derived类构造函数
        // Base类的构造函数通过初始化列表调用，传递参数 x
    }
};
```

>派生类的拷贝构造函数必须调用基类的拷贝构造完成基类的拷贝初始化。

当使用派生类对象初始化另一个派生类对象或将派生类对象传递给函数时，需要调用拷贝构造函数。在派生类的拷贝构造函数中，必须`调用基类的拷贝构造函数`来完成基类成员的拷贝初始化，以确保基类部分正确地复制。

```cpp
class Base {
public:
    Base(const Base& other) {
        // Base类拷贝构造函数
    }
};

class Derived : public Base {
public:
    Derived(const Derived& other) : Base(other) {
        // Derived类拷贝构造函数
        // Base类的拷贝构造函数通过初始化列表调用
    }
};
```


> 派生类的operator=必须要调用基类的operator=完成基类的复制。

类的赋值运算符（operator=）用于将一个对象的值赋给另一个对象。当派生类需要赋值运算符时，应该在派生类的operator=中`调用基类的operator=`来完成基类部分的复制操作。

```cpp
class Base {
public:
    Base& operator=(const Base& other) {
        // Base类赋值运算符
        return *this;
    }
};

class Derived : public Base {
public:
    Derived& operator=(const Derived& other) {
        // Derived类赋值运算符
        Base::operator=(other); // 调用基类的赋值运算符
        // 处理派生类的赋值操作
        return *this;
    }
};
```


>派生类的析构函数会在被调用完成后`自动调用基类的析构函数`清理基类成员。因为这样才能保证派生类对象先清理派生类成员再清理基类成员的顺序。


```cpp
class Base {
public:
    ~Base() {
        // Base类析构函数
    }
};

class Derived : public Base {
public:
    ~Derived() {
        // Derived类析构函数
        // 在析构函数执行完毕后，会自动调用Base类的析构函数
    }
};
```

# :dromedary_camel:友元和静态成员

## :leopard:友元
友元关系`不能继承`，也就是说基类友元不能访问子类私有和保护成员 。

```cpp
class Base {
private:
    int privateData;

protected:
    int protectedData;

public:
    Base() : privateData(1), protectedData(2) {}

    // 友元函数声明
    friend void friendFunction(Base& obj);
};

void friendFunction(Base& obj) {
    // 在友元函数中可以访问Base类的私有和保护成员
    int a = obj.privateData;    // 可以访问私有成员
    int b = obj.protectedData;  // 可以访问保护成员
}

class Derived : public Base {
private:
    int derivedPrivateData;

public:
    Derived() : derivedPrivateData(3) {}
};

int main() {
    Base baseObj;
    friendFunction(baseObj); // 友元函数可以访问Base类的私有和保护成员

    Derived derivedObj;
    friendFunction(derivedObj); // error！友元关系不能继承，不能直接访问Derived类的私有和保护成员
    return 0;
}
```

Base类声明了一个友元函数friendFunction，该函数可以访问Base类的私有和保护成员。当我们创建一个Base类对象baseObj时，friendFunction可以访问baseObj的私有和保护成员。
当创建一个Derived类对象derivedObj时，尝试调用friendFunction(derivedObj)，会抛出异常。因为派生类Derived继承了Base类的友元关系，但是这个继承并不会使friendFunction可以直接访问Derived类的私有和保护成员。`友元关系只对声明为友元的类有效，不会在继承层次中传递`。

## :cat2:静态成员

运行下面的代码：

```cpp
class Person
{
public:
    Person() { ++_count; }
protected:
    std::string _name; 
public:
    static int _count; 
};
int Person::_count = 0;
class Student : public Person
{
protected:
    int _stuNum; 
};
class Graduate : public Student
{
protected:
    std::string _seminarCourse; 
};
void test() {
   
   
        Student s1;
        Student s2;
        Student s3;
        Graduate s4;
        std::cout << " 人数 :" << Person::_count << std::endl;
}

```

得到结果：4。因为基类定义了static静态成员，则整个继承体系里面`只有一个这样的成员`。

# :poodle:菱形虚拟继承

## :dragon:概念
>菱形虚拟继承是指在多重继承中，通过使用虚拟继承来解决由于多个基类共同派生同一个中间基类而导致的二义性和资源浪费问题。

假设有一个基类Animal，两个派生类Bird和Fish，以及一个继承自Bird和Fish的派生类FlyingFish。此时，如果Bird和Fish都派生自Animal，而FlyingFish又同时继承自Bird和Fish，那么会形成一个菱形继承的结构。

```
        Animal
         /    \
       Bird   Fish
         \    /
        FlyingFish
```

在这种情况下，FlyingFish类会同时继承自Bird和Fish，而Bird和Fish都继承自Animal。这样就会导致FlyingFish类中`有两份Animal类的副本`，而这两份副本实际上是`同一个类的不同实例`，造成了资源浪费。

此外，如果Bird和Fish分别定义了相同名称的成员函数或成员变量，那么在FlyingFish中使用这些名称时将会产生`二义性`。

C\+\+提供了虚拟继承（virtual inheritance）的机制。在虚拟继承中，使用`关键字virtual`来声明继承，使得派生类只继承基类的一个`共同基类的单一实例`，而不是每个直接或间接基类都有一份实例。


```cpp
class Animal {
public:
    // Animal 类定义
};

class Bird : virtual public Animal {
public:
    // Bird 类定义
};

class Fish : virtual public Animal {
public:
    // Fish 类定义
};

class FlyingFish : public Bird, public Fish {
public:
    // FlyingFish 类定义
};
```

通过在Bird和Fish类的继承中使用virtual关键字，FlyingFish类将只继承一份Animal类的实例，从而避免了资源浪费和二义性。


>如何做到消除二义性呢？

```cpp
class Animal {
public:
    void eat() {
        cout << "Animal is eating." << endl;
    }
};

class Bird : virtual public Animal {
public:
    void fly() {
        cout << "Bird is flying." << endl;
    }
};

class Fish : virtual public Animal {
public:
    void swim() {
        cout << "Fish is swimming." << endl;
    }
};

class FlyingFish : public Bird, public Fish {
public:
    // FlyingFish 类继承自 Bird 和 Fish
    // Bird 和 Fish 类虚拟继承自 Animal
};
```


在这个类层次结构中，FlyingFish类继承自Bird和Fish，而Bird和Fish类都虚拟继承自Animal。

如果没有使用虚拟继承，FlyingFish类将同时继承来自Bird和Fish的各自实例的Animal部分。这样，在FlyingFish类中调用eat()函数时，会发生二义性，因为存在两个Animal的实例。

>问题代表本质就在于`编译器无法确定应该调用哪个Animal类的eat()函数`，从而导致了二义性。



## :goat:原理

如果不用虚拟继承，那么如果有下列结构，会出现数据冗余。

```cpp
class A
{
public:
    int _a;
};
class B : public A
//class B : virtual public A
{
public:
    int _b;
};
class C : public A
//class C : virtual public A
{
public:
    int _c;
};
class D : public B, public C
{
public:
    int _d;
};
```

D里会有两个A的_a,这样对内存是一种浪费，也造成了二义性。当改用`虚拟继承`后，D就只会继承到一个公用的_a了。

```cpp
int main()
{
    D d;
    d.B::_a = 2;
    d.C::_a = 9;
    std::cout << d._a;  //输出9
    return 0;
}
```

运行下面的代码：
```cpp
int main()
{
    D d;
    d.B::_a = 1;
    d.C::_a = 2;
    d._b = 3;
    d._c = 4;
    d._d = 5;
    std::cout << d._a;
    return 0;
}
```
当不采用虚拟继承时：
打开内存窗口，监视d的地址。

```
0x000000E2788FF968  01 00 00 00  ....
0x000000E2788FF96C  03 00 00 00  ....
0x000000E2788FF970  02 00 00 00  ....
0x000000E2788FF974  04 00 00 00  ....
0x000000E2788FF978  05 00 00 00  ....
```

可以看到存在两个不同的_a。


当采用虚拟继承时：

```
0x0000007AA7CFFB08  18 dc 9f 3d  .??=
0x0000007AA7CFFB0C  f7 7f 00 00  ?...   
0x0000007AA7CFFB10  03 00 00 00  ....      //d._b
0x0000007AA7CFFB14  cc cc cc cc  ????
0x0000007AA7CFFB18  20 dc 9f 3d   ??=
0x0000007AA7CFFB1C  f7 7f 00 00  ?...
0x0000007AA7CFFB20  04 00 00 00  ....      //d._c
0x0000007AA7CFFB24  cc cc cc cc  ????
0x0000007AA7CFFB28  05 00 00 00  ....      //d._d
0x0000007AA7CFFB2C  cc cc cc cc  ????
0x0000007AA7CFFB30  02 00 00 00  ....      //d._a
0x0000007AA7CFFB34  cc cc cc cc  ????
0x0000007AA7CFFB38  cc cc cc cc  ????
```

可以看到，此时只存在唯一的_a了。

当一个类进行虚继承（virtual inheritance），意味着该类继承自一个虚基类。

虚继承的实现依赖于`虚基表（virtual table）和虚基表指针（vptr）`：

>虚基表（virtual table）

虚基表是一个数据结构，包含了虚基类的相关信息，如虚基类的`数据成员偏移量`和虚函数表的指针（如果存在虚函数）。
对于每个虚基类，编译器会生成一个虚基表。虚基表中的条目和顺序与虚基类的声明顺序一致。
>虚基表指针（vptr）

每个含有虚函数或者继承了虚基类的类都会在其对象中包含一个`指向虚基表的指针`，称为虚基表指针（vptr）。
vptr 存储着对应类的虚基表的地址。编译器会在每个对象的`起始位置`（通常是对象的内存布局的开头）存储这个指针。


class D 对象中的内存布局包含`两个虚基表指针`，其中一个指向 class B 的虚基表，另一个指向 class C 的虚基表。通过查询各自的虚基表中所存放的偏移量，再和各自指向虚基表的指针的地址相加，就可以获取到_a的真实位置了！

