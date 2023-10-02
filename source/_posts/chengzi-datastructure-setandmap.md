---
title: 【数据结构】map与set，基于红黑树的简单模拟实现
date: 2023-09-13 15:42:22
tags:
- c++
- map
- set
- 红黑树
- 数据结构
categories:
- 数据结构与算法
ai: ture
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.jpg
---

>阅读本文前最好了解AVL树的相关简单实现。

# map与set

map是一种关联数组（Associative Array），也被称为字典（Dictionary）或键值对（Key-Value）容器。它将`键和值`一一对应存储，通过键快速查找对应的值。在map中，键是`唯一`的，且按照一定的排序规则进行存储。

简单的代码示例(使用map存储学生姓名与分数)：
```cpp
#include <iostream>
#include <map>

int main() {
    std::map<std::string, int> studentScores;

    // 添加学生姓名和分数
    studentScores["Alice"] = 95;
    studentScores["Bob"] = 80;
    studentScores["Charlie"] = 90;

    // 遍历map，输出学生姓名和分数
    for (const auto& pair : studentScores) {
        std::cout << pair.first << ": " << pair.second << std::endl;
    }

    return 0;
}

```

set是一种`有序`集合容器，它存储的元素按照一定的排序规则进行存储，并且元素是`唯一`的，不允许重复。set提供了快速查找元素的功能，并且可以进行插入和删除等操作。


使用set储存有序数组：
```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> numbers;

    // 添加整数到set
    numbers.insert(5);
    numbers.insert(3);
    numbers.insert(8);
    numbers.insert(1);
    numbers.insert(5);  // 重复的元素将不会被添加

    // 遍历set，输出整数
    for (const auto& num : numbers) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    // 查找元素
    int target = 3;
    auto iter = numbers.find(target);
    if (iter != numbers.end()) {
        std::cout << target << " found in the set." << std::endl;
    } else {
        std::cout << target << " not found in the set." << std::endl;
    }

    return 0;
}

```


查看库函数，会发现在map和set的实现中，都用到了`红黑树`。

红黑树通过`自平衡`的机制，可以保持树的高度相对较小，从而提供了较好的平衡性能。这意味着在插入、删除和搜索等操作中，红黑树能够保持相对较稳定的性能，避免出现极端情况导致操作的时间复杂度变大。
由于红黑树是一种二叉搜索树，它具有`快速搜索`的特性。根据红黑树的特点，可以通过比较节点的值来确定搜索方向，从而快速找到目标节点。在map和set等容器中，通过红黑树实现的关联数组和有序集合，可以在较快的时间内完成搜索操作。
红黑树具有较好的动态性能，即在插入和删除节点时能够保持树的平衡。通过旋转和变色等操作，红黑树能够在O(log n)的时间复杂度内完成节点的插入和删除操作，保持树的平衡性，避免出现极端情况导致树的高度过大。
除此之外，红黑树还具备实现简单，占用空间小等优点，就不一一列举了。


# 红黑树的简单实现


## 概念
想要模拟实现set与map，就得先实现红黑树。

>红黑树是一种自平衡的二叉搜索树，它在插入和删除节点时会通过一系列的旋转和变色操作来保持树的平衡。

红黑树会遵循以下的原则：
1. 每个节点都有一个颜色属性，只能是`红色或黑色`。
2. 根节点是`黑色`的。
3. `所有叶子节点`都是黑色的。
4. 如果一个节点是`红色`的，则它的两个子节点都是`黑色`的。
5. 任意节点到其每个叶子节点的路径上，黑色节点的数量是`相同`的。

在这五条规则的制约下，红黑树就可以保证最重要的一条特性：`最长路径不会超过最短路径的两倍`。

最短路径就是`全黑`节点，最长路径就是`一个红节点一个黑节点`（因为红色规定不能连续且每个路径上黑色节点数量一致），最后黑色节点相同时，最长路径刚好是最短路径的两倍。


