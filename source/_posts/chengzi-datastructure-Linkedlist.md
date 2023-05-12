---
title: 【数据结构】单链表的实现
date: 2023-05-02 13:42:51
tags:
- 链表
- 数据结构
- c++
categories: 
- 数据结构与算法
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.jpg
ai: ture
---


> 本篇内容为：使用c++实现单链表。


单链表（Singly Linked List），又称单向链表或者链表，是一种常用的数据结构。它由一连串的节点组成，每个节点包括两部分：`数据域和指针域`。数据域存储`节点的数据`，指针域存储`下一个节点的地址`。通过指针连接各个节点，从而形成链表。
单链表是一种常用的数据结构，具有`动态性和空间利用率高`的特点，可以应用于线性表、队列和栈等场景，并且也可以实现图的遍历和文件系统等应用。在实际开发中，需要根据具体场景选择不同的数据结构，以提高程序的效率和可维护性。下面我们将详细介绍单链表的基本概念、特点、操作及其应用。


<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E5%8D%95%E9%93%BE%E8%A1%A8%E5%AE%9E%E7%8E%B01.png'>



# :bamboo: 链表的一些概念

节点：链表中的每个数据元素都被称为节点。每个节点包含两个部分：`数据域和指针域`。

数据域：节点中存储的数据，可以是`任何类型的数据`，例如：整数、浮点数、字符、结构体等。

指针域：节点中存储的指针，`指向下一个节点的地址`。

头结点：链表中第一个节点之前的一个节点，它的指针域指向第一个节点。

尾节点：链表中最后一个节点，它的指针域指向空地址。

空链表：不包含任何节点的链表。

# :christmas_tree: 链表的优缺点

随机访问：单链表不支持随机访问，只能从`头结点开始遍历`链表，依次访问每个节点。

动态性：单链表的长度可以动态变化，可以根据需要进行插入、删除等操作。

空间利用率高：单链表的节点可以动态分配，不会浪费空间。

插入和删除操作快：单链表插入和删除操作只需要改变节点的指针域，时间复杂度为O(1)。


# :gift: 链表实现

## :tada: linklist类

在实现链表类前，还需要一个`node结构体`。每个node代表一个节点，而链表就是将每一个node按照一定的次序连接起来。
```c++
// 单个节点的结构体。
typedef struct node {
	struct node* next;
	int val;
	
}node;
```

linklist类：
```c++
class linklist
{
private:
    node* head;
public:
    //构造函数
    linklist();
    //析构函数
    ~linklist();
    //判空
    bool isempty();
    //取链表长度
    int length();
    //取指定位置元素
    node* getelem(int i);
    //查找元素位置
    node* findelem(int e);
   //队尾插入
    void Insert(int val);
   //指定位置插入
    void Insertindex(int val, int i);
   //删除指定位置
    void  Delete(int i);
};
```

head可以是头节点，也可以作为哨兵位。
哨兵节点是一种特殊的节点，它不存储数据，只用来简化链表的操作。将哨兵节点插入到链表的头部或尾部，可以使得链表的操作更加方便和高效。
<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E5%8D%95%E9%93%BE%E8%A1%A8%E5%AE%9E%E7%8E%B02.png'>
使用哨兵位可以避免很多问题。
其一，可以简化链表的操作。哨兵节点可以用来`避免链表为空`的情况，这样就可以不用考虑头结点和尾节点是否为空的情况。
其二，可以提高代码的效率。哨兵节点可以`避免很多判断语句`，从而提高代码的效率。
没有哨兵位，就表示链表`可能为空`，那么此时执行insert就要分情况判断，在c的实现中，甚至有时候因为要改变头节点的指向而不得不传递`二级指针`。

同样，哨兵位也是存在一些缺点的。
首先，需要`额外的内存空间`。哨兵节点需要占用额外的内存空间，这可能会导致内存的浪费，尤其是在链表的长度很小的情况下。
其次，可能会导致代码的复杂性增加。如果使用哨兵节点的不当，可能会导致代码的复杂性增加。

本文是将head当做哨兵位来实现的。

## :santa: 函数实现

首先是构造函数和析构函数，因为涉及到malloc或者new，所以析构是需要重新写的。
构造函数：

```c++
linklist::linklist() {
	//创建一个哨兵位。
	linklist() {
        head = new node;
        head->next = NULL;
    }
}
```
head不需要val，因为它只充当哨兵位的角色。

析构函数：
```c++
~linklist() {
        node* p = head;
        while (p != NULL) {
            node* q = p->next;
            delete p;
            p = q;
        }
    }
```
设置了一个p指针指向head，并不断的将p想后遍历，一边遍历一边删除。但需要注意的是，在删除之前需要`先保存下一个节点`，不然删除了就找不到继续的路了。

<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E5%8D%95%E9%93%BE%E8%A1%A8%E5%AE%9E%E7%8E%B03.png'>

判断链表是否为空与长度判断是两个工具，常在其他的函数中用到。
```c++
 // 判空
    bool isempty() {
        return head->next == NULL;
    }
    // 取链表长度
    int length() {
        int len = 0;
        node* p = head->next;
        while (p != NULL) {
            len++;
            p = p->next;
        }
        return len;
    }
```

判空直接返回head->next == NULL即可，如果哨兵位后面没有节点，那此链表就一定是空了。
取长度则依次遍历即可。

接下来的两个功能还是很重要的，分别是`取指定坐标的元素节点`和`取指定元素的节点`。
例如有一个链表，1-》2-》3-》4
前者可以取出4个坐标的任意一个节点，后者可以在链表中寻找是否有合乎要求的值，有则返回此节点。
取指定位置元素:
```c++
 node* getelem(int i) {
        if (isempty()) {
            return NULL;
        }
        int j = 0;
        node* p = head->next;
        while (p != NULL && j < i) {
            j++;
            p = p->next;
        }
        return p;
    }
```
链表为空则返回。

取指定元素的节点：
```c++
  node* findelem(int e) {
        node* p = head->next;
        while (p != NULL && p->val != e) {
            p = p->next;
        }
        return p;
    }
```
这里是否为空可判可不判，哪怕为空函数也可以返回NULL值。


队尾插入函数：
```c++
void Insert(int val) {
        node* p = head;
        while (p->next != NULL) {
            p = p->next;
        }
        node* q = new node;
        q->val = val;
        q->next = NULL;
        p->next = q;
    }
```
有几点需要注意：
1. while里的语句是`p->next != NULL`而非p!=NULL,如果是后者，p会走到队尾节点的再后一个，也就是NULL，此时让NULL的next为q是非常错误的！
2. 不要忘了让`q的next置为NULL`！NULL是链表是否结束的标志。

<img src='https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E5%8D%95%E9%93%BE%E8%A1%A8%E5%AE%9E%E7%8E%B04.png'>

指定位置插入：
```c++
 void Insertindex(int val, int i) {
        if (isempty() || i < 1 || i > length()) {
            return;
        }
        node* p = getelem(i - 1);
        node* q = new node;
        q->val = val;
        q->next = p->next;
        p->next = q;
    }
```

这里的if里就用到了isempty和length两个工具来判断传入的`i和链表的合法性`。如果链表为空则返回，或者i输入不符合范围要求也返回。
需要注意的是`getelem(i - 1)`，例如我们要在第`四`个位置插入新节点，那么应该找的是第`三`个节点的位置才行。

删除也是同理：
```c++
 void Delete(int i) {
        if (isempty() || i < 1 || i > length()) {
            return;
        }
        node* p = getelem(i - 1);
        node* q = p->next;
        p->next = q->next;
        delete q;
    }
```

