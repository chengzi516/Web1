---
title: 【算法】链式二叉树里的遍历与分治思想
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E7%AE%97%E6%B3%95.png
tags:
- 算法
- 二叉树
- 分治
- c语言
swiper_index: 8
categories: 
- 数据结构与算法
abbrlink: da3f1096
date: 2023-03-26 12:43:26
ai: ture
---
> 在上一篇里我讲到了使用数组来模拟二叉树，也就是物理结构虽然是动态开辟的`数组`，但其逻辑上我们却认为其是一棵`二叉树`，每个节点都与对应的节点有着父或者子的关系。这一篇使用的是`链式存储`的二叉树，使用`结构体`为每个节点开辟空间，讲解四种`遍历`，并使用到了`分治的思想`来求解二叉树的节点个数与高度等问题。

#  💑链式存储
定义一个结构体，其包含值（本章定位int），两个指针，分别指向其两个孩子，也可以没有指向，当不存在左孩子或者右孩子时，其指针`指向NULL`。
用图表示其关系即如下：
<img src='https://imgbed.link/file/20652'>
```c
typedef struct binarytreenode {
	int val;
	struct binarytreenode* left;
	struct binarytreenode* right;
}btnode;
```
这一步很常规，没有什么值得深入的。
再写一个创建节点的函数。
```c
btnode* creatnode(int x) {
	btnode* newnode = (btnode*)malloc(sizeof(btnode));
	newnode->left = NULL;
	newnode->right = NULL;
	newnode->val = x;
	return newnode;

}
```
返回类型也可以置为void，传参采用`二级指针`的形式。


#  👨‍❤️‍👨四种遍历
链式二叉树有四种基本的遍历方法，分别为：
1. 层序遍历
2. 前序遍历
3. 中序遍历
4. 后序遍历

##   👨‍👩‍👧‍👦层序遍历
层序遍历就是直接的从`第一层`开始往下按`顺序`遍历。
比如如下图的结构：
<img src='https://imgbed.link/file/20653'>
得到的遍历结果就是
1->3->7->4->5->6->7
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E5%B1%82%E5%BA%8F%E9%81%8D%E5%8E%86.png'>
因为是链式存储，所以在此处可以用到队列的思想，先将根节点入队，然后入左孩子，右孩子，再弹出根节点，此时左孩子就是队头，入队左孩子的两个孩子，再弹出左孩子······这样就可以实现层序遍历了。如果是数组型的二叉树，直接挨个访问下标即可。
代码实现如下：

```c
void levelorder(btnode* root) {
	assert(root);
    if (root != NULL)
    {
        push(root->val);   //根节点进队列
    }

    while (!empty())  //队列不为空判断
    {
        printf("%d->", root->val);

        if (root->left)   //如果有左孩子，leftChild入队列
        {
            push(root->left->val);
            root = root->left;
        }

        if (root->right)   //如果有右孩子，rightChild入队列
        {
            push(root->right->val);
            if (!(root->left))
                root = root->right;
        }
        pop();  //已经遍历过的节点出队列

    }
}
```

