---
title: 【c++】string库的常用函数的模拟实现
date: 2023-07-17 13:37:44
tags:
- c语言
- string
categories:
- c/c++
ai: ture
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/c++.png
---

>最近学到了string的相关的库，于是为了加深对函数的理解便打算模拟实现一下常用的string库函数，也顺便补习了一些之前不太理解的语法或概念。


# :sunny:引入

这是一个简单的string类的用法。

```cpp
#include <string>
#include <iostream>

int main() 
{
    std::string str = "Hello, world!";
    std::cout << "The length of the string is: " << str.size() << std::endl;
    return 0;
}

```

由上可见，string类是C++标准库中的一个类，它定义在`std命名空间`中。因此，当使用string类时，需要指定其命名空间为std，以便编译器可以找到正确的定义。如果没有指定命名空间，编译器可能会认为string是一个未定义的标识符，导致编译错误。

>std是C++标准库中定义的命名空间，它包含了许多C++标准库中的类、函数和类型定义。在C++中，命名空间提供了一种将名称隔离开来以`避免名称冲突`的机制。这样，不同库中的同名函数或类就可以在同一个程序中共存，而不会发生冲突。

所以我想进行模拟的string类实现，为了避免与std的string类冲突，就自定义了一个新的namespace——newstring。

```cpp
namespace newstring {
	class string {
	private:
		size_t _size;
		size_t _capacity;
		char* _str;
    }
}
```

在newstring这块命名空间中，我创建了自己的string类，有三个基本元素：
1. 字符串_str
2. 容量_capacity
3. 目前大小_size

设置容量和大小的目的主要是为了方便内存的管理，同时需要注意，字符串虽然是以'\0'结尾，但在string类中，是否结束的标志是size。

```cpp
 std::string str1 = "123\\01212121";
    size_t len = str1.size(); // 获取字符串的长度，不包括 null 结尾符
    for (auto c : str1) { // 遍历字符串，包括 null 字符和换行符
        std::cout << c;
    }
```
如上，输出的结果是123\01212121，并没有遇到\0就终止。

# :umbrella:函数模拟实现

## :cloud:构造函数与析构函数
首先要实现的肯定是构造和析构，其重要性不必我多说。

```cpp
		string(const char* str = "") {
			_size = strlen(str);
			_capacity = _size;
			_str = new char[_size + 1];
			strcpy(_str, str);

		}
		~string()
		{
			delete[] _str;
			_str = nullptr;
			_size = _capacity = 0;
		}
```

>这里还有一点要注意，虽然常见的输入都是以'\0'为结尾的字符串，但有时候仍然会遇见
中间出现'\0'的串，这时使用这里的构造函数就会出现问题了。

比如，一个串为"hello\0world",在进行strcpy时就只会copy到`第一个遇到的\0`为止，后面的内容就会遗失，所以为了避免这种极少见的情况，可以考虑将strcpy换成`memcpy`。

不清楚的可以看下这里的对比分析。

strcpy 和 memcpy 都是 C/C++ 中常用的`字符串/内存拷贝`函数。
strcpy 函数用于将一个以 `null` 结尾的字符串从源地址拷贝到目标地址，其函数原型如下：

```cpp
char* strcpy(char* dest, const char* src);
```

其中，dest 是`目标地址`，src 是`源地址`。strcpy 函数会将源地址（包括 null 结尾符）中的所有字符拷贝到目标地址中，并`返回目标地址的指针`。strcpy 函数`不会检查目标地址是否有足够的空间`来存储源地址中的所有字符，如果目标地址不够大，可能会导致内存溢出和程序崩溃的问题。

例如，下面的代码使用 strcpy 函数将字符串 "Hello, world!" 拷贝到字符数组 str 中：

```cpp
char str[20];
strcpy(str, "Hello, world!");
```

memcpy 函数用于将`指定长度的内存块`从源地址拷贝到目标地址，其函数原型如下：

```cpp
void* memcpy(void* dest, const void* src, size_t count);
```


其中，dest 是目标地址，src 是源地址，count 是要拷贝的字节数。memcpy 函数会将源地址中的指定长度的内存块拷贝到目标地址中，并返回目标地址的指针。与 strcpy 不同，memcpy 函数`不会自动添加 null 结尾符`，而且要求目标地址有足够的空间来存储拷贝的数据。

例如，下面的代码使用 memcpy 函数将长度为 10 的内存块从数组 src 中拷贝到数组 dest 中：

```cpp
int src[10] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
int dest[10];
memcpy(dest, src, sizeof(src));
```

下面这一点非常重要！

