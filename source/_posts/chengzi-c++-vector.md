---
title: 【c++】vector的简单模拟实现
date: 2023-07-24 20:55:30
tags:
- c语言
- vector
categories:
- c/c++
ai: ture
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/c++.png
---

# :smile:什么是vector？

在C\+\+中，vector是一个非常有用的`容器类`，用于`存储一组元素`，类似于数组。它提供了动态大小的数组功能，使得在运行时可以轻松地添加、删除和访问元素。vector是C++标准模板库（STL）的一部分，因此只需包含`头文件<vector>`即可使用。

vector主要有以下几种作用：
1. 动态大小: vector可以根据需要`动态增长或缩小其大小`。这意味着`不需要在创建时指定其大小`，而是可以在运行时根据需要添加或删除元素。

2. 随机访问: 类似于数组可以使用`索引`来直接访问vector中的元素。

3. 自动内存管理: vector会自动进行`内存管理`，这意味着不用担心内存分配和释放的细节。

4. 元素操作: vector提供了许多用于操作元素的函数，例如在尾部添加元素（push_back()）、删除尾部元素（pop_back()）、插入元素（insert()）、删除指定位置元素（erase()）等等。

5. 与算法的集成: 由于vector是STL的一部分，它可以与STL的其他容器和算法无缝集成，使得数据处理变得非常方便。

下面是一个使用vector的简单示例：

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> myVector; // 创建一个空的整数向量

    myVector.push_back(10); // 在尾部添加元素
    myVector.push_back(20);
    myVector.push_back(30);

    std::cout << "Vector size: " << myVector.size() << std::endl; // 输出元素个数
    std::cout << "Vector capacity: " << myVector.capacity() << std::endl; // 输出容量

    std::cout << "Elements: ";
    for (int i = 0; i < myVector.size(); ++i) {
        std::cout << myVector[i] << " "; // 随机访问元素
    }

    return 0;
}

```
以上代码会输出：

```cpp
Vector size: 3
Vector capacity: 4
Elements: 10 20 30
```

# :laughing:模拟实现

还是用模拟string时的办法，新建一个namespace，重写vector。为方便，默认这里全局展开std。

## :smiley:模版类

在C\+\+中，vector是一个模板类（template class），它通过`模板参数`来指定所存储元素的类型。这意味着vector可以存放各种类型的值。
例如，如果想要存储整数类型的元素：

```cpp
std::vector<int> intVector; // 存放整数类型的元素
```

如果想存储浮点数类型的元素：

```cpp
std::vector<float> floatVector; // 存放浮点数类型的元素
```

简而言之，vector是通过模板参数生成不同类型的容器。
在上面的示例中，vector的行为是相同的，只是存储的元素类型不同。模板类使得我们能够轻松创建可以存储不同类型元素的向量容器。

所以我们在实现的过程中肯定会用到模版来对vector进行处理。
vector容器类通常使用两个指针来表示元素范围，`start和finish`，即有效元素的`起始位置和结束位置`。还使用一个指针来表示内存中可用于`存储元素的结束位置`，这个指针通常称为`endofstorage`，指向存储在内存中的元素数组的指针，指向当前 vector 对象的`内存缓冲区的末尾位置`。

1. start: 指向vector中的`第一个有效元素`的位置。通常，它指向存储在内存中的元素数组的首地址。

2. finish: 指向`vector中最后一个有效元素的下一个位置`。换句话说，它指向存储在内存中的元素数组中的下一个可用位置。

3. endofstorage: 指向可用存储空间的最后一个位置。 

>这三个指针共同定义了vector中存储元素的范围。有效的元素是从start指针开始，一直到finish指针之前的位置（即`左闭右开`区间）。因此，vector中的实际存储元素数量可以通过 `finish - start `来计算。
当添加或删除元素时，指针会`随之更新`，以反映vector中元素的当前范围。如果 vector 的元素数量超过当前内存缓冲区的容量，vector 将会重新分配`更大`的内存空间，并更新 endofstorage 指针为新内存缓冲区的末尾位置。





```cpp
namespace myclass
{
	template<class T>
	class vector
	{
    private:
		iterator _start = nullptr;
		iterator _finish = nullptr;
		iterator _endofstorage = nullptr;       
	public:
		typedef T* iterator;   //迭代器因为很重要，也在这里一并写上了。
		typedef const T* const_iterator;