红黑树与上一篇写的AVL树其实大差不差，但AVL树却很少运用到实际的操作中去，因为相比于AVL树，红黑树是更简单的。
红黑树不像AVL树要求节点的高度差`必须小于等于1`，只要部分达到平衡即可。所以在实现上，可以认为红黑树比AVL数更简单。
AVL树肯定是更平衡，结构上也更加直观，但维护慢，空间开销也较大。
总的来说：

>AVL树的时间复杂度虽然优于红黑树，但现在基于高效的计算机算力，基本可以忽略二者的性能差异。
红黑树的插入删除等操作比AVL树更容易。
红黑树因为不要求严格平衡所以`旋转情况少于`AVL树。


## 插入操作

红黑树的插入大致可以归纳为以下两点：
1. 当根节点为NULL时，直接插入新节点并置为黑色。
2. 根节点不为NULL时，找到要插入新节点的位置。同时判断新插入节点的颜色，并随之更新其他受影响节点的颜色。

寥寥几句，可能看起来很容易，但实现却依旧很复杂。

比如插入新节点的颜色如何确定？黑色的话确实方便，不管哪里都能插，好像不受到限制，但基于刚才提到的第五点规则，插入新的黑节点必然要调解其他路径的黑节点数量，反而很复杂。如果插入红节点就不需要调整路径，所以我们一般设定插入新节点都是`红色`。
当父节点是黑色，那么`直接插入`就好，当父节点是红色，就需要像AVL树一样进行调整来满足规则。

当插入节点的父亲是红色，可以分成两种情况。

>新节点(cur)红，父亲(parent)红，父亲的父亲(pparent)是黑色，叔叔(uncle)节点（父亲节点的兄弟节点）是红色。

<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/rbtree1.jpg'>
parent,uncle置为黑色，pparent改为红，然后把pparent当成cur，继续向上调整。

>cur红，parent红，pparent为黑，uncle不存在或者uncle黑。

parent为pparent的左孩子，cur为parent的左孩子，则进行右单旋转。然后pparent变红，parent变黑。
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/rbtree2.jpg'>
parent为pparent的右孩子，cur为parent的右孩子，则进行左单旋转。然后pparent变红，parent变黑。
更多的情况就不在赘述，直接看代码。

