---
title: 【数据结构】AVL树
date: 2023-09-08 21:21:46
tags:
- 数据结构
- c++
- 树
categories:
- 数据结构与算法
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.jpg
ai: ture
---

# 引入

>AVL树是一种自平衡的二叉搜索树，它可以在插入和删除操作之后自动调整树的结构，以保持树的`平衡性`。AVL树是由计算机科学家Adelson-Velsky和Landis于1962年提出的，它的名称来源于他们的姓氏的首字母。
AVL树是`平衡的搜索二叉树`。
在AVL树中，每个节点都有一个`平衡因子`，用来衡量左子树和右子树的`高度差`。平衡因子可以是`-1、0或1`，如果平衡因子超过这个范围，就表示树不平衡了。为了保持树的平衡性，AVL树使用`旋转操作`来调整树的结构。

下面是一个简单的AVL树的图例：
```
        11
       /  \
      7    16
     / \     \
    3   9     26
         \   /
         10  18
        
       
```
这个AVL树是平衡的，因为每个节点的左右子树的高度差不超过1。

平衡因子一般是`右子树高度减去左子树高度`。其取值只能是-1，0，1三个中的一个。

# 代码示例


## AVLTree类

```cpp
#pragma once

#include<iostream>
#include<assert.h>
using namespace std;


struct AVLTreeNode
{
	int _key;   
	AVLTreeNode<K, V>* _left;  //节点的左右孩子以及父节点都必须有所指向，后面会用
	AVLTreeNode<K, V>* _right;
	AVLTreeNode<K, V>* _parent;
	int _bf;  // balance factor 平衡因子

	AVLTreeNode(const int& key)
		:_key(key)
		,_left(nullptr)
		,_right(nullptr)
		,_parent(nullptr)
		,_bf(0)
	{}
};


class AVLTree
{
	typedef AVLTreeNode Node;
public:
	
private:
	Node* _root = nullptr;  //根节点
};

```

AVL树需要维持其左右子树高度差不超过1，对其子树也是同理，所以需要在insert函数对此条件进行约束。bf(平衡因子)就是控制代码逻辑的关键。

```cpp
bool Insert(const int& key)
	{
		if (_root == nullptr)  //如果是空树，则new一个，不需要其他操作
		{
			_root = new Node(kv);
			return true;
		}

		Node* parent = nullptr; //如果不是空树，则从根节点开始遍历，寻找合适的位置插入
		Node* cur = _root;
		while (cur)//不管bf，先找到一个坑位
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
			else
			{
				return false;
			}
		}

		cur = new Node(key); //创建新节点并与parent进行连接
		if (parent->_key < key)
		{
			parent->_right = cur;
		}
		else
		{
			parent->_left = cur;
		}

		cur->_parent = parent;
        //此时就需要检查高度差是否为1，-1,0中的一个了
		
```


无非也就是考虑几种情况。
当新增节点在右子树时，bf会增加，反之在左边的话，bf就会减小。
只要bf在1，0，-1三数中浮动，这棵树就是符合要求的，一旦出现2或者-2，就需要通过`旋转`操作来`降低高度差`。

更加细化下来：
当更新后parent的bf=0，则说明补齐了左子树或右子树中的一个，此时是`不影响`parent往上的分支的！
```
        11
       /  \
      7    16
     /  \    \
    3(new)9   26
```
如上，若新增的是3这个节点，则7的bf从1变到0，但不影响7的parent节点11的bf。

当更新后parent的bf=1或-1，说明树高度发生了变化，需要沿着parent的分支向上更新bf。

```
        11
       /  \
      7    16
     / \     \
    3   9     26(new)
```

如上，新增的是26这个节点，那么16的bf从0变到了1，11也受影响从-1变到了0，此时需要更新bf值，而无需改动树的结构。

那么当更新后出现了-2或者2的bf时，就说明树的结构遭到了破坏，需要通过一些操作来`降低高度差`。


接着往下写代码：

```cpp        
  
		while (parent)  //检查parent的bf
		{
			if (cur == parent->_left)
			{
				parent->_bf--;
			}
			else // if (cur == parent->_right)
			{
				parent->_bf++;
			}

			if (parent->_bf == 0)
			{
				// 更新结束
				break;
			}
            else if (parent->_bf == 1 || parent->_bf == -1)
			{
				// 继续往上更新
				cur = parent;
				parent = parent->_parent;
			}
			else if (parent->_bf == 2 || parent->_bf == -2)
			{
				// 子树不平衡了，需要旋转
				/* 
                  
                  
                                这里的代码就是avl的核心代码
                
                
                                */
				break;
			}
			else
			{
				assert(false);
			}
		}


		return true;
	}
```