		iterator begin()
		{
			return _start;
		}

		iterator end()
		{
			return _finish;
		}

		const_iterator begin() const  //在函数后加上 const 关键字，表示这个成员函数是一个常量成员函数（const member function）。这样的设计告诉编译器这个函数不会修改类对象的成员变量。
		{
			return _start;
		}

		const_iterator end() const
		{
			return _finish;
		}
        //有参和无参构造
		vector(size_t n, const T& val = T())
		{
			resize(n, val);
		}


		vector()
		{}

		vector(const vector<T>& v)  
		{
			_start = new T[v.capacity()]; //C++ STL 的标准容器已经对内存管理和容量调整进行了优化，可以正确地处理不同类型的元素，包括自定义类型。所以在这里哪怕T是自定义类型也无所谓。
			for (size_t i = 0; i < v.size(); i++)
			{
				_start[i] = v._start[i];
			}

			_finish = _start + v.size();
			_endofstorage = _start + v.capacity();
		}
        
		~vector()
		{
			if (_start)
			{
				delete[] _start;
				_start = _finish = _endofstorage = nullptr;
			}
		}
        
    }
}
```

因为篇幅较长，所以拎出来单独解释一下这个函数：
```cpp
vector(size_t n, const T& val = T())
```

第一个参数是 size_t n，表示要创建的 vector 的大小，即其中包含的元素数量。第二个参数是 const T& val，表示要用来初始化 vector 中元素的值，它可以作为一个常量引用来`接收内置类型的调用`，同时也可以接收自己写的类，比如上一篇文章重写的string类。

当我们使用类似 vector<int> myVector(10, 1); 这样的调用时，它是有效的，因为这其实是一个`隐式的类型转换`。
1 是一个整数常量，而不是 const int& 类型的常量引用。然而，C++ 允许进行隐式类型转换，可以将 1 转换为 const int& 类型的常量引用，以匹配构造函数的参数类型。当我们传递 1 作为第二个参数时，虽然 1 是一个`临时的常量值`，但它会被绑定到构造函数参数 const T& val 上的常量引用。这样做可以延长 1 的生命周期，确保它在 vector 的构造函数中被正确使用，即在 vector 对象创建过程中可以通过常量引用访问这个值。

这也给我提了个醒，`无参构造`有时候还是蛮重要的，如果缺省了第二个参数，编译器就会自动去调用该类的无参构造，如果此时咱没写无参构造，难免会出现一些问题。


## :smirk:一些其他常用的函数

其实和string类的模拟实现大差不差，就不单独一个一个的写了（懒）。
单独提一下insert和erase函数。
在 C++ STL 中，std::vector 的 insert 函数是用于在指定位置插入元素的成员函数。它`返回一个迭代器`（iterator），指向插入的元素。

```cpp
iterator insert(iterator pos, const T& value);
```

该函数的参数如下：
pos：一个迭代器，表示插入元素的位置。
value：要插入的元素的值。
insert 函数的作用是将新元素插入到指定位置，它会改变 vector 的大小和容量，可能导致内部的数据重新分配。为了方便使用者处理插入后的元素，函数返回一个指向插入元素的迭代器，这样用户就可以继续对新元素进行操作，或者在需要时获取其位置或修改其值。

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> myVector = {1, 2, 3, 4, 5};

    // 在位置 2 插入新元素 100
    auto insertPos = myVector.insert(myVector.begin() + 2, 100);  //因为vector本质上还是顺序存储的，所以+2可以顺利读取到第三个元素，而重写list也这样简单带过就不行了，其底层逻辑是链表，到时候详细讲。

    std::cout << "Inserted Element: " << *insertPos << std::endl; // 输出：100

    return 0;
}
```