完整代码如下：
```cpp
enum Colour
{
	RED,
	BLACK
};


struct RBTreeNode
{
	RBTreeNode* _left;
	RBTreeNode* _right;
	RBTreeNode* _parent;

	int _key;
	Colour _col;

	RBTreeNode(const int& key)
		:_left(nullptr)
		,_right(nullptr)
		,_parent(nullptr)
		,_key(key)
		,_col(RED)
	{}
};


struct RBTree
{
	typedef RBTreeNode Node;
public:
	bool Insert(const int& key)
	{
		if (_root == nullptr)  //插入的是首节点
		{
			_root = new Node(key);
			_root->_col = BLACK;
			return true;
		}

		Node* parent = nullptr;
		Node* cur = _root;
		while (cur)
		{
			if (cur->_key < key)
			{
				parent = cur;
				cur = cur->_right;
			}
			else if (cur->_key> key)
			{
				parent = cur;
				cur = cur->_left;
			}
			else
			{
				return false;
			}
		}

		cur = new Node(key);
		cur->_col = RED;
		if (parent->_key < key)
		{
			parent->_right = cur;
		}
		else
		{
			parent->_left = cur;
		}

		cur->_parent = parent;
        //以下为核心操作
		while (parent && parent->_col == RED)
		{
			Node* grandfather = parent->_parent;
			if (parent == grandfather->_left)
			{
				Node* uncle = grandfather->_right;
				// uncle存在且为红
				if (uncle && uncle->_col == RED)
				{
					// 变色
					parent->_col = uncle->_col = BLACK;
					grandfather->_col = RED;

					// 继续向上处理
					cur = grandfather;
					parent = cur->_parent;
				}
				else // u不存在 或 存在且为黑
				{
					if (cur == parent->_left)
					{
						RotateR(grandfather);
						parent->_col = BLACK;
						grandfather->_col = RED;
					}
					else
					{
						RotateL(parent);
						RotateR(grandfather);

						cur->_col = BLACK;
						grandfather->_col = RED;
					}

					break;
				}
			}
			else // parent == grandfather->_right
			{
				Node* uncle = grandfather->_left;
				// u存在且为红
				if (uncle && uncle->_col == RED)
				{
					// 变色
					parent->_col = uncle->_col = BLACK;
					grandfather->_col = RED;

					// 继续向上处理
					cur = grandfather;
					parent = cur->_parent;
				}
				else
				{
					if (cur == parent->_right)
					{
						RotateL(grandfather);
						grandfather->_col = RED;
						parent->_col = BLACK;
					}
					else
					{
						RotateR(parent);
						RotateL(grandfather);
						cur->_col = BLACK;
						grandfather->_col = RED;
					}

					break;
				}
			}
		}

		_root->_col = BLACK;

		return true;
	}

	void RotateL(Node* parent)
	{
		++_rotateCount;

		Node* cur = parent->_right;
		Node* curleft = cur->_left;

		parent->_right = curleft;
		if (curleft)
		{
			curleft->_parent = parent;
		}

		cur->_left = parent;

		Node* ppnode = parent->_parent;

		parent->_parent = cur;


		if (parent == _root)
		{
			_root = cur;
			cur->_parent = nullptr;
		}
		else
		{
			if (ppnode->_left == parent)
			{
				ppnode->_left = cur;
			}
			else
			{
				ppnode->_right = cur;

			}

			cur->_parent = ppnode;
		}
	}


	void RotateR(Node* parent)
	{
		++_rotateCount;

		Node* cur = parent->_left;
		Node* curright = cur->_right;

		parent->_left = curright;
		if (curright)
			curright->_parent = parent;

		Node* ppnode = parent->_parent;
		cur->_right = parent;
		parent->_parent = cur;

		if (ppnode == nullptr)
		{
			_root = cur;
			cur->_parent = nullptr;
		}
		else
		{
			if (ppnode->_left == parent)
			{
				ppnode->_left = cur;
			}
			else
			{
				ppnode->_right = cur;
			}

			cur->_parent = ppnode;
		}
	}
	
	
private:
	Node* _root = nullptr;

public:
	int _rotateCount = 0;
};
```

# set与map的模拟实现

## KeyofT设计
有了基本的红黑树结构，我们就能动手尝试实现set与map。
set只需要key，而map需要key和value，如何实现在同一棵红黑树的结构下，同时实现map和set的兼容呢？
比如我们要实现find函数，在寻找的过程中必然会遇到`比较`的问题，此时map如果是pair的数据结构，比起set一个简单的key来说，比较的逻辑就不太好实现，我们也不能因为这就重新写一棵树。
所以就有了KeyofT的设计。
以map为例：
直接在map类里定义一个新的结构，也可以在类外面定义，以指针的形式引用函数。
```cpp
template<class k,class v>
	class map {
		struct MapKeyOfT {
			const k& operator()(const pair<k, v>& kv) {
				return kv.first;
			}
		};
	private:     
		RBTree<k, pair< k,v>,MapKeyOfT> _t; 
		}
```
set为了与map保持一致，可以传入两个key：
```cpp
template<class k>
	class set {
		struct SetKeyOfT {
			const k& operator()(const k& key) {
				return key;
			}
		};
	private:
		RBTree<k, k, SetKeyOfT> _t;
		}
```
完成上面的步骤后，比如find函数就可以修改为这样：
```cpp
Node* Find(const K& key)
	{
		Node* cur = _root;
		KeyOfT kot;
		while (cur)
		{
			if (kot(cur->_data) < key)
			{
				cur = cur->_right;
			}
			else if (kot(cur->_data) > key)
			{
				cur = cur->_left;
			}
			else
			{
				return cur;
			}
		}

		return nullptr;
	}
```