##  👪前序遍历
前序遍历的规则是：`先根节点，再访问左子树，最后右子树`
仍然沿用上面的二叉树，其前序遍历的结果为：
1-> 3-> 4-> 7-> NULL-> NULL-> NULL-> 5-> NULL-> NULL-> 7-> 6-> NULL-> NULL-> NULL
在访问时总是会优先的访问根节点，然后去遍历其左右子树，当左子树遍历完才会去遍历右子树。当遍历第一棵左子树时，左子树的头就变成了`新的根节点`，新根节点也有他自己的左右子树，挨着遍历即可。其实就是递归的思想，可以尝试画图理解。
借用一张前序遍历动图([来源](https://xiaozhuanlan.com/topic/7869231504))：
<img src='https://images.xiaozhuanlan.com/photo/2019/fa65550a2db8de371ee88581d7fa24f1.gif'>
代码实现如下：
```c
void PreOrder(btnode* root) {
    if (root == NULL) {
        printf("NULL ");
        return;
    }
    printf("%d ", root->val);
    PreOrder(root->left);
    PreOrder(root->right);
}
```


## 👨‍👩‍👧‍👦中序与后序遍历
有了前序的基础，中序和后序也是找猫画虎即可。
中序的遍历规则就是：`先遍历左子树，再访问根节点，最后遍历右子树`。
后序的遍历规则就是：`先遍历左子树，再遍历右子树，最后访问根节点`。
仍然使用上图：
<img src='https://imgbed.link/file/20653'>
中序遍历结果：NULL->7->NULL->4->NULL->3->NULL->5->NULL->1->NULL->6->NULL->7->NULL
代码如下：
```c
void InOrder(btnode* root) {
    if (root == NULL) {
        printf("NULL->");
        return;
    }
    InOrder(root->left);
    printf("%d->", root->val);
    InOrder(root->right);
}
```
后序遍历结果：NULL->NULL->7->NULL->4->NULL->NULL->5->3->NULL->NULL->6->NULL->7->1
代码如下：

```c
void PostOrder(btnode* root)
{
    if (root == NULL)
    {
        printf("NULL->");
        return;
    }

    PostOrder(root->left);
    PostOrder(root->right);
    printf("%d->", root->val);
}
```
也附上两张动图：
<img src='https://images.xiaozhuanlan.com/photo/2019/9994375b6bf2cd4d0d6a2417b37c3d3f.gif'>
<img src='https://images.xiaozhuanlan.com/photo/2019/c9a33d3aeb741b82c297a0f45739387c.gif'>

# 👨‍❤️‍👨分治思想
分治的思想在链式结构的二叉树里应用非常多。
其本质上就是强调`分而治之`，将一个大问题分解为一个个的小问题。
举一个很形象的例子：现在一个学校正在统计学校的总人数，最后将统计结果交给校长。
那么校长肯定不会亲自去数学校的人数（校长也是很忙的）。
任务首先会被分派到每个学院的院长，然后院长再分给每个班的班长，班长再将任务分给室长，就这样层层的下发分解，最终就会将一个复杂的问题化解为简单的问题了。
## 👩‍👩‍👧‍👦求二叉树的节点个数
将根节点比作校长，往下的子节点就是院长，室长······
当然，这里也可以使用上面提到的遍历来计数，添加一个计数器即可。但我并不推荐这么做。
首先，计数器往哪里添加？添加到结构体，会增加大量不必要空间，添加到函数里，由于`形参是实参的一份拷贝`，需要传指针，添加到函数外部，就需要使用全局变量，在`多线程`问题下或者`频繁`使用此函数时，全局变量稍有一点使用不当就会造成错误。
所以我推荐在二叉树多采取`分治思想来代替遍历`处理问题。分治可以看作是一种另类的遍历，但其可操作性和可读性比遍历更优。
代码如下：
```c
int treesize(btnode* root) {
    if (root == NULL)
        return 0;
    return treesize(root->left) + treesize(root->right) + 1;
}
```
还是用开头的例子：
1把任务分给了3和7，在3和7完成计数后，`再加上一个本身`就可以得到总数。
体现在treesize(root->left) + treesize(root->right) + 1处。
那么3与7为了完成任务就需要把任务下发给4，5，6，当遇到NULL即返回，不进行加数。
<img src='https://imgbed.link/file/20655'>

## 👨‍👨‍👦求二叉树高度
同理，求整棵二叉树的高度等于先让左右二子树去计算其高度，计算完后+1即可。
```c
int TreeHeight(btnode* root)
{
    if (root == NULL)
        return 0;
    int leftHeight = TreeHeight(root->left);
    int rightHeight = TreeHeight(root->right);
    return leftHeight > rightHeight ? leftHeight + 1 : rightHeight + 1;
}
```
因为计算的是高度，所以需要取左右子树的`较大`的那一个即可。

## 👩‍👧‍👧求二叉树第k层的节点个数
求第k层的节点数，等于让根节点的下一层去求`他的第k-1层`的节点个数。
```c
int treeklevel(btnode* root, int k) {
    if (root == NULL)
        return 0;
    if (k == 1)
        return 1;

    return TreeKLevel(root->left, k - 1)+ TreeKLevel(root->right, k - 1);

}
```
求的是指定层的节点个数，那么当来到第k层时即可`返回1`，没有节点则返回0。
## 👩‍👩‍👦单值二叉树
如果二叉树每个节点都具有相同的值，那么该二叉树就是单值二叉树。
只有给定的树是单值二叉树时，才返回 true；否则返回 false。
示例 1：
输入：[1,1,1,1,1,null,1]
输出：true
示例 2：
输入：[2,2,2,5,2]
输出：false

```c
bool isUnivalTree(struct TreeNode* root){
  if(root==NULL)
    return true;
  if(root->left && root->left->val!=root->val)
   return false;  
  if(root->right && root->right->val!=root->val)
   return false;
   
   return isUnivalTree(root->left)&&isUnivalTree(root->right);   
}
```
用root与其左右孩子相比，相同即往下接着比且返回true，不相同返回false，新的root就是左右子树的根节点，不断向下遍历。
## 👩‍👧相同的树
给你两棵二叉树的根节点 p 和 q ，编写一个函数来检验这两棵树是否相同。
如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

示例 1：
<img src='https://assets.leetcode.com/uploads/2020/12/20/ex1.jpg'>
输入：p = [1,2,3], q = [1,2,3]
输出：true
示例 2：
<img src='https://assets.leetcode.com/uploads/2020/12/20/ex2.jpg'>

输入：p = [1,2], q = [1,null,2]
输出：false
示例 3：
<img src='https://assets.leetcode.com/uploads/2020/12/20/ex3.jpg'>
输入：p = [1,2,1], q = [1,1,2]
输出：false
```c
bool isSameTree(struct TreeNode* p, struct TreeNode* q){
     if(p==NULL&&q==NULL)
      return true;
      else if(p==NULL||q==NULL)
      return false;
      else if(p->val!=q->val)
      return false;
      else
      return isSameTree(p->left,q->left) && isSameTree(p->right,q->right);
}
```

## 👩‍👧‍👦翻转二叉树
给你一棵二叉树的根节点 root ，翻转这棵二叉树，并返回其根节点。
示例 1：
<img src='https://assets.leetcode.com/uploads/2021/03/14/invert1-tree.jpg'>
输入：root = [4,2,7,1,3,6,9]
输出：[4,7,2,9,6,3,1]
示例 2：
<img src='https://assets.leetcode.com/uploads/2021/03/14/invert2-tree.jpg'>


输入：root = [2,1,3]
输出：[2,3,1]
示例 3：

输入：root = []
输出：[]

将树的指向进行交换即可。
```c
void _invertTree(struct TreeNode* root){
    if(root){
        struct TreeNode* tmp = root->left;
        root->left = root->right;
        root->right = tmp;
        _invertTree(root->left);
        _invertTree(root->right);
    }
}
 
struct TreeNode* invertTree(struct TreeNode* root){
    _invertTree(root);
    return root;
}
 
```

## 😒对称二叉树
给你一个二叉树的根节点 root ， 检查它是否轴对称。

示例 1：

<img src='https://assets.leetcode.com/uploads/2021/02/19/symtree1.jpg'>
输入：root = [1,2,2,3,4,4,3]
输出：true
示例 2：
<img src='https://assets.leetcode.com/uploads/2021/02/19/symtree2.jpg'>

输入：root = [1,2,2,null,3,null,3]
输出：false

思路很简单，仍然是进行分治，不断往下比较。
```c
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
bool issame(struct TreeNode* t1,struct TreeNode* t2){
    if(t1==NULL && t2==NULL)
      return true;
    else if(t1==NULL || t2==NULL)
      return false;  
    if(t1->val==t2->val)
    return    issame(t1->left,t2->right) && issame(t1->right,t2->left);

    return false;  
    
}
bool isSymmetric(struct TreeNode* root){
    if(root==NULL)
      return true;
    else 
     return issame(root->left,root->right);
}
```

## 🤡另一棵树的子树
给你两棵二叉树 root 和 subRoot 。检验 root 中是否包含和 subRoot 具有相同结构和节点值的子树。如果存在，返回 true ；否则，返回 false 。

二叉树 tree 的一棵子树包括 tree 的某个节点和这个节点的所有后代节点。tree 也可以看做它自身的一棵子树。

示例 1：

<img src='https://assets.leetcode.com/uploads/2021/04/28/subtree1-tree.jpg'>
输入：root = [3,4,5,1,2], subRoot = [4,1,2]
输出：true
示例 2：

<img src='https://assets.leetcode.com/uploads/2021/04/28/subtree2-tree.jpg'>
输入：root = [3,4,5,1,2,null,null,null,null,0], subRoot = [4,1,2]
输出：false

```c
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     struct TreeNode *left;
 *     struct TreeNode *right;
 * };
 */
//这里会用到之前一道判定两树是否相同的函数，不断往下遍历即可。
bool issametree(struct TreeNode* tr1,struct TreeNode* tr2){
       if(tr1==NULL&&tr2==NULL)
         return true;
       else if(tr1==NULL || tr2==NULL)
          return false;
       else if(tr1->val!=tr2->val)
         return false;
       else 
     return issametree(tr1->left,tr2->left)&& issametree(tr1->right,tr2->right);
}
bool isSubtree(struct TreeNode* root, struct TreeNode* subRoot){
       if(root==NULL)
       return false;
       
       if(issametree(root,subRoot))
         return true;
        else if(root->left==NULL&&root->right==NULL)
          return false;
        else
        return isSubtree(root->left,subRoot) || isSubtree(root->right,subRoot);
}
```

