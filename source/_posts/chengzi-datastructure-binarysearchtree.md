---
title: 【数据结构】二叉搜索树
date: 2023-08-03 15:27:46
tags:
- 数据结构
- c++
- 二叉树
categories:
- 数据结构与算法
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.jpg
ai: ture
---

# :aries:概念

二叉搜索树（Binary Search Tree，BST）是一种二叉树数据结构，它具备以下的几种特性：

1. 有序性： 对于树中的任意节点，左子树中的所有节点值都`小于`等于该节点的值，而右子树中的所有节点值都`大于`等于该节点的值。这也就意味着可以高效地进行`搜索、插入和删除`操作。相比于华而不实的二分查找，二叉搜索树更适用于实际一些，至少我是这么认为的。

2. 左右子树也是BST： 二叉搜索树的左子树和右子树也都是二叉搜索树。

3. 中序遍历有序： 对BST进行中序遍历，将会得到一个有序的节点值序列。

```
      8
     / \
    3   10
   / \    \
  1   6    14
     / \   /
    4   7  13

```

这是一个二叉搜索树。对于节点上的数字来说，左子树的所有节点值都小于等于该节点，而右子树的所有节点值都大于等于该节点。



因为二叉搜索树具备有序性，可以高效地进行搜索操作。从根节点开始，比较目标值与当前节点的值：

>若目标值等于当前节点的值，直接找到了目标节点；
若目标值小于当前节点的值，则在左子树中继续搜索；
若目标值大于当前节点的值，则在右子树中继续搜索。


在插入节点时，首先进行搜索操作，找到合适的位置。然后将新节点插入到相应的位置，保持BST的有序性。



删除节点时需要考虑不同情况：

1. 节点是叶子节点：直接删除该节点。
2. 节点有一个子节点：将子节点替代删除的节点。
3. 节点有两个子节点：找到其`右子树中最小`的节点或`左子树中最大`的节点，用该节点的值替换删除的节点，并删除该节点。

时间复杂度：

在平衡的二叉搜索树中，搜索、插入和删除操作的平均时间复杂度为O(log n)，其中n为树中节点的数量。但在最坏情况下（树退化成链表），时间复杂度会退化为O(n)。



二叉搜索树广泛应用于各种算法和数据结构，例如在数据库索引、哈希表等数据结构的实现中。
然而，需要注意的是，当数据集过于偏斜或有序时，BST的性能可能会下降。为了解决这个问题，人们提出了平衡二叉搜索树（如AVL树、红黑树等）来保持树的平衡性，从而保证各种操作的稳定性和高效性。这些会在之后的博客中给出具体的展示。

# :taurus:实现

## :gemini:非递归的普通实现（循环）

我个人是“能不用递归就不用递归”的原则的忠实信徒，因为递归的水真的很深，稍有不慎就会掉进坑里。
首先就是在使用递归时，如果递归深度太大，`堆栈可能会溢出`。并且递归函数在每次调用自身时都需要将当前的上下文信息（局部变量、返回地址等）保存在堆栈中，这会导致堆栈的不断增长，可能造成内存消耗较大，
其次则是可读性和维护性，递归可能会导致代码变得复杂，难以理解和维护。特别是对于复杂的递归调用，代码的逻辑可能会变得混乱，`不易于debug`。
所以就先用循环的方式来完成二叉搜索树。

先给出基本的结构：
因为二叉搜索树可以存放不同类型的值，所以模版是必要的。每个node都有左孩子，右孩子和该位置所存储的_key。_key可以是任何类型。

```cpp
template<class K>
struct BSTreeNode
{
	BSTreeNode<K>* _left;
	BSTreeNode<K>* _right;
	K _key;

	BSTreeNode(const K& key)
		:_left(nullptr)
		,_right(nullptr)
		,_key(key)
	{}
};

template<class K>
class BSTree
{
	typedef BSTreeNode<K> Node;
    public:
    BSTree()
      :_root(nullptr);
    {}
    private:
	Node* _root;
}    

```