## 迭代器设计
有了基本的红黑树结构，我们就能动手尝试实现set与map。
set只需要一个key，而map需要key和value，且map的value可以被更改，而key同set一样，是不允许被修改的。这些要素都需要被考虑进去。
所以重点在于迭代器的设计上。
参照库里的实现，可以发现：
```cpp
typedef typename RBTree<k, k, SetKeyOfT>::const_iterator iterator;
typedef typename RBTree<k, k, SetKeyOfT>::const_iterator const_iterator;
```
即两种迭代器都被实例化为const型（因为set不允许更改key），虽然在这里很洒脱，但在后面的设计上却造成了不小的问题。

先实现一个新的类__TreeIterator,里面也都是基础的迭代器实现。
```cpp
template<class T,class Ptr,class Ref>
struct __TreeIterator
{
	typedef RBTreeNode<T> Node;
	typedef __TreeIterator<T, Ptr, Ref> Self;
	Node* _node;

	__TreeIterator(Node* node)
		:_node(node)
	{}
	}
	Ref operator*()
	{
		return _node->_data;
	}

	Ptr operator->()
	{
		return &_node->_data;
	}

	bool operator!=(const Self& s)
	{
		return _node != s._node;
	}
	Self& operator--()
	{
		if (_node->_left)
		{
			Node* subRight = _node->_left;
			while (subRight->_right)
			{
				subRight = subRight->_right;
			}

			_node = subRight;
		}
		else
		{
			// 孩子是父亲的右的那个节点
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_left)
			{
				cur = cur->_parent;
				parent = parent->_parent;
			}

			_node = parent;
		}

		return *this;
	}

	Self& operator++()
	{
		if (_node->_right)
		{
			// 右树的最左节点(最小节点)
			Node* subLeft = _node->_right;
			while (subLeft->_left)
			{
				subLeft = subLeft->_left;
			}

			_node = subLeft;
		}
		else
		{
			Node* cur = _node;
			Node* parent = cur->_parent;
			// 找孩子是父亲左的那个祖先节点，就是下一个要访问的节点
			while (parent && cur == parent->_right)
			{
				cur = cur->_parent;
				parent = parent->_parent;
			}

			_node = parent;
		}

		return *this;
	}
};
```

在红黑树文件里，先按部就班写一个基础的结构：
里面有普通和const两种`不同类型`(这点很重要，本质上普通和const是两种完全不同的类型)的迭代器类型。
```cpp
//同一个类模版，但实例化出的不同类型，所以要提供两个版本的迭代器返回
	typedef __TreeIterator<T,T*,T&> iterator;
	typedef __TreeIterator<T,const T*,const T& > const_iterator;
	// const_iterator

	iterator begin()
	{
		Node* leftMin = _root;
		while (leftMin && leftMin->_left)
		{
			leftMin = leftMin->_left;
		}

		return iterator(leftMin);
	}

	iterator end()
	{
		return iterator(nullptr);
	}
	const_iterator begin() const
	{
		Node* leftMin = _root;
		while (leftMin && leftMin->_left)
		{
			leftMin = leftMin->_left;
		}

		return iterator(leftMin);
	}

	const_iterator end() const
	{
		return iterator(nullptr);
	}

```

map里添加：
```cpp
       //map和set不一样，value是可以修改的。 
		typedef typename RBTree<k, pair<k,v>, MapKeyOfT>::iterator iterator;
		typedef typename RBTree<k, pair<const k, v>,MapKeyOfT>::const_iterator const_iterator;
		//只给k添加const修饰就可以保证k不可被修改
		
			pair<iterator,bool> Insert(const pair<k, v>& kv) {
				return _t.Insert(kv);
		}
			iterator begin() {  
				return _t.begin();
			}
			iterator end() {
				return _t.end();
			}
			const_iterator begin() const {
				return _t.begin();
			}
			const_iterator end() const {
				return _t.end();
			}
			v& operator[](const k& key)
			{
				pair<iterator, bool> ret = insert(make_pair(key, v()));
				return ret.first->second;
			}
```

