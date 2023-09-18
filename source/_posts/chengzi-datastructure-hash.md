---
title: 【数据结构】哈希表的实现
date: 2023-09-17 20:16:54
tags:
- c++
- hash表
- 数据结构
categories:
- 数据结构与算法
ai: ture
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.jpg
---


# 哈希表概念

查找是编程里无法绕开的一环，在顺序结构或者树结构中，由于不存在某种元素与位置的映射关系，所以顺序查找时间复杂度为`O(N)`，平衡树中为树的高度，即`O($log_2 N$)`，搜索的效率取决于搜索过程中`元素的比较次数`。
那么一种较为理想的查找就是可以不经过任何比较，`一次`直接从表中得到要搜索的元素。
如果构造一种存储结构，通过某种函数(hashFunc)使得元素的存储位置与它的关键码之间能够建立`一一映射`的关系，那么在查找时通过该函数可以很快找到该元素。

>哈希表（Hash Table）是一种常见的数据结构，主要用于存储`键值对`。它通过将键映射到一个固定长度的`数组`中来实现快速访问和查找。

哈希表由一个固定长度的数组和一组哈希函数组成。当插入一个键值对时，哈希函数会将键`映射`到数组的一个位置上，然后将该键值对存储在该位置上。当需要查找一个键对应的值时，哈希函数可以快速地将键映射到数组的位置，并返回存储在该位置上的值。这也是哈希表工作的基本原理。

然而，由于哈希函数的映射过程是将一个无限的键空间映射到一个有限的数组空间中，所以不可避免地会出现`哈希冲突`。哈希冲突指的是两个不同的键被映射到了数组的同一个位置上。这会导致存储在该位置上的键值对发生冲突，无法正确地插入和查找。

那么，如何解决哈希冲突呢？


>开放地址法（Open Addressing）：该方法使用数组中的其他位置来解决哈希冲突。当一个键被映射到一个已经被占用的位置上时，我们会根据某种规则（如线性探测、二次探测等）找到下一个可用的位置，并将键值对存储在该位置上。这样，所有的键值对都可以被正确地插入和查找。


例如：
通过哈希函数来获取元素该插入的位置。位置为value%capacity,如果此位置被占了那就往后移动，直到找到空位。那么1，3，4，5，6，7都能找到合适的位置，但当插入45时，根据函数规则应该插入到下标为5的位置，但这个位置已经被5这个元素占了，所以它只能往后移动。这种方法也被叫做`线性探测`。
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E5%93%88%E5%B8%8C%E8%A1%A81.png'>

还可以采取`伪删除`的办法。
因为在`闭散列`中不能随便物理删除哈希表中已有的元素，若直接删除元素，就会影响其他元素的搜索。比如删除元素5，如果直接删除掉，45查找起来可能会受影响。因此线性探测采用标记的伪删除法来删除一个元素。
```cpp
enum State{EMPTY, EXIST, DELETE};
```

本篇博客主要采取开散列的办法，因此以上两种方法仅做了解，有兴趣可以参考他人博客。

# 开散列

## 概念

开散列法又叫链地址法(开链法)，首先对关键码集合用散列函数计算散列地址，具有相同地址的关键码归于同一子集合，每一个子集合称为一个`桶`，各个桶中的元素通过一个`单链表`链接起来，各链表的头结点存储在哈希表中。

>链地址法（Chaining）：该方法使用链表来解决哈希冲突。每个数组位置维护一个链表，当多个键被映射到同一个位置上时，它们会被插入到链表中。这样，每个位置上的键值对都可以被正确地查找到。

<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E5%93%88%E5%B8%8C%E8%A1%A82.png'>


## 实现


