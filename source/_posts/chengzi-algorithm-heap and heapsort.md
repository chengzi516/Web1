---
title: 【算法】堆和堆排序
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E7%AE%97%E6%B3%95.png
tags:
  - 算法
  - c语言
  - 二叉树
  - 排序
abbrlink: 967ab8a8
swiper_index: 6
categories: 
- 数据结构与算法 
date: 2023-03-21 21:44:28
ai: ture
---
# 📲堆

## 🏒概念
以下源自百度百科对`堆`的解释说明。
> 堆（heap）是计算机科学中一类特殊的数据结构的统称。堆通常是一个可以被看做`一棵树`的`数组对象`。堆总是满足下列性质：
堆中某个结点的值总是`不大于或不小于`其父结点的值；
堆总是一棵`完全二叉树`。
将根结点最`大`的堆叫做`最大堆`或大根堆，根结点最`小`的堆叫做`最小堆`或小根堆。常见的堆有二叉堆、斐波那契堆等。
堆是非线性数据结构，相当于`一维数组`，有两个直接后继。
若将和此次序列对应的一维数组（即以一维数组作此序列的存储结构）看成是一个完全二叉树，则堆的含义表明，完全二叉树中所有非终端结点的值均不大于（或不小于）其左、右孩子结点的值。由此，若序列{k1,k2,…,kn}是堆，则堆顶元素（或完全二叉树的根）必为序列中n个元素的最小值（或最大值）。
<img src='https://www.runoob.com/wp-content/uploads/2020/09/heap-01.png'>
（以上源自百度百科）

堆的定义如下：n个元素的序列{k1,k2,ki,…,kn}当且仅当满足下关系时，称之为堆。
$$ 
k_i<=k_{2*i+1}且k_i<=k_{2*i+2} 
$$
这种则被称为小堆，其父节点`小于`他的两个子节点。
$$ 
k_i<=k_{2*i+1}且k_i<=k_{2*i+2} 
$$
反之则被称为大堆。
因为堆的逻辑结构是一棵`完全二叉树`，所以虽然用数组的形式来存，但我们思考其实现要考虑以树的结构来做。这里的下标用到了二叉树的知识点，当一个节点为i时，其左右子节点（存在的话）为2 *  i+1与2 *  i+2,请注意下标从`0`开始计算。

堆的实现多种多样，本文给出一种简单可行的建堆方式。
## 🏸初始化堆
首先是实现堆的`初始化`。
但在实现堆的初始化要对其`结构`进行定义。
这里给出的是以`动态开辟内存`的数组来实现的，定义arr作为数组开始的`基址`，capacity为`总的容量`，size为`当前的容量`，二者一起控制整个数组的大小。
```c
typedef struct heap{
	int* arr;
	int capacity;
	int size;
}
```
书写初始化函数。
```c
void heapinit(heap* hp) {
	assert(hp); //添加适当的断言方便调试。
	hp->arr = (int*)malloc(sizeof(int) * 4);
	if (hp->arr == NULL)
		printf("malloc fail\n");
	hp->capacity = 4;
	hp->size = 0;
}
```
为arr申请四个int的空间来存放数据，同时将capacity置为4，sizie置0，没什么特别需要解释的地方。
## 🥋往堆中放入数据
下一步就是书写push的函数。插入数据没什么难的，难点在于插入这个数据后，堆是否还是个堆。例如刚开始没有数据，一个数据就是一个堆，那插入第二个数据，就要基于建大小堆考虑调整方案。
```c
void heapinit(heap* hp) {
	assert(hp); //添加适当的断言方便调试。
	hp->arr = (int*)malloc(sizeof(int) * 4);
	if (hp->arr == NULL)
		printf("malloc fail\n");
	hp->capacity = 4;
	hp->size = 0;
}
void heappush(heap* hp, int x) {
	assert(hp);
	//容量不够就要扩容
	if(hp->size==hp->capacity){
		int* new = (int*)realloc(hp->arr,sizeof(int) * hp->capacity * 2);
		if (new == NULL)
			printf("realloc fail\n");
		hp->arr = new;
		hp->capacity *= 2;
	}
	//插入数据
	hp->arr[hp->size++] = x;
	//堆的调整
	adjustup(hp->arr, hp->size - 1);
}
```
### 🤸‍♀向上调整算法
重点要讲的就是堆调整的算法---`向上调整算法`。
<img src='https://imgbed.link/file/19939'>
以上图为例，我们建立一个`大堆`，那么向上调整的意思就是从最后一个节点开始，与自己的父节点进行比较，如果比父节点大，就将值和父节点进行交换，然后`从下往上`遍历这个分支。
```c
void adjustup(int* arr, int child){
	int parent = (child - 1) / 2;
	while (child > 0) {
		if (arr[child] > arr[parent]) {
			swap(&arr[child], &arr[parent]);
			child = parent;
			parent = (child - 1) / 2;
		}
		else {
			break;
		}
	}
}
```
父节点和子节点的关系在开头就已经提出过，在此就不多赘述。 但请让在插入整个数据前，前面的数据已经是一个堆了。
<img src='https://imgbed.link/file/19940'>
以上一个图为例，因为是插入一个数据就调整一次，当插入到最后的60时，前面就已经是大堆了。所以我们只需要沿着60这个分支往上进行遍历。
### 🤸‍♂向下调整算法
如果不是单个数据的插入，而是直接给出一个数组让你将他`调整为堆`，可以使用`向下调整算法`。又或者你将堆中一个数据进行了删除，也得使用向下调整来调整整个结构。
使用这个算法有前提：
若想将其调整为`小堆`，那么根结点的左右子树必须都为`小堆`。
若想将其调整为`大堆`，那么根结点的左右子树必须都为`大堆`。
```c
void adjustdown(int* arr, int parent,int n) {
	int child = parent * 2 + 1;
	while (child < n)
	{
		// 选出左右孩子中大的那一个
		if (child + 1 < n && arr[child + 1] > arr[child])
		{
			++child;
		}

		if (arr[child] > arr[parent])
		{
			swap(&arr[child], &arr[parent]);
			parent = child;
			child = parent * 2 + 1;
		}
		else
		{
			break;
		}
	}
}
```
这里加入的参数比向上调整多了一个n，n在这里代表传入的数据长度，用来判断孩子是否越界的。