set:
```cpp
        //可以只提供const版本 
		typedef typename RBTree<k, k, SetKeyOfT>::const_iterator iterator;
		typedef typename RBTree<k, k, SetKeyOfT>::const_iterator const_iterator;

		pair<iterator,bool> Insert(const k& key) {  //会报错，因为这里的iterator是const版本，而insert是普通版本。
		    	return _t.Insert(key);
			
		}
		iterator begin() {
			return _t.begin();
		}
		iterator end() {
			return _t.end();
		}
		//set只提供const，不支持修改
		const_iterator begin() const{
			return _t.begin();
		}
		const_iterator end() const {
			return _t.end();
		}
```

这样写就会发生报错，因为Insert函数里return的是一个`普通迭代器类型`，而pair<iterator,bool>是一个`const类型`，二者的类型是不同的，所以会报错：

>错误	C2440	“return”: 无法从“__TreeIterator<T,T *,T &>”转换为“__TreeIterator<T,const T *,const T &>”	

解决方案：提供一个复制函数即可。
```cpp
__TreeIterator(const Iterator& it)
		:_node(it._node)
	{}
```
同时修改Insert函数：
```cpp
pair<iterator,bool> Insert(const k& key) {  //会报错，因为这里的iterator是const版本，而insert是普通版本。
		    //	return _t.Insert(key);
			pair<RBTree<k, k, SetKeyOfT>::iterator, bool> ret = _t.Insert(key);  //先用一个普通迭代器接受数据
			return pair<iterator, bool>(ret.first, ret.second);  //将first从普通转变为const
		}
```



# 完整代码

## rbtree.h

