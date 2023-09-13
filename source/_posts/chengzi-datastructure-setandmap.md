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