### 基本结构
```cpp

template<class K> //DefaultHashFunc是一个模板结构体，用于`给不同类型的键值提供默认的哈希函数`。在模拟哈希表的代码中，根据键值类型的不同，会选择相应的哈希函数来计算键值的哈希值。如果键值类型是基本类型，则使用默认的哈希函数，将键值直接转换为size_t类型作为哈希值；如果键值类型是字符串类型，则使用BKDR哈希算法来计算字符串的哈希值。
struct DefaultHashFunc
{
	size_t operator()(const K& key)
	{
		return (size_t)key;
	}
};

template<>
struct DefaultHashFunc<string>
{
	size_t operator()(const string& str)
	{
		// BKDR
		size_t hash = 0;
		for (auto ch : str)
		{
			hash *= 131;
			hash += ch;
		}

		return hash;
	}
};

//采用哈希桶结构来实现哈希表
namespace hash_bucket
{
	template<class T>
	struct HashNode
	{
		T _data;   //每个节点都有一个元素以及指向下一个元素的指针
		HashNode<T>* _next;

		HashNode(const T& data)
			:_data(data)
			,_next(nullptr)
		{}
	};

	//因为迭代器和哈希表的实现都会互相利用，所以使用前置声明
	template<class K, class T, class KeyOfT, class HashFunc>
	class HashTable;

	template<class K, class T, class KeyOfT, class HashFunc>
	struct HTIterator
	{
		typedef HashNode<T> Node;
		typedef HTIterator<K, T, KeyOfT, HashFunc> Self;

		Node* _node;
		HashTable<K, T, KeyOfT, HashFunc>* _pht;

		HTIterator(Node* node, HashTable<K, T, KeyOfT, HashFunc>* pht)
			:_node(node)
			,_pht(pht)
		{}

		T& operator*()
		{
			return _node->_data;
		}

		T* operator->()
		{
			return &_node->_data;
		}

		Self& operator++()
		{
			if (_node->_next)
			{
				// 当前桶还没完
				_node = _node->_next;
			}
			else
			{
				KeyOfT kot;
				HashFunc hf;
				size_t hashi = hf(kot(_node->_data)) % _pht->_table.size();
				// 从下一个位置查找查找下一个不为空的桶
				++hashi;
				while (hashi < _pht->_table.size())
				{
					if (_pht->_table[hashi])
					{
						_node = _pht->_table[hashi];
						return *this;
					}
					else
					{
						++hashi;
					}
				}

				_node = nullptr;
			}

			return *this;
		}

		bool operator!=(const Self& s)
		{
			return _node != s._node;
		}
	};

	//本篇博客实现hashtable是为了之后的unordered系列容器做准备，也就是map和set
	// set -> hash_bucket::HashTable<K, K> _ht;
	// map -> hash_bucket::HashTable<K, pair<K, V>> _ht;
	template<class K, class T, class KeyOfT, class HashFunc = DefaultHashFunc<K>>
	class HashTable   //模版中的keyoft是为了之后在map和set进行复用，在后面会详细讲这个地方
	{
		typedef HashNode<T> Node;

		// 友元声明
		template<class K, class T, class KeyOfT, class HashFunc>
		friend struct HTIterator;
	public:
		typedef HTIterator<K, T, KeyOfT, HashFunc> iterator;

		iterator begin()
		{
			// 找第一个桶
			for (size_t i = 0; i < _table.size(); i++)
			{
				Node* cur = _table[i];
				if (cur)
				{
					return iterator(cur, this);
				}
			}

			return iterator(nullptr, this);
		}

		iterator end()
		{
			return iterator(nullptr, this);
		}
        //构造和析构函数
		HashTable()
		{
			_table.resize(10, nullptr);
		}

		~HashTable()
		{
			for (size_t i = 0; i < _table.size(); i++)
			{
				Node* cur = _table[i];
				while (cur)
				{
					Node* next = cur->_next;
					delete cur;
					cur = next;
				}

				_table[i] = nullptr;
			}
		}
       //插入操作
		bool Insert(const T& data)
		{
			KeyOfT kot;  //现在暂时可以理解为因为map中的pair存在，在和set进行复用此结构时无法兼容二者，通过kot来取出map中的pair的k。

			if(Find(kot(data))) //找到了就不用插入了
			{
				return false;
			}

			HashFunc hf;

			// 负载因子到1就扩容，因为一旦一个位置插入元素过多，就形成了很长的链表，对效率有影响
			if (_n == _table.size())
			{
				
				size_t newSize = _table.size()*2;
				vector<Node*> newTable;
				newTable.resize(newSize, nullptr);

				// 遍历旧表，把节点牵下来挂到新表
				for (size_t i = 0; i < _table.size(); i++)
				{
					Node* cur = _table[i];
					while (cur)
					{
						Node* next = cur->_next;

						// 头插到新表
						size_t hashi = hf(kot(cur->_data)) % newSize;
						cur->_next = newTable[hashi];
						newTable[hashi] = cur;

						cur = next;
					}

					_table[i] = nullptr;
				}

				_table.swap(newTable);
			}

			size_t hashi = hf(kot(data)) % _table.size();
			// 头插
			Node* newnode = new Node(data);
			newnode->_next = _table[hashi];
			_table[hashi] = newnode;
			++_n;
			return true;
		}

		Node* Find(const K& key)
		{
			HashFunc hf;
			KeyOfT kot;
			size_t hashi = hf(key) % _table.size();
			Node* cur = _table[hashi];
			while (cur)
			{
				if (kot(cur->_data) == key)
				{
					return cur;
				}

				cur = cur->_next;
			}

			return nullptr;
		}

		bool Erase(const K& key)
		{
			HashFunc hf;
			KeyOfT kot;
			size_t hashi = hf(key) % _table.size();
			Node* prev = nullptr;
			Node* cur = _table[hashi];
			while (cur)
			{
				if (kot(cur->_data) == key)
				{
					if (prev == nullptr)
					{
						_table[hashi] = cur->_next;
					}
					else
					{
						prev->_next = cur->_next;
					}

					delete cur;	
					return true;
				}

				prev = cur;
				cur = cur->_next;
			}

			return false;
		}

		void Print()
		{
			for (size_t i = 0; i < _table.size(); i++)
			{
				printf("[%d]->", i);
				Node* cur = _table[i];
				while (cur)
				{
					cout << cur->_kv.first <<":"<< cur->_kv.second<< "->";
					cur = cur->_next;
				}
				printf("NULL\n");
			}
			cout << endl;
		}

	private:
		vector<Node*> _table; // 指针数组
		size_t _n = 0; // 存储了多少个有效数据
	};
}
```


# q