```cpp
#pragma once
#include<iostream>
using namespace std;

enum Colour
{
	RED,
	BLACK
};

template<class T>
struct RBTreeNode
{
	RBTreeNode<T>* _left;
	RBTreeNode<T>* _right;
	RBTreeNode<T>* _parent;

	T _data;
	Colour _col;

	RBTreeNode(const T& data)
		:_left(nullptr)
		, _right(nullptr)
		, _parent(nullptr)
		, _data(data)
		, _col(RED)
	{}
};

template<class T,class Ptr,class Ref>
struct __TreeIterator
{
	typedef RBTreeNode<T> Node;
	typedef __TreeIterator<T, Ptr, Ref> Self;
	typedef __TreeIterator<T,T*,T&> Iterator;
	Node* _node;

	__TreeIterator(Node* node)
		:_node(node)
	{}
	__TreeIterator(const Iterator& it)
		:_node(it._node)
	{

	}
	Ref operator*()
	{
		return _node->_data;
	}

	Ptr operator->()
	{
		return &_node->_data;
	}

	bool operator!=(const Self& s)
	{
		return _node != s._node;
	}
	Self& operator--()
	{
		if (_node->_left)
		{
			Node* subRight = _node->_left;
			while (subRight->_right)
			{
				subRight = subRight->_right;
			}

			_node = subRight;
		}
		else
		{
			// 孩子是父亲的右的那个节点
			Node* cur = _node;
			Node* parent = cur->_parent;
			while (parent && cur == parent->_left)
			{
				cur = cur->_parent;
				parent = parent->_parent;
			}

			_node = parent;
		}

		return *this;
	}

	Self& operator++()
	{
		if (_node->_right)
		{
			// 右树的最左节点(最小节点)
			Node* subLeft = _node->_right;
			while (subLeft->_left)
			{
				subLeft = subLeft->_left;
			}

			_node = subLeft;
		}
		else
		{
			Node* cur = _node;
			Node* parent = cur->_parent;
			// 找孩子是父亲左的那个祖先节点，就是下一个要访问的节点
			while (parent && cur == parent->_right)
			{
				cur = cur->_parent;
				parent = parent->_parent;
			}

			_node = parent;
		}

		return *this;
	}
};

// set->RBTree<K, K, SetKeyOfT> _t;
// map->RBTree<K, pair<K, V>, MapKeyOfT> _t;

template<class K, class T, class KeyOfT>
struct RBTree
{
	typedef RBTreeNode<T> Node;
public:
	//同一个类模版，但实例化出的不同类型，所以要提供两个版本的迭代器返回
	typedef __TreeIterator<T,T*,T&> iterator;
	typedef __TreeIterator<T,const T*,const T& > const_iterator;
	// const_iterator

	iterator begin()
	{
		Node* leftMin = _root;
		while (leftMin && leftMin->_left)
		{
			leftMin = leftMin->_left;
		}

		return iterator(leftMin);
	}

	iterator end()
	{
		return iterator(nullptr);
	}
	const_iterator begin() const
	{
		Node* leftMin = _root;
		while (leftMin && leftMin->_left)
		{
			leftMin = leftMin->_left;
		}

		return iterator(leftMin);
	}

	const_iterator end() const
	{
		return iterator(nullptr);
	}


	Node* Find(const K& key)
	{
		Node* cur = _root;
		KeyOfT kot;
		while (cur)
		{
			if (kot(cur->_data) < key)
			{
				cur = cur->_right;
			}
			else if (kot(cur->_data) > key)
			{
				cur = cur->_left;
			}
			else
			{
				return cur;
			}
		}

		return nullptr;
	}

	pair<iterator,bool> Insert(const T& data) //t是k还是pair都行
	{
		if (_root == nullptr)
		{
			_root = new Node(data);
			_root->_col = BLACK;
			return make_pair(iterator(_root),true);
		}

		Node* parent = nullptr;
		Node* cur = _root;

		KeyOfT kot;
		while (cur)
		{
			if (kot(cur->_data) < kot(data)) //这里是为了解决pair中key无法取出的问题，pair在库中的比较不是我们想要的方式，我们期望按first比。而库会比较first和second
			{
				parent = cur;
				cur = cur->_right;
			}
			else if (kot(cur->_data) > kot(data))
			{
				parent = cur;
				cur = cur->_left;
			}
			else
			{
				return make_pair(iterator(cur),false);
			}
		}

		cur = new Node(data);
		cur->_col = RED;
		Node* newnode = cur;
		if (kot(parent->_data) < kot(data))
		{
			parent->_right = cur;
		}
		else
		{
			parent->_left = cur;
		}

		cur->_parent = parent;

		while (parent && parent->_col == RED)
		{
			Node* grandfather = parent->_parent;
			if (parent == grandfather->_left)
			{
				Node* uncle = grandfather->_right;
				// u存在且为红
				if (uncle && uncle->_col == RED)
				{
					// 变色
					parent->_col = uncle->_col = BLACK;
					grandfather->_col = RED;

					// 继续向上处理
					cur = grandfather;
					parent = cur->_parent;
				}
				else // u不存在 或 存在且为黑
				{
					if (cur == parent->_left)
					{
						
						RotateR(grandfather);
						parent->_col = BLACK;
						grandfather->_col = RED;
					}
					else
					{
						
						
						RotateL(parent);
						RotateR(grandfather);

						cur->_col = BLACK;
						grandfather->_col = RED;
					}

					break;
				}
			}
			else // parent == grandfather->_right
			{
				Node* uncle = grandfather->_left;
				// u存在且为红
				if (uncle && uncle->_col == RED)
				{
					// 变色
					parent->_col = uncle->_col = BLACK;
					grandfather->_col = RED;

					// 继续向上处理
					cur = grandfather;
					parent = cur->_parent;
				}
				else
				{
					if (cur == parent->_right)
					{
					
						RotateL(grandfather);
						grandfather->_col = RED;
						parent->_col = BLACK;
					}
					else
					{
			
						RotateR(parent);
						RotateL(grandfather);
						cur->_col = BLACK;
						grandfather->_col = RED;
					}

					break;
				}
			}
		}

		_root->_col = BLACK;

		return make_pair(iterator(newnode),true);
	}

	void RotateL(Node* parent)
	{
		++_rotateCount;

		Node* cur = parent->_right;
		Node* curleft = cur->_left;

		parent->_right = curleft;
		if (curleft)
		{
			curleft->_parent = parent;
		}

		cur->_left = parent;

		Node* ppnode = parent->_parent;

		parent->_parent = cur;


		if (parent == _root)
		{
			_root = cur;
			cur->_parent = nullptr;
		}
		else
		{
			if (ppnode->_left == parent)
			{
				ppnode->_left = cur;
			}
			else
			{
				ppnode->_right = cur;

			}

			cur->_parent = ppnode;
		}
	}


	void RotateR(Node* parent)
	{
		++_rotateCount;

		Node* cur = parent->_left;
		Node* curright = cur->_right;

		parent->_left = curright;
		if (curright)
			curright->_parent = parent;

		Node* ppnode = parent->_parent;
		cur->_right = parent;
		parent->_parent = cur;

		if (ppnode == nullptr)
		{
			_root = cur;
			cur->_parent = nullptr;
		}
		else
		{
			if (ppnode->_left == parent)
			{
				ppnode->_left = cur;
			}
			else
			{
				ppnode->_right = cur;
			}

			cur->_parent = ppnode;
		}
	}

	

private:
	Node* _root = nullptr;

public:
	int _rotateCount = 0;
};

```

