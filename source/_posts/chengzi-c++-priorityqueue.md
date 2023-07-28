---
title: 【c++】优先级队列的简单模拟
date: 2023-07-28 15:51:28
tags:
- c++
- queue
categories:
- c/c++
ai: ture
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/c++.png
---

# 概念

>在 C\+\+ 中，std::priority_queue 是一个优先级队列的实现，它提供了按照优先级进行元素存储和访问的功能。与 Python 中的 PriorityQueue 类似，C++ 的 std::priority_queue 使用`堆数据结构`来实现。



创建一个优先级队列：

```cpp
#include <queue>

std::priority_queue<int> pq; // 创建一个存储 int 类型的优先级队列
```

添加元素到队列：

```cpp
pq.push(5);  // 添加 5 到队列
pq.push(3);  // 添加 3 到队列
pq.push(8);  // 添加 8 到队列
```

获取并移除最高优先级的元素：

```cpp
int topElement = pq.top();  // 获取最高优先级的元素（不移除）
pq.pop();  // 移除最高优先级的元素
```

检查队列是否为空：

```cpp
bool isEmpty = pq.empty();  // 检查队列是否为空
```

返回队列中元素的个数：

```cpp
int size = pq.size();  // 返回队列中元素的个数
```

>默认情况下，std::priority_queue 的元素是按照`降序`排列的。也就是说，最大的元素拥有最高的优先级。如果想要实现最小值优先的队列，可以传递一个比较函数对象作为模板参数。

# 模拟

模拟实现只需要建堆就好了。
不过在此之前还得解决一下如何实现最小值优先。

>在这里引入一个概念----仿函数。

函数对象（Function Object）：也称为仿函数（Functor），是一个`类或结构体`，重载了函数调用运算符operator()。通过定义函数调用运算符，可以使用对象的实例作为函数来进行比较操作。
```cpp
struct Compare {
    bool operator()(int a, int b) const {
        return a < b; // 比较 a 是否小于 b
    }
};

// 使用函数对象进行比较
Compare cmp;
bool result = cmp(5, 10);
```

上述代码解决了int型的比较，但queue里可以塞入各种类型，所以还得借助模版参数来进行优化。

```cpp
template<class T>
class Less
{
public:
	bool operator()(const T& x, const T& y)
	{
		return x < y;
	}
};

template<class T>
class Greater
{
public:
	bool operator()(const T& x, const T& y)
	{
		return x > y;
	}
};
```

这样就完成了最大值排列和最小值排列。

然后建堆就行了。因为之前对建堆这块已经写过很详细的文章，在这里就不多啰嗦了。

```cpp

template<class T, class Container = vector<T>, class Comapre = less<T>>
	class priority_queue
	{
	private:
		void AdjustDown(int parent)
		{
			Comapre com;

			// 找左右孩子大的那一个
			size_t child = parent * 2 + 1;
			while (child < _con.size())
			{
				if (child + 1 < _con.size() && com(_con[child], _con[child + 1]))
				{
					++child;
				}

				if (com(_con[parent], _con[child]))
				{
					swap(_con[child], _con[parent]);
					parent = child;
					child = parent * 2 + 1;
				}
				else
				{
					break;
				}
			}
		}

		void AdjustUp(int child)
		{
			Comapre com;

			int parent = (child - 1) / 2;
			while (child > 0)
			{
				if (com(_con[parent], _con[child]))
				{
					swap(_con[child], _con[parent]);
					child = parent;
					parent = (child - 1) / 2;
				}
				else
				{
					break;
				}
			}
		}


	public:
		priority_queue()
		{}

		template<class InputIterator>
		priority_queue(InputIterator first, InputIterator last)
		{
			while (first != last)
			{
				_con.push_back(*first);
				++first;
			}

			// 建堆
			for (int i = (_con.size() - 1 - 1) / 2; i >= 0; i--)
			{
				AdjustDown(i);
			}
		}

		void pop()
		{
			swap(_con[0], _con[_con.size() - 1]);
			_con.pop_back();

			AdjustDown(0);
		}

		void push(const T& x)
		{
			_con.push_back(x);

			AdjustUp(_con.size() - 1);
		}

		const T& top()
		{
			return _con[0];
		}

		bool empty()
		{
			return _con.empty();
		}

		size_t size()
		{
			return _con.size();
		}
	private:
		Container _con;
	};

```