insert和erase
```cpp
iterator insert(iterator pos, const T& x)
		{
			assert(pos >= _start && pos <= _finish);

			if (_finish == _endofstorage)
			{
				size_t len = pos - _start;

				size_t newcapacity = capacity() == 0 ? 4 : capacity() * 2;
				reserve(newcapacity);

				// 解决pos迭代器失效问题
				pos = _start + len;
			}

			iterator end = _finish - 1;
			while (end >= pos)
			{
				*(end + 1) = *end;
				--end;
			}

			*pos = x;
			++_finish;

			return pos;
		}

		iterator erase(iterator pos)
		{
			assert(pos >= _start && pos < _finish);

			iterator it = pos + 1;
			while (it != _finish)
			{
				*(it - 1) = *it;
				++it;
			}

			--_finish;

			return pos;
		}
```
在注释了我提到了`迭代器失效`。为什么迭代器会失效？
这里的迭代器失效指的是在向 vector 中`插入新元素`后，之前的迭代器可能会变得无效。
在 vector 中插入元素可能会导致`内部数据重新分配`，这涉及到扩容操作。如果扩容后，原先的内存块不够存放所有元素，vector 就会在新的内存地址上`重新分配内存`，并将原先的元素复制到新的位置。这样一来，原先指向vector 中元素的迭代器就会失效，`因为它们指向了之前的内存地址，而这些地址现在已经不再有效`。

在上述代码中，为了解决这个问题，首先会检查插入新元素后是否需要扩容。如果需要扩容，就会重新分配更大的内存块，并将原先的元素复制到新的位置。然后，为了使原先的迭代器仍然有效，会将 pos 这个迭代器重新设置为插入元素后的位置，即 _start + len。这样一来，之前的迭代器 pos 就仍然指向正确的位置，不会失效。

所以，为了避免迭代器失效，你可以采取以下措施：

1. 在使用迭代器之前，检查是否执行了可能导致迭代器失效的操作。

2. 尽量使用索引而不是迭代器进行元素的插入和删除操作，因为插入和删除时索引的调整比迭代器更容易控制。

3. 在使用迭代器时，尽量避免在插入或删除元素后继续使用之前的迭代器。

4. 在需要重新分配内存的情况下，尽量一次性分配足够的空间，避免多次不必要的重新分配。




其他的一些函数

```cpp
		vector<T>& operator=(vector<T> v)
		{
			std::swap(_start, v._start);
			std::swap(_finish, v._finish);
			std::swap(_endofstorage, v._endofstorage);

			return *this;
		}


	    void reserve(size_t n)
		{
			if (n > capacity())
			{
				size_t sz = size();
				T* tmp = new T[n];
				if (_start)
				{
					for (size_t i = 0; i < sz; i++)
					{
						tmp[i] = _start[i];
					}

					delete[] _start;
				}

				_start = tmp;
				_finish = _start + sz;
				_endofstorage = _start + n;
			}
		}

		void resize(size_t n, const T& val = T())
		{
			if (n < size())
			{
				_finish = _start + n;
			}
			else
			{
				reserve(n);

				while (_finish != _start + n)
				{
					*_finish = val;
					++_finish;
				}
			}
		}

		void push_back(const T& x)
		{
			insert(end(), x);
		}

		void pop_back()
		{
			erase(--end());
		}

		size_t capacity() const
		{
			return _endofstorage - _start;
		}

		size_t size() const
		{
			return _finish - _start;
		}

		T& operator[](size_t pos)
		{
			assert(pos < size());

			return _start[pos];
		}

		const T& operator[](size_t pos) const
		{
			assert(pos < size());

			return _start[pos];
		}

		

```