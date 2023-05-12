---
title: 【leetcode】巧解深度拷贝“带随机指针”的链表
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/1610173947-QjrHlC-LeetCode.png
tags:
  - 刷题
  - c语言
abbrlink: e7490f5a
date: 2023-03-15 21:23:18
categories: 
- 刷题
ai: ture
---
题目来源：leetcode [138.复制带随机指针的链表](https://leetcode.cn/problems/copy-list-with-random-pointer)
给你一个长度为 n 的链表，每个节点包含一个额外增加的随机指针 random ，该指针可以指向链表中的任何节点或空节点。

构造这个链表的 深拷贝。 深拷贝应该正好由 n 个 全新 节点组成，其中每个新节点的值都设为其对应的原节点的值。新节点的 next 指针和 random 指针也都应指向复制链表中的新节点，并使原链表和复制链表中的这些指针能够表示相同的链表状态。复制链表中的指针都不应指向原链表中的节点 。

例如，如果原链表中有 X 和 Y 两个节点，其中 X.random --> Y 。那么在复制链表中对应的两个节点 x 和 y ，同样有 x.random --> y 。

返回复制链表的头节点。

用一个由 n 个节点组成的链表来表示输入/输出中的链表。每个节点用一个 [val, random_index] 表示：

val：一个表示 Node.val 的整数。
random_index：随机指针指向的节点索引（范围从 0 到 n-1）；如果不指向任何节点，则为  null 。
你的代码 只 接受原链表的头节点 head 作为传入参数。

 

示例 1：

<img src='https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e1.png'>

输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
示例 2：

<img src='https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e2.png'>

输入：head = [[1,1],[2,1]]
输出：[[1,1],[2,1]]
示例 3：

<img src='https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e3.png'>

输入：head = [[3,null],[3,0],[3,null]]
输出：[[3,null],[3,0],[3,null]]

本题的解法思路有很多，其主要需要解决的问题在于`随机指针的复制`。

如果我们先将原本的链表本身进行一个复制，然后再去寻找每个节点的random指针，思路是可行的，但是很麻烦，其时间复杂度为$ O（n*n）$，有n个元素就需要遍历n遍。同时还需要设计一个计步器，因为我们进行的是`深度拷贝`,所以我们复制的链表的random只能指向复制的链表，而不可以指向原先的链表，所以需要遍历原先链表时，记录其每个节点的random节点`距离他本身多少步`，这样做无疑是麻烦的，而且很容易出错。本文提供思路是，先`复制整个链表的节点到原先的链表节点后面`。
<img src='https://imgbed.link/file/19742'>
再将每个复制的节点的random指向复制的相应的`random节点`，这一步是本题的核心，在后面会详细的讲解。
当复制完整个链表，就需要将其从原先的链表上`剥离`，再`恢复`原链表。
那么本题的主要思想就如上所示了，接下来进行每一步的代码书写。
以下是题目给出的接口函数。
```c
/**
 * Definition for a Node.
 * struct Node {
 *     int val;
 *     struct Node *next;
 *     struct Node *random;
 * };
 */
 /*要求复杂度为O（n）*/

struct Node* copyRandomList(struct Node* head) {}
```

第一步就是`复制`每一个节点到原先节点的后面。
```c
 struct Node* cur=head;
    while(cur){
        struct Node* newnode=(struct Node*)malloc(sizeof(struct Node));
        newnode->val=cur->val;
        newnode->next=cur->next;
        cur->next=newnode;
        cur=cur->next->next;
    }
```
一定要`单独`设定一个指针来进行链表的遍历哦，因为head会在后续频繁的被使用，如果现在就把head往后移了，之后要用就找不到`头`了。
我们在这里定义了一个指针cur，用cur来遍历整个链表，同时在每个原节点后面malloc一个新的节点，然后更改其指向，就如上面的第一张图所示。如果不熟悉链表的指向更改可以多设立一个指针，方便自己的理解。

第二步是核心，需要将每一个复制节点的random补齐。譬如我们的节点1的随机指针是是指向节点3的，那么此时将`复制`的节点1的random指向`原`节点3的next即可。
<img src='https://imgbed.link/file/19743'>
代码如下：
```c
 cur=head;
    while(cur){
        struct Node* copy=cur->next;
        if(cur->random == NULL)
          copy->random=NULL;
          else{
              copy->random=cur->random->next;  //重点语句
          }
          cur=cur->next->next;
    }
```
此时我们就又用到了头指针head，所以如果上面将head指向末尾了，现在就没得用了。
第三步就是将复制的链表转移出原链表，同时要恢复原链表，因为是深度拷贝，所以题目的要求是复现一个与原链表相同的链表，并返回其节点，同时保证原链表的不变。
在建立一个新链表时，可能会因为链表是否`为空`而分情况讨论，所以我们常常处理这类问题采用哑节点，为其临时分配一个`哨兵位`，从而使得链表`永不为空`，处理起来就会简单很多，但本题因为并不复杂，所以加不加哨兵位全看自己。
```c
cur=head;
    struct Node* copyhead=(struct Node*)malloc(sizeof(struct Node));  //哨兵位
    struct Node* copytail=copyhead;
    while(cur){
        struct Node* copy=cur->next;
        copytail->next=copy;
        copytail=copy;
       //恢复
       cur->next=cur->next->next;
       //继续
       cur=cur->next;
    }
```

最后返回copyhead->next即可。
需要注意力扣给出的用例可能为空，那么可单独提出来判断。
```c
	if(head==NULL)
     return NULL;
```


