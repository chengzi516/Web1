---
title: 【c++】list的简单模拟
date: 2023-07-25 16:42:17
tags:
- c++
- list
- 数据结构
categories:
- 数据结构与算法
ai: ture
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.jpg
---

# :imp:介绍list
list指的是标准模板库（STL）中的`双向链表`容器。list是一种动态数组的替代品，它可以在运行时`调整大小`，并且支持高效的插入和删除操作。
要使用list需要包含头文件<list>。

```cpp
#include <iostream>
#include <list>

int main() {
    // 创建一个空的list
    std::list<int> myList;

    // 在list的末尾添加元素
    myList.push_back(1);
    myList.push_back(2);
    myList.push_back(3);

    // 在list的开头添加元素
    myList.push_front(4);

    // 使用迭代器遍历list并打印元素
    for (std::list<int>::iterator it = myList.begin(); it != myList.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // 插入元素
    std::list<int>::iterator insertPos = std::find(myList.begin(), myList.end(), 2);
    if (insertPos != myList.end()) {
        myList.insert(insertPos, 9); 

    // 删除元素
    myList.remove(3); // 删除所有值为3的元素

    return 0;
}
```

list是双向链表，它允许高效地在`任意位置`执行插入和删除操作。然而，由于链表的存储方式，随机访问元素（例如myList[i]）的`性能较差`，因为它需要遍历链表。如果需要随机访问，请考虑使用std::vector或std::array等容器。

# :smiling_imp:模拟

## :two_hearts:节点和迭代器

由于list本质上是一个`双向的链表`结构，所以先实现每一个链表的节点和迭代器作为切入口。

节点一定会有两个指针next和prev，分别指向其前一个节点和后一个节点，还有存储的值val，因为list可以存储各种各样的类型，所以还是需要模版来帮忙。
因为对节点本身并没有什么操作，都是放在链表的层次上进行的，所以再补充一个构造函数即可。

```cpp
    template<class T>
	struct list_node
	{
		list_node<T>* _next;
		list_node<T>* _prev;
		T _val;

		list_node(const T& val = T())
			:_next(nullptr)
			, _prev(nullptr)
			, _val(val)
		{}
	};
```

倒是迭代器的实现有些复杂了，之前模拟的vector或者string，模拟迭代器只需要很简单的几步就可以完成。
例如在vector中的实现：

```cpp
	public:
	    typedef T* iterator;   
	public:
		

		iterator begin()
		{
			return _start;
		}

		iterator end()
		{
			return _finish;
		}

```

>这样设置以后就可以正常使用范围for等需要迭代器支持的功能。之所以可以如此便利，是因为vector本质上进行的是`顺序存储`，而list确是一个链表，所以begin++或者end--大概率是会指向错误的位置的，因为节点的下一个地址可是`随机`的。节点之间是通过next和prev两个指针进行联系的，而非物理上的简单排序。

所以vector里的迭代器类型是T* ，++或者--都是一个T类型的距离，是没问题的。在list里重写迭代器，实际上也是一个指针，只是稍微复杂些。
```cpp
template<class T>
	struct __list_iterator
	{
		typedef list_node<T> Node;
		typedef __list_iterator<T> self;
		Node* _node; // 这是一个指向链表节点的指针成员变量，用来表示当前迭代器所指向的节点。

		//这是迭代器类的构造函数。它接受一个指向链表节点的指针作为参数，并将其保存在 _node 成员变量中。
        __list_iterator(Node* node)
			:_node(node)
		{}

		T operator*()
		{
			return _node->_val;
		}

		T* operator->()
		{
			return &_node->_val;
		}

		self& operator++()  //前置++
		{
			_node = _node->_next;
			return *this;
		}

		self operator++(int)  //后置++
		{
			self tmp(*this);

			_node = _node->_next;

			return tmp;
		}

		self& operator--()
		{
			_node = _node->_prev;
			return *this;
		}

		self operator--(int)
		{
			self tmp(*this);

			_node = _node->_prev;

			return tmp;
		}

		bool operator!=(const self& it) const
		{
			return _node != it._node;
		}

		bool operator==(const self& it) const
		{
			return _node == it._node;
		}
	};
```

## :revolving_hearts:list类和函数实现

有了前面的铺垫，list的重写就很简单了。

```cpp
template<class T>
	class list
	{
		typedef list_node<T> Node;
    private:
		Node* _head;
		size_t _size;
	public:
		typedef __list_iterator<T> iterator;

		iterator begin()
		{
			
			return iterator(_head->_next);
		}

		iterator end()
		{
			
			return iterator(_head);
		}


		void empty_init()
		{
			_head = new Node;
			_head->_prev = _head;
			_head->_next = _head;

			_size = 0;
		}

		list()
		{
			empty_init();
		}

		list(const list<T>& lt)
		{
			empty_init();

			for (auto& e : lt)
			{
				push_back(e);
			}
		}

		list<T>& operator=(list<T> lt)
		{
			std::swap(_head, lt._head);
			std::swap(_size, lt._size);

			return *this;
		}

		~list()
		{
			clear();

			delete _head;
			_head = nullptr;
		}

		void clear()
		{
			iterator it = begin();
			while (it != end())
			{
				it = erase(it);
			}

			_size = 0;
		}

		void push_back(const T& x)
		{
			insert(end(), x);
		}

		void push_front(const T& x)
		{
			insert(begin(), x);
		}

		void pop_back()
		{
			erase(--end());
		}

		void pop_front()
		{
			erase(begin());
		}

		// pos位置之前插入
		iterator insert(iterator pos, const T& x)
		{
			Node* cur = pos._node;
			Node* prev = cur->_prev;
			Node* newnode = new Node(x);

			prev->_next = newnode;
			newnode->_next = cur;

			cur->_prev = newnode;
			newnode->_prev = prev;

			++_size;

			return newnode;
		}

		iterator erase(iterator pos)
		{
			assert(pos != end());

			Node* cur = pos._node;
			Node* prev = cur->_prev;
			Node* next = cur->_next;

			prev->_next = next;
			next->_prev = prev;

			delete cur;

			--_size;

			return next;
		}

		size_t size()
		{
			return _size;
		}

	
	};


```

## :cupid:const的加入

其实可以发现，上面的代码还没完工，因为还有const的迭代器加入了使用。普通的办法就是重新再写一个类型出来：
```cpp
typedef __list_const_iterator<T> const_iterator;
```

但这样看起来就会很冗余，相当于重写了一遍iterator。所以我参照c++库里的实现，用模版完成了代码的改善。

>引入了ref和ptr来接收引用类型与指针类型。

Ref：表示迭代器返回元素值的引用类型。通过重载解引用操作符 *，可以使用迭代器获取节点的值的引用。这使得用户可以修改节点的值。

Ptr：表示迭代器返回元素值的指针类型。通过重载箭头操作符 ->，可以使用迭代器获取节点值的指针。这使得用户可以通过指针访问节点的成员变量或调用节点的成员函数。



```cpp
// __list_iterator的改动
template<class T, class Ref, class Ptr>
	struct __list_iterator
	{
		typedef list_node<T> Node;
		typedef __list_iterator<T, Ref, Ptr> self;
		Node* _node;
        Ref operator*()
		{
			return _node->_val;
		}

		Ptr operator->()
		{
			return &_node->_val;
		}
 
    }
//class list里的改动
typedef __list_iterator<T, T&, T*> iterator;
typedef __list_iterator<T, const T&, const T*> const_iterator;
```

