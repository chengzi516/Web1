---
title: 【算法】二叉树里的层序遍历与完全二叉树的判断
cover: 'https://imgbed.link/file/21264'
tags:
  - 算法
  - 二叉树
  - c语言
  - 排序
swiper_index: 10
description: 填上一篇的层序遍历的坑。
categories:
  - 数据结构与算法
abbrlink: 84c9fbe7
date: 2023-03-31 10:51:47
---
> 本篇文章主要是填上一篇的层序遍历的坑，具体的讲解层序遍历的代码实现与完全二叉树的判断。

# 💿思想
在之前就已经提到过，层序遍历就是`一层一层`的按顺序遍历，也是几种遍历中最简单明了的，但其实现相比于前序中序这种巧用`递归`思想的在代码书写中却要复杂不少。
给出一棵树：
<img src='https://imgbed.link/file/21263'>
我采取的办法是建立一个`队列`，然后将`根节点`进行入队，此时队伍里就只有根节点，将根节点出队，同时将其`两个孩子`按顺序入队。
之后要执行的操作就是不断的进行`出队`,在出队的同时将出队元素的`两个孩子`进行入队，当`队列为空`后，整个出队的顺序就是层序遍历的顺序了。
<img src='https://imgbed.link/file/21264'>

# 📀队的函数
既然要入队，首先肯定是建立一个队列的结构及其功能的实现。
在上述的思想中，我们肯定会用到初始化队列，入队，出队，取队头四种功能。
还有`队的销毁`，这点很重要，为了防止内存泄漏，一定要牢记`malloc`的节点如果不用就要销毁掉！
先是书写队列的结构，那么考虑队列里`放什么类型`呢？答案是`指针`，如果存放的是树结构里`节点的值`，那么在出队后，将其左右孩子进行入队就会很麻烦，因为单一的值是`无法连通前后`的！
```c
typedef struct BinaryTreeNode* QDatatype;

typedef struct QueueNode
{
	struct QueueNode* next;
	QDatatype data;
}QNode;

typedef struct Queue
{
	QNode* head;
	QNode* tail;
	int size;
}Queue;
```
这里建立队列使用到了两个结构体，因为这里建立的队列是链式结构的，方便连通前后，要做成数组型的队列虽然简单，但之后的层序遍历就不太好写了。所以还是在这里稍微麻烦一点吧。
下面给出几种基本功能的实现。
```c
//队列的初始化
void QueueInit(Queue* pq)
{
	assert(pq);

	pq->head = pq->tail = NULL;
	pq->size = 0;
}
//队列的销毁
void QueueDestroy(Queue* pq)
{
	assert(pq);
	QNode* cur = pq->head;
	while (cur)
	{
		QNode* next = cur->next;
		free(cur);
		cur = next;
	}

	pq->head = pq->tail = NULL;
	pq->size = 0;
}
//入队操作
void QueuePush(Queue* pq, QDatatype x)
{
	QNode* newnode = (QNode*)malloc(sizeof(QNode));
	if (newnode == NULL)
	{
		perror("malloc fail");
		return;
	}
	newnode->data = x;
	newnode->next = NULL;

	if (pq->head == NULL)
	{
		assert(pq->tail == NULL);

		pq->head = pq->tail = newnode;
	}
	else
	{
		pq->tail->next = newnode;
		pq->tail = newnode;
	}

	pq->size++;
}
//出队操作
void QueuePop(Queue* pq)
{
	assert(pq);
	assert(pq->head != NULL);
	if (pq->head->next == NULL)
	{
		free(pq->head);
		pq->head = pq->tail = NULL;
	}
	else
	{
		QNode* next = pq->head->next;
		free(pq->head);
		pq->head = next;
	}

	pq->size--;
}
//判断队空
bool QueueEmpty(Queue* pq)
{
	assert(pq);

	return pq->size == 0;
}
//取队头元素
QDatatype QueueFront(Queue* pq)
{
	assert(pq);
	assert(!QueueEmpty(pq));

	return pq->head->data;
}
```

# 💽层序遍历
先将头结点入队。
```c
Queue q;
QueueInit(&q);
QueuePush(&q, root);
```
接下来的关键就是确定循环的建立，其终止条件为`队列为空`。
也就是先将队头出队，再将其孩子入队，不断循环，当最后一个孩子出队，队为空，即停止。
```c
while (!QueueEmpty(&q)) {
		BTNode* new = QueueFront(&q);
		QueuePop(&q);
		printf("%d->", new->_data);
		if (new->_left)
			QueuePush(&q, new->_left);
		if (new->_right)
			QueuePush(&q, new->_right);
	}
```
这里就体现出了链式队列的优越性，层次遍历就写的非常简单了。
还有销毁操作不要忘记。
```c
QueueDestroy(&q);
```

# ⏱完全二叉树的判断
之所以会在这里提到完全二叉树的判断，是因为其思想与层次是相似的。
完全二叉树的特点就是`在遇到第一个null节点后，就不会再遇到非null节点`！
这个特点就是解题的关键所在。
这里提供一种稍微简单的解题思路：将每个节点按顺序（层次）入队，当遇到`null`就立刻停止循环，接着遍历剩下的队列，只要出现了`非null节点`，就说明此二叉树不是完全二叉树。
代码实现如下（是返回0，非返回1）：
```c
int BinaryTreeComplete(BTNode* root) {
	Queue q;
	QueueInit(&q);
	QueuePush(&q, root);
	while (!QueueEmpty(&q))
	{
		BTNode* new = QueueFront(&q);
		QueuePop(&q);
		if (new == NULL)
			break;
		if (new->_left)
			QueuePush(&q, new->_left);
		else
			QueuePush(&q, NULL);
		if (new->_right)
			QueuePush(&q, new->_right);
		else
			QueuePush(&q, NULL);


	}
	while (!QueueEmpty(&q)) {
		if (QueueFront(&q) != NULL)
			return 1;
		else
			QueuePop(&q);
	}
	return 0;
	
}
```
可以看到其代码与层次遍历是相似的。