插入操作
```cpp
bool Insert(const K& key)
	{    
        //如果树为空，则直接新建节点。也可以在构造函数里新建，看自己想法。
		if (_root == nullptr)
		{
			_root = new Node(key);
			return true;
		}

		Node* parent = nullptr;
		Node* cur = _root;
		while (cur)
		{
            //如果cur存储的值小于key，就往右走
			if (cur->_key < key)
			{
				parent = cur;
				cur = cur->_right;
			}  //同理，符合cur->_key > key就往左走
			else if(cur->_key > key)
			{
				parent = cur;
				cur = cur->_left;
			}
			else
			{
				return false;
			}
		}
        //当循环结束,cur就指向了某个节点的空孩子。直接新建节点然后与parent连接即可。

		cur = new Node(key);
		if (parent->_key < key)
		{
			parent->_right = cur;
		}
		else
		{
			parent->_left = cur;
		}

		return true;
	}

```

查找操作

```cpp
bool Find(const K& key)
	{
		Node* cur = _root;
		while (cur)
		{
			if (cur->_key < key)
			{
				cur = cur->_right;
			}
			else if (cur->_key > key)
			{
				cur = cur->_left;
			}
			else
			{
				return true;
			}
		}

		return false;
	}

```

删除操作

```cpp
bool Erase(const K& key)
	{
		Node* parent = nullptr;
		Node* cur = _root;

		while (cur)
		{
			if (cur->_key < key)
			{
				parent = cur;
				cur = cur->_right;
			}
			else if (cur->_key > key)
			{
				parent = cur;
				cur = cur->_left;
			}
			else // 找到了符合要求的节点
            //1. 节点是叶子节点：直接删除该节点。
            //2. 节点有一个子节点：将子节点替代删除的节点。
            //3. 节点有两个子节点：找到其`右子树中最小`的节点或`左子树中最大`的节点，用该节点的值替换删除的节点，并删除该节点
			{
				 // 左为空
				if (cur->_left == nullptr)
				{
					if (cur == _root) //不要漏掉删除节点为root的情况！
					{
						_root = cur->_right;
					}
					else
					{   //本质都是删除该节点让子节点来代替，但需要考虑子节点在左边还是右边！
						if (parent->_right == cur)
						{
							parent->_right = cur->_right;
						}
						else
						{
							parent->_left = cur->_right;
						}
					}
				}// 右为空
				else if (cur->_right == nullptr)
				{
					if (cur == _root)
					{
						_root = cur->_left;
					}
					else
					{
						if (parent->_right == cur)
						{
							parent->_right = cur->_left;
						}
						else
						{
							parent->_left = cur->_left;
						}
					}					
				} // 左右都不为空 
				else
				{
					// 找替代节点
					Node* parent = cur;
					Node* leftMax = cur->_left; //这里找的是左边最大的
					while (leftMax->_right)
					{
						parent = leftMax;
						leftMax = leftMax->_right;
					}

					swap(cur->_key, leftMax->_key);

					if (parent->_left == leftMax)
					{
						parent->_left = leftMax->_left;
					}
					else
					{
						parent->_right = leftMax->_left;
					}

					cur = leftMax;
				}

				delete cur;
				return true;
			}
		}

		return false;
	}


```

## :cancer:递归实现

insert操作

因为insert要改变指针的指向，所以传指针的引用或二级指针。

```cpp
bool _InsertR(Node*& root, const K& key)
		{
			if (root == nullptr)
			{
				root = new Node(key);
				return true;
			}

			if (root->_key < key)
			{
				return _InsertR(root->_right, key);
			}
			else if (root->_key > key)
			{
				return _InsertR(root->_left, key);
			}
			else
			{
				return false;
			}
		}

```

find操作

```cpp
bool _FindR(Node* root, const K& key)
		{
			if (root == nullptr)
				return false;

			if (root->_key < key)
			{
				return _FindR(root->_right, key);
			}
			else if (root->_key > key)
			{
				return _FindR(root->_left, key);
			}
			else
			{
				return true;
			}
		}

```

删除操作

```cpp
bool _EraseR(Node*& root, const K& key)
		{
			if (root == nullptr)
				return false;

			if (root->_key < key)
			{
				return _EraseR(root->_right, key);
			}
			else if (root->_key > key)
			{
				return _EraseR(root->_left, key);
			}
			else
			{
				Node* del = root;

				if (root->_left == nullptr)
				{
					root = root->_right; //这里用引用就无需考虑左右了
				}
				else if (root->_right == nullptr)
				{
					root = root->_left;
				}
				else
				{
					Node* leftMax = root->_left;
					while (leftMax->_right)
					{
						leftMax = leftMax->_right;
					}

					swap(root->_key, leftMax->_key);

					return _EraseR(root->_left, key);
				}

				delete del;
				return true;
			}
		}

```

使用传引用是很方便的，对照两次的代码：