>由于 memcpy 函数`不会自动添加 null 结尾符`，如果使用 memcpy 函数拷贝字符串时，需要手动添加 null 结尾符，例如：

```cpp
char str[20];
char* src = "Hello, world!";
memcpy(str, src, strlen(src) + 1);
```

strcpy 和 memcpy 在使用时要根据具体的需求选择，需要注意它们的区别和各自的用法。


## :snowflake:获取其长度，容量，内容
由于是自定义的类，无法直接使用cout来输出，那么就可以考虑实现c_str来获取字符串内容。长度和容量由于在其他函数中复用很多，也就一并拿出来了。

```cpp
	    // 获取字符串长度
		size_t size() const {
			return _size;
		}

		// 获取字符串容量
		size_t capacity() const {
			return _capacity;
		}

		// 获取字符串内容
		const char* c_str() const {
			return _str;
		}

```

## :snowman:reserve函数
reserve 函数用于为`字符串预分配内存空间`，以提高字符串操作的效率和性能。其函数原型如下：

```cpp
void reserve(size_type new_cap);
```

其中，new_cap 是要预分配的内存空间大小，以字节为单位。reserve 函数会尝试为字符串分配至少 new_cap 个字节的内存空间，如果当前已经分配的内存空间足够大，则不会进行分配，如果分配失败，则会抛出 `std::bad_alloc` 异常。

下面的代码使用 reserve 函数为字符串 str 预分配 100 个字节的内存空间：

```cpp
std::string str;
str.reserve(100);
```
调用reserve函数仅仅是为字符串预分配内存空间，并`不会改变字符串的长度`。如果要修改字符串的长度，可以使用 resize 函数或者直接对字符串进行赋值操作。
另外，由于 C++ STL 中的 std::string 类已经封装了内存管理的细节，因此在大多数情况下不需要手动调用 reserve 函数进行内存管理，只需要使用字符串类提供的成员函数和操作符即可。只有在特殊的性能优化或者内存限制的情况下，才需要手动调用 reserve 函数。

模拟实现如下：
```cpp
	void reserve(size_t n)
		{
			if (n > _capacity) //只有当预留空间大于当前分配空间才会进行
			{
                 //核心思路就是开辟一个新的空间，让旧指针指向他
				char* tmp = new char[n + 1];
				strcpy(tmp, _str);
				delete[] _str;
				_str = tmp;
				_capacity = n;
			}
		}
```


## :zap:push_back和append函数

push_back 和 append用于向字符串`末尾`添加新的字符或字符串。它们的区别和用法如下：

### :cyclone:push_back
push_back 函数用于向字符串末尾添加`一个字符`。其函数原型如下：

```cpp
void push_back(char ch);
```

其中，ch 是要添加的字符。例如，下面的代码使用 push_back 函数向字符串 str 中添加字符 'a'：

```cpp
std::string str = "Hello, world!";
str.push_back('a');
```

上述代码中，push_back 函数将字符 'a' 添加到字符串 str 的末尾。
模拟实现为
```cpp
void push_back(char ch)
		{
			if (_size == _capacity)
			{
				// 2倍扩容
				reserve(_capacity == 0 ? 4 : _capacity * 2);
			}

			_str[_size] = ch;

			++_size;
			_str[_size] = '\0'; //不要忘记\0
		}
```

### :foggy:append
append 函数用于向字符串末尾添加`一个字符串`。其函数原型如下：

```cpp
basic_string& append(const basic_string& str);
basic_string& append(const char* s);
basic_string& append(const char* s, size_t n);
basic_string& append(size_t n, char c);
```

其中，str 是要添加的字符串，s 是要添加的字符数组，n 是要添加的字符个数，c 是要添加的字符。例如，下面的代码使用 append 函数向字符串 str 中添加一个字符串和一个字符：

```cpp
std::string str = "Hello, world!";
str.append(" - C++", 5).append(1, '!');
```

上述代码中，append 函数先将字符串 " - C++" 中的前 5 个字符（即 " - C"）添加到字符串 str 的末尾，然后再将字符 '!' 添加到字符串 str 的末尾。
append 函数可以一次性向字符串中添加多个字符和字符串，比 push_back 函数更加灵活。

模拟实现为

```cpp
void append(const char* str)
		{
			size_t len = strlen(str);
			if (_size + len > _capacity)
			{
				// 至少扩容到_size + len
				reserve(_size + len);
			}

			strcpy(_str + _size, str);
			_size += len;
		}
```

## :ocean:+=操作符
有了append和push_back，+=直接复用就好。