解决bf的绝对值大于1通常使用旋转操作。常用的旋转操作有四种。
左旋（Left Rotation）：当一个节点的`右子树比左子树高度高`时，需要进行左旋操作。左旋操作将当前节点的右子节点提升为新的根节点，当前节点成为新根节点的左子节点，新根节点的左子节点成为当前节点的右子节点。

右旋（Right Rotation）：当一个节点的`左子树比右子树高度高`时，需要进行右旋操作。右旋操作将当前节点的左子节点提升为新的根节点，当前节点成为新根节点的右子节点，新根节点的右子节点成为当前节点的左子节点。

左右旋（Left-Right Rotation）：当一个节点的`左子树的右子树比左子树的左子树高度高`时，需要进行左右旋操作。左右旋操作先对当前节点的左子节点进行左旋操作，然后再对当前节点进行右旋操作。

右左旋（Right-Left Rotation）：当一个节点的`右子树的左子树比右子树的右子树高度高`时，需要进行右左旋操作。右左旋操作先对当前节点的右子节点进行右旋操作，然后再对当前节点进行左旋操作。


## 左旋

当在子树的`右子树的右子树`(RR型)上插入新节点时，需要使用左旋操作。

```
        11
       /  \
      7    16
           / \
         15   18
               \
                20(new)
```
此时11的bf会发生变化为2，不符合要求，需要进行旋转。
设11为parent，16为cur。
则核心操作为：
```cpp
parent->right = cur ->left
cur->left = parent
```
整棵树变化为

```
           16
           / \
          11  18
         /  \   \
        7    15  20(new)

```


<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/avltree1.jpg">

```cpp
void RotateL(Node* parent)
	{
		Node* cur = parent->_right;
		Node* curleft = cur->_left;

		parent->_right = curleft;
		
		if (curleft)
		{
			curleft->_parent = parent;  //写代码的时候别忘了parent的存在，要记得进行连接
		}

		cur->_left = parent;
        //调整后cur处于子树顶端，也需要连接到它的parent，如果cur==root，那就指向空
		Node* ppnode = parent->_parent; //找到parent的原始parent

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

		parent->_bf = cur->_bf = 0;  //记得调整bf
	}

```


## 右旋

右旋和左旋同理，不过是LL型，也就是插入到子树的左子树的左子树位置上。
核心操作为
```
parent->left =cur->right
cur->right = parent
```
<img src="https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/avltree2.jpg">

```cpp
void RotateR(Node* parent)
	{
		Node* cur = parent->_left;
		Node* curright = cur->_right;

		parent->_left = curright;
		if (curright)
			curright->_parent = parent;

		Node* ppnode = parent->_parent;
		cur->_right = parent;

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

		parent->_bf = cur->_bf = 0;
	}

```


>总结一下这两种简单的旋转，第一步就是看哪边高，哪边高了就通过旋转的办法降低高度，左边高了就往右边旋，右边高了就往左边旋，在旋转过程中别忘了节点的parent需要更新，对于cur的parent更新时，需要判断parent的parent来选择不同的代码逻辑。


## 左右旋

在某个子树的左子树的右子树上插入新节点时需要左右旋。(LR型)

```
          6                     6                      4
        / \                    / \                    / \
       2  7                   4   7                   2   6
       /  \     ---->先左旋   / \   ---->再右旋      /   /\
      1    4                 2  5                   1   5 7
            \               /
             5(new)        1
```



## 右左旋

同理，在某个子树的右子树的左子树上插入节点时需要右左旋。(RL型)

```cpp
void RotateRL(Node* parent)
	{
		Node* cur = parent->_right;
		Node* curleft = cur->_left;
		int bf = curleft->_bf;

		RotateR(parent->_right);
		RotateL(parent);

		if (bf == 0)
		{
			cur->_bf = 0;
			curleft->_bf = 0;
			parent->_bf = 0;
		}
		else if (bf == 1)
		{
			cur->_bf = 0;
			curleft->_bf = 0;
			parent->_bf = -1;
		}
		else if (bf == -1)
		{
			cur->_bf = 1;
			curleft->_bf = 0;
			parent->_bf = 0;
		}
		else
		{
			assert(false);
		}
	}

```