```cpp
//递归写法
if (root->_left == nullptr)
				{
					root = root->_right; //这里用引用就无需考虑左右了
				}
//非递归写法
if (cur->_left == nullptr)
				{
					if (cur == _root) //不要漏掉删除节点为root的情况！
					{
						_root = cur->_right;
					}
					else
					{   //本质都是删除该节点让子节点来代替，但需要考虑子节点在左边还是右边！
						if (parent->_right == cur)
						{
							parent->_right = cur->_right;
						}
						else
						{
							parent->_left = cur->_right;
						}
					}
				}

```

>指针是可以改变其指向的，而引用不行。

当传递的是引用时，root是存储在要删除节点的父节点中的。所以我们改变root直接就改变了父节点的左或右孩子！
这点有点绕，理解不了就自己画图看看和指针的区别。



# :leo:搜索二叉树的实例用法

增加一个value值来实现统计次数或者中英翻译配对的功能。


```cpp
template<class K, class V>
	struct BSTreeNode
	{
		BSTreeNode<K, V>* _left;
		BSTreeNode<K, V>* _right;
		K _key;
		V _value;

		BSTreeNode(const K& key, const V& value)
			:_left(nullptr)
			, _right(nullptr)
			, _key(key)
			, _value(value)
		{}
	};

	template<class K, class V>
	class BSTree
	{
		typedef BSTreeNode<K, V> Node;
	public:
		BSTree()
			:_root(nullptr)
		{}

		void InOrder()
		{
			_InOrder(_root);
			cout << endl;
		}

		Node* FindR(const K& key)
		{
			return _FindR(_root, key);
		}

		bool InsertR(const K& key, const V& value)
		{
			return _InsertR(_root, key, value);
		}

		bool EraseR(const K& key)
		{
			return _EraseR(_root, key);
		}

	private:
		bool _EraseR(Node*& root, const K& key)
		{
			if (root == nullptr)
				return false;

			if (root->_key < key)
			{
				return _EraseR(root->_right, key);
			}
			else if (root->_key > key)
			{
				return _EraseR(root->_left, key);
			}
			else
			{
				Node* del = root;
				if (root->_left == nullptr)
				{
					root = root->_right;
				}
				else if (root->_right == nullptr)
				{
					root = root->_left;
				}
				else
				{
					Node* leftMax = root->_left;
					while (leftMax->_right)
					{
						leftMax = leftMax->_right;
					}

					swap(root->_key, leftMax->_key);

					return _EraseR(root->_left, key);
				}

				delete del;
				return true;
			}
		}

		bool _InsertR(Node*& root, const K& key, const V& value)
		{
			if (root == nullptr)
			{
				root = new Node(key, value);
				return true;
			}

			if (root->_key < key)
			{
				return _InsertR(root->_right, key, value);
			}
			else if (root->_key > key)
			{
				return _InsertR(root->_left, key, value);
			}
			else
			{
				return false;
			}
		}

		Node* _FindR(Node* root, const K& key)
		{
			if (root == nullptr)
				return nullptr;

			if (root->_key < key)
			{
				return _FindR(root->_right, key);
			}
			else if (root->_key > key)
			{
				return _FindR(root->_left, key);
			}
			else
			{
				return root;
			}
		}

		void _InOrder(Node* root)
		{
			if (root == NULL)
			{
				return;
			}

			_InOrder(root->_left);
			cout << root->_key << ":" << root->_value << endl;
			_InOrder(root->_right);
		}
	private:
		Node* _root;
	};

```

使用示例：

翻译
```cpp

        BSTree<string, string> dict;
		dict.InsertR("hello", "你好");
		dict.InsertR("world", "世界");
		dict.InsertR("computer", "计算机");
		dict.InsertR("science", "科学");

		string str;
		while (cin>>str)
		{
			BSTreeNode<string, string>* ret = dict.FindR(str);
			if (ret)
			{
				cout << ret->_value << endl;
			}
			else
			{
				cout << "找不到该单词！" << endl;
			}
		}

```

统计

```cpp

		string arr[] = { "小明", "小红", "小明", "小刚", "小红", "小红", "小明", "小红", "小明", "小红", "小刚" };
		BSTree<string, int> countTree;
		for (auto& str : arr)
		{
			auto ret = countTree.FindR(str);
			if (ret == nullptr)
			{
				countTree.InsertR(str, 1);
			}
			else
			{
				ret->_value++;
			}
		}

```