```cpp
	string& operator+=(char ch)
		{
			push_back(ch);
			return *this;
		}

	string& operator+=(const char* str)
		{
			append(str);
			return *this;
		}
```

## :cat:insert函数

insert用于向字符串中插入字符或字符串。其函数原型如下：

```cpp
basic_string& insert(size_type pos, const basic_string& str);
basic_string& insert(size_type pos, const basic_string& str, size_type subpos, size_type sublen);
basic_string& insert(size_type pos, const char* s, size_type n);
basic_string& insert(size_type pos, const char* s);
basic_string& insert(size_type pos, size_type n, char c);
iterator insert(const_iterator p, char c);
void insert(const_iterator p, size_type n, char c);
template< class InputIt >
void insert(const_iterator p, InputIt first, InputIt last);
```

其中，pos 是要插入的位置，str 是要插入的字符串，subpos 和 sublen 是要插入的子字符串的起始位置和长度，s 是要插入的字符数组，n 是要插入的字符个数，c 是要插入的字符，p 是要插入的位置，first 和 last 是要插入的字符序列的起始和结束迭代器。

在指定位置插入一个字符或一个字符序列。
例如，下面的代码使用 insert 函数在字符串 str 的第 5 个位置插入字符 'a'：

```cpp
std::string str = "Hello, world!";
str.insert(5, 1, 'a');
```

上述代码中，insert 函数将字符 'a' 插入到字符串 str 的第 5 个位置。

insert 函数还可以插入一个字符序列，例如：

```cpp
std::string str = "Hello, world!";
str.insert(5, " - C++");
```

上述代码中，insert 函数将字符串 " - C++" 插入到字符串 str 的第 5 个位置。

常用的也就是上面两种。
模拟实现如下：

```cpp
void insert(size_t pos, size_t n, char ch)
		{
			assert(pos <= _size);

			if (_size + n > _capacity)
			{
				// 至少扩容到_size + len
				reserve(_size + n);
			}
			size_t end = _size;
			while (end >= pos && end<_size)//重要的地方
			{
				_str[end + n] = _str[end];
				--end;
			}

			for (size_t i = 0; i < n; i++)
			{
				_str[pos + i] = ch;
			}

			_size += n;
		}

		void insert(size_t pos, const char* str)
		{
			assert(pos <= _size);

			size_t len = strlen(str);
			if (_size + len > _capacity)
			{
				// 至少扩容到_size + len
				reserve(_size + len);
			}
			size_t end = _size;
			while (end >= pos && end != -1)
			{
				_str[end + len] = _str[end];
				--end;
			}

			for (size_t i = 0; i < len; i++)
			{
				_str[pos + i] = str[i];
			}

			_size += len;
		}
```

insert的模拟实现就是先判定空间是否够大，不够大就扩容，然后把插入位置pos后面的元素都往后挪n个位置，再回到pos插入指定的元素。
>while (end >= pos && end<_size)

这个地方单独提一下，因为end存在减为0继续减到-1的情况，但这里end是size_t类型，是一个`无符号整型`，当0再减1时，end就变成了一个`极大数`，此时需要end<_size来终止循环。

## :dog:find函数
find用于在字符串中查找指定子串的位置。其函数原型如下：

```cpp
size_t find(const basic_string& str, size_t pos = 0) const noexcept;
size_t find(const char* s, size_t pos, size_t n) const;
size_t find(const char* s, size_t pos = 0) const;
size_t find(char c, size_t pos = 0) const noexcept;
```
其中，str 是要查找的子串，s 是要查找的字符数组，n 是要查找的字符个数，c 是要查找的字符，pos 是查找起始位置。具体用法如下：

查找指定字符串在源字符串中的位置。
例如，下面的代码使用 find 函数在字符串 str 中查找子串 "world" 的位置：

```cpp
std::string str = "Hello, world!";
size_t pos = str.find("world");
```

上述代码中，find 函数查找字符串 "world" 在字符串 str 中第一次出现的位置，并将其返回。如果字符串 "world" 不在字符串 str 中，find 函数将返回 std::string::npos。

查找指定字符数组在源字符串中的位置。
例如，下面的代码使用 find 函数在字符串 str 中查找字符数组 "world" 的位置：

```cpp
std::string str = "Hello, world!";
size_t pos = str.find("world", 7, 5);
```

上述代码中，find 函数在字符串 str 的第 7 个位置开始查找字符数组 "world" 的前 5 个字符，并将其返回。如果字符数组 "world" 不在字符串 str 中，find 函数将返回 std::string::npos。