## 🥅堆删除（pop）
先给出一个判断堆是否为空的函数，方便后面进行断言

```c
bool heapempty(heap* hp)
{
	assert(hp);
	return hp->size == 0;
}
```

pop函数

```c
void heappop(heap* hp) {
	assert(hp);
	assert(!heapempty(hp));
	swap(&hp->arr[0], &hp->arr[hp->size - 1]);
	hp->size--;
	adjustdown(hp->arr, hp->size, 0);
}
```

将弹出的数据与最后一个下标交换，然后使用向下调整即可。
# 🏏topk问题
topk问题的本质就是筛选出一组数据中最大或最小的前k个。
虽然排序可以解决，但是一旦数据量大起来，比如100亿个数据，就无法全部加载到内存来进行排序，而且耗费的时间也很大，例如选`最大的k个数`，我们采取先将前k个数据建立一个`小堆`，再遍历剩下的数据，如果遇到比堆顶大的数据，就将其进行入堆并且向下调整。选取最小的k个数，则建立一个大堆。
我们以选取前k个最大值为例，进行代码书写：
1.建堆
2.遍历剩下的数据，比堆顶大则入堆替换，并向下调整。
代码如下：
```c
void printtopk(char* file, int k)
{
	assert(file);
	int* topk= (int*)malloc(sizeof(int) * k);
	
	FILE* fout = fopen(file, "r");
	if (fout == NULL)
	{
		perror("fopen error");
		return;
	}

	// 读出前k个数据建小堆
	for(int i = 0; i < k; ++i)
	{
		fscanf(fout, "%d", &topk[i]);
	}

	for (int i = (k-2)/2; i >= 0; --i)
	{
		AdjustDown(topk, k, i);
	}
}
```
> int i = (k - 2) / 2这里需要解释以下，因为k在这里代表的是数据个数，然而数组的小标从0开始，所以要额外减一个1。

当然建堆也可以使用向上调整，那么从`下标1`开始即可，因为第一个也没必要去调。
```c
for (int i = 1; i < k; ++i)
	{
		adjustup(topk, i);
	}
```
然后就只需要将剩下的数据挨个进行读取，然后与堆顶进行比较即可，比堆顶大则代替堆顶，并向下调整。

# 🏄堆排序与其时间复杂度

堆排序有了topk问题的铺垫，就很简单了，首先仍然是`建堆`。
```c
// 建堆 -- 向上调整建堆 -- O(N*logN)
	for (int i = 1; i < n; ++i)
	{
		adjustup(a, i);
	}

	// 建堆 -- 向下调整建堆 -- O(N)
	for (int i = (n - 1 - 1) / 2; i >= 0; --i)
	{
		adjustdown(a, i, n);
	}
```
那么此时出现了问题，如果我们需要的是一个`升序序列`，那应该建大堆还是小堆呢？
答案是`大堆`。如果建的是小堆，确实可以筛选出最小的那个数，就在堆顶，很方便，但是当你取走了堆顶，就会造成关系的混乱，小根堆的性质就已经变了，此时可能出现`儿子变成爹`的情况，此时就需要重新调整整个结构，得不偿失。如果建立的是大堆，每次向下调整都可以得到`剩余的数据最大值`。只需要将每次调整得到的最大值`从后往前放`即可。
这要和topk问题分清楚哦。
所以剩余的代码如下：
```c
int end = n - 1;
	while (end > 0)
	{
		swap(&a[end], &a[0]);
		adjustdown(a, 0, end);

		--end;
	}
```
<img  src='https://imgbed.link/file/20032'>

堆排序是一种非常快速的排序，其时间复杂度为O（n*logn）。
以向下调整为例，是从最后一层开始调的：
<img src='https://imgbed.link/file/20087'>
高度从1开始，假设为一个满二叉树。
建堆时，每一个数都会与下面的层数进行对比，满足条件即交换，最多的情况下就会交换`h-n`次，n为当前的层数。
可见建堆的比较次数为：
$$
t=1*(h-1)+2*(h-2)+3*(h-3)+·······+2^{h-1}*0
$$
这是一个等比等差的数列，使用高中的知识，可以将t*2，再将两式相减，即可得出：
$$
t=2^h-h-1
$$
又因为这是满二叉树，即$n=2^h-1$,所以
$$
t=n-log_2(n+1)
$$
那么就可以近似认为复杂度为O（n）了。

向上调整也是同理，从第一层开始调整：
$$
t=1*0+2*1+4*2+·····+2^{h-1}*(h-1)
$$
仍然是错位相减，可以得到：
$$
t=n*log(n-2)
$$
也就是n*logn了。
为何向上要比向下更复杂呢？因为向上用到了`最后一层`，最后一层相当于有`接近总数一半`的节点！
建堆的复杂度分析完毕，接下来就是排序的部分了。
<img src='https://imgbed.link/file/20086'>
只看最后一层，都有$2^{h-1}$个节点，最坏情况下，每个都要比较h-1次，也就是$2^{h-1}*(h-1)$
这里就很想向上调整了：
$$
t=1*0+2*1+4*2+·····+2^{h-1}*(h-1)
$$
所以得到排序的复杂度也是o（n*logn）了。