## set.h
```cpp
#pragma once
#include "rbtree.h"

namespace chengzi {
	template<class k>
	class set {
		struct SetKeyOfT {
			const k& operator()(const k& key) {
				return key;
			}
		};
	private:
		RBTree<k, k, SetKeyOfT> _t;
	public:
		//可以只提供const版本 
		typedef typename RBTree<k, k, SetKeyOfT>::const_iterator iterator;
		typedef typename RBTree<k, k, SetKeyOfT>::const_iterator const_iterator;

		pair<iterator,bool> Insert(const k& key) {  //会报错，因为这里的iterator是const版本，而insert是普通版本。
		    //	return _t.Insert(key);
			pair<RBTree<k, k, SetKeyOfT>::iterator, bool> ret = _t.Insert(key);  //先用一个普通迭代器接受数据
			return pair<iterator, bool>(ret.first, ret.second);  //将first从普通转变为const
		}
		iterator begin() {
			return _t.begin();
		}
		iterator end() {
			return _t.end();
		}
		//set只提供const，不支持修改
		const_iterator begin() const{
			return _t.begin();
		}
		const_iterator end() const {
			return _t.end();
		}
	};
}
```

## map.h
```cpp
#pragma once
#include "rbtree.h"

namespace chengzi {
	template<class k,class v>
	class map {
		struct MapKeyOfT {
			const k& operator()(const pair<k, v>& kv) {
				return kv.first;
			}
		};
	private:     
		RBTree<k, pair< k,v>,MapKeyOfT> _t; //如果只传value，那么无法使用const key_type去接受参数，比如find函数，他需要的是key，而map只有value是无法提取的。
	public:
		//map和set不一样，value是可以修改的，所以不能套用set的方法。 
		typedef typename RBTree<k, pair<k,v>, MapKeyOfT>::iterator iterator;
		typedef typename RBTree<k, pair<const k, v>, MapKeyOfT>::const_iterator const_iterator;
		//只给k添加const修饰就可以保证k不可被修改
		
			pair<iterator,bool> Insert(const pair<k, v>& kv) {
				return _t.Insert(kv);
		}
			iterator begin() {  
				return _t.begin();
			}
			iterator end() {
				return _t.end();
			}
			const_iterator begin() const {
				return _t.begin();
			}
			const_iterator end() const {
				return _t.end();
			}
			v& operator[](const k& key)
			{
				pair<iterator, bool> ret = insert(make_pair(key, v()));
				return ret.first->second;
			}
		};
	
}
```