查找指定字符在源字符串中的位置。
例如，下面的代码使用 find 函数在字符串 str 中查找字符 'o' 的位置：

```cpp
std::string str = "Hello, world!";
size_t pos = str.find('o', 5);
```

上述代码中，find 函数在字符串 str 的第 5 个位置开始查找字符 'o' 的位置，并将其返回。如果字符 'o' 不在字符串 str 中，find 函数将返回 std::string::npos。

>在string类中存在`npos`这个`特殊的常量`。在字符串操作中，通常使用 find 等函数来查找指定子串或字符在原字符串中的位置。如果查找失败，这些函数会`返回一个特殊的值 npos`，以表示查找失败的情况。

以下是两种常用形式的模拟实现

```cpp
size_t find(char ch, size_t pos = 0)
		{
			assert(pos < _size);

			for (size_t i = pos; i < _size; i++)
			{
				if (_str[i] == ch)
				{
					return i;
				}
			}

			return -1;  //也可以是npos，不过需要自己新定义一个，npos 是 std::string::npos 的一个静态成员变量。
		}

		size_t find(const char* str, size_t pos = 0)
		{
			assert(pos < _size);

			const char* ptr = strstr(_str + pos, str); //strstr 是 C 语言标准库中的一个函数，用于在一个字符串中查找另一个字符串的第一次出现位置。
			if (ptr)
			{
				return ptr - _str;
			}
			else
			{
				return -1;
			}
		}


```

## :mouse:substr函数

substr用于从一个字符串中提取子串。其函数原型为：

```cpp
std::string substr(size_t pos = 0, size_t count = npos) const;
```

其中，pos 是子串的起始位置，count 是子串的长度。如果省略 count 参数，则返回从 pos 开始到字符串末尾的所有字符。如果 pos 大于等于字符串的长度，或者 count 为 0，那么函数返回一个空字符串。

下面是一个使用 substr 函数提取子串的示例：

```cpp
#include <iostream>
#include <string>

int main() {
    std::string str = "hello, world";
    std::string sub = str.substr(7, 5);  // 从位置 7 开始提取长度为 5 的子串
    std::cout << sub << std::endl;  // 输出 "world"
    return 0;
}
```

上述代码中，substr 函数从字符串 str 中提取了一个子串，其起始位置是 7，长度是 5。提取出的子串是 "world"，并赋值给了变量 sub。
模拟实现如下

```cpp
string substr(size_t pos = 0, size_t len = -1)
		{
			assert(pos < _size);

			size_t n = len;
			if (len == -1 || pos + len > _size)
			{
				n = _size - pos;
			}

			string tmp;
			tmp.reserve(n);
			for (size_t i = pos; i < pos + n; i++)
			{
				tmp += _str[i];
			}

			return tmp;
		};
//测试代码
newstring::string s1("hello world");
newstring::string ss = s1.substr(2, 3);
std::cout << ss.c_str();		
```

>在这里我进行测试时发现程序崩溃，经过调试发现是忘记了写拷贝构造函数。因为我下意识地去重载了=操作符，而忽略了此时ss = s1.substr(2, 3)里的=`调用的并非是=，而是拷贝构造函数`。

$直接初始化$

>如果创建新对象，一定有构造函数被调用。

使用直接初始化时，可能使用构造函数，也可能使用拷贝构造函数。用（）来定义对象的就为直接初始化。

$拷贝初始化$

>将一个对象给另一个对象初始化。用=定义对象的就为拷贝初始化。


拷贝赋值运算符也用在=的情况下，但是在对象`已经创建并存在`的情况下，只修改对象的值。
而用=拷贝初始化是发生在定义一个对象的情况下，即对象此前尚未存在。这也就是我的代码这里的情况，ss是一个先前不存在的对象，所以在这里调用的是拷贝构造而非=操作符。


## :hamster:迭代器

因为是自己实现的类，所以要用到范围for这样的代码，就得自己实现迭代器。注意有无const的情况最好都写一下。

```cpp
public:
		typedef char* iterator;
		typedef const char* const_iterator;

		iterator begin()
		{
			return _str;
		}

		iterator end()
		{
			return _str + _size;
		}

		const_iterator begin() const
		{
			return _str;
		}

		const_iterator end() const
		{
			return _str + _size;
		}
```

这样就可以使用范围for等需要迭代器的用法了。

```cpp
 for (auto e : s1) {
        std::cout << e << std::endl;
    }
```

如果你的类支持正向遍历和反向遍历，你可能需要实现两个迭代器：一个`正向迭代器和一个反向迭代器`。

一旦你实现了迭代器，你的自定义类就可以像标准容器一样使用范围for语句了。在范围for语句中，你需要使用 `begin() `方法返回你的类的起始迭代器，使用`end()` 方法返回你的类的结束迭代器。例如：

```cpp
class MyContainer {
public:
    // ...
    Iterator begin() { /* 返回正向迭代器 */ }
    Iterator end() { /* 返回正向迭代器 */ }
    ReverseIterator rbegin() { /* 返回反向迭代器 */ }
    ReverseIterator rend() { /* 返回反向迭代器 */ }
};

MyContainer container;
for (auto& element : container) {
    // 处理元素
}
for (auto& element : std::reverse(container)) {
    // 反向遍历元素
}
```

## :rabbit:resize函数

resize() 是 C++ 标准库中容器和字符串类中的一个成员函数，用于改变容器或字符串的大小。
resize() 会`改变容器中元素的个数`。
对容器而言：
1. 如果新的大小`小于`当前大小，那么超出新大小的元素将被截断；
2. 如果新的大小`大于`当前大小，那么容器将增加新元素，并使用默认构造函数初始化这些新元素。

对于字符串：

1. 如果新的大小`小于`当前大小，那么超出新大小的字符将被截断；
2. 如果新的大小`大于`当前大小，那么字符串将增加新字符，并使用` '\0' 字符`填充这些新字符。

```cpp
void resize(size_t n, char ch = '\0')
		{
			if (n < _size)
			{
				_size = n;
				_str[_size] = '\0';
			}
			else
			{
				reserve(n);

				for (size_t i = _size; i < n; i++)
				{
					_str[i] = ch;
				}

				_size = n;
				_str[_size] = '\0';
			}
		}
```

## :wolf:一些运算符

```cpp
        bool operator<(const string& s) const
		{
			int ret = memcmp(_str, s._str, _size < s._size ? _size : s._size);
			//_str比s._str大则返回正数，小则返回负数，相等则返回0
			return ret == 0 ? _size < s._size : ret < 0;
		}
```
第一个 const 修饰的是`函数参数 s`，表示该参数是一个常量引用，避免了对参数进行修改。而第二个 const 修饰的是`函数自身`，表示该成员函数不会修改对象的状态，即不会修改该对象的成员变量。
用一个写好的运算符在其他的运算符中进行复用也是一个常见的技巧。

```cpp
		bool operator==(const string& s) const
		{
			return _size == s._size 
			&& memcmp(_str, s._str, _size) == 0;
		}

		bool operator<=(const string& s) const
		{
			return *this < s || *this == s;  //像这个，直接就复用了才写好的<和==		
		}

		bool operator>(const string& s) const
		{
			return !(*this <= s);
		}

		bool operator>=(const string& s) const
		{
			return !(*this < s);
		}

		bool operator!=(const string& s) const
		{
			return !(*this == s);
		}

```

## :frog:<<与>>

之前进行输出采用的是借助c_str来实现的，要实现

```cpp
cout<<s<<endl;
```

这样的效果，就需要重载>>与<<操作符。

```cpp
    ostream& operator<<(ostream& out, const string& s)
	{
		for (auto ch : s)
		{
			out << ch;
		}

		return out;
	}

	istream& operator>>(istream& in, string& s)
	{
		s.clear();

		char ch = in.get(); //从输入流中读取一个字符
		// 处理前缓冲区前面的空格或者换行
		while (ch == ' ' || ch == '\n')
		{
			ch = in.get();
		}

		char buff[128];
		int i = 0;

		while (ch != ' ' && ch != '\n')
		{
			buff[i++] = ch;
			if (i == 127)
			{
				buff[i] = '\0';
				s += buff;
				i = 0;
			}

			ch = in.get();
		}

		if (i != 0)
		{
			buff[i] = '\0';
			s += buff;
		}

		return in;
	}

```

>这两个重载需要放在全局情况中，不要放到string类中！

这是因为在 C\+\+ 中，运算符重载是一种基于函数重载的机制，它允许自定义类对象的行为，并支持与内置类型的相同语义。在大多数情况下可以重载任何运算符，包括 <, >, ==, << 和 >> 等。
然而，对于 << 和 >> 运算符，它们具有特殊的语义，通常用于`输入和输出`操作，因此在重载时需要特别注意。如果将 << 或 >> 运算符`作为成员函数`来定义，其左操作数将被隐式地设置为`该类的对象`，而右操作数则需要作为`函数参数`传入。但是，对于输入流对象和输出流对象，它们是作为`全局对象`存在的，而不是作为类的成员对象存在的，因此需要使用全局函数来重载这些运算符。