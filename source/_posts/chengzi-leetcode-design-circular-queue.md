---
title: 【leetcode】设计循环队列
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/1610173947-QjrHlC-LeetCode.png
abbrlink: a256b950
tags:
  - 刷题
  - c语言
date: 2023-03-20 16:02:18
categories: 
- 刷题
ai: ture
---

> 来源：[622.设计循环队列](https://leetcode.cn/problems/design-circular-queue/)

设计你的循环队列实现。 循环队列是一种线性数据结构，其操作表现基于 FIFO（先进先出）原则并且队尾被连接在队首之后以形成一个循环。它也被称为环形缓冲器。

循环队列的一个好处是我们可以利用这个队列之前用过的空间。在一个普通队列里，一旦一个队列满了，我们就不能插入下一个元素，即使在队列前面仍有空间。但是使用循环队列，我们能使用这些空间去存储新的值。

你的实现应该支持如下操作：

MyCircularQueue(k): 构造器，设置队列长度为 k 。
Front: 从队首获取元素。如果队列为空，返回 -1 。
Rear: 获取队尾元素。如果队列为空，返回 -1 。
enQueue(value): 向循环队列插入一个元素。如果成功插入则返回真。
deQueue(): 从循环队列中删除一个元素。如果成功删除则返回真。
isEmpty(): 检查循环队列是否为空。
isFull(): 检查循环队列是否已满。
示例：
MyCircularQueue circularQueue = new MyCircularQueue(3); // 设置长度为 3
circularQueue.enQueue(1);  // 返回 true
circularQueue.enQueue(2);  // 返回 true
circularQueue.enQueue(3);  // 返回 true
circularQueue.enQueue(4);  // 返回 false，队列已满
circularQueue.Rear();  // 返回 3
circularQueue.isFull();  // 返回 true
circularQueue.deQueue();  // 返回 true
circularQueue.enQueue(4);  // 返回 true
circularQueue.Rear();  // 返回 4
提示：
所有的值都在 0 至 1000 的范围内；
操作数将在 1 至 1000 的范围内；
请不要使用内置的队列库。
# 🍏思路
本题是想实现一个类似于下图结构的队列，本文采取`数组`的形式实现，但是难点就在于判空和判满。
<img src='https://imgbed.link/file/19458'>
> rear指针用于当填入数据时，rear+1.  
front指针用于删除数据时，front+1.
 
 当队空的时候，我们很容易知道front指针等于rear指针时，整个数组便可以判空，那如果一直往数组里塞入数据，导致rear指针不断的后移，最终和front指针相等时，整个队列也满了，那么此时队空和队满就是一个相同的条件了，无法做出判断。

 所以我们采取额外开辟一个空位的方法，此空位`不存储任何数据`，仅作为判满的辅助条件。
 当rear+1到达此空位时，（rear+1）%capacity ==front  ，便可作为判满条件。
 为何要取余呢，因为rear在不断的向前推进，会不断加1，而我们设计循环队列的思路就如上面展示的图一样，其`逻辑`路径是一条`环路`，所以通过取余可以让rear指针回到原本开始时的位置。

# 🍎定义结构
```c
typedef struct {
  int* arr;
  int front;
  int rear;
  int capacity;
} MyCircularQueue;

```
按照上面的说法，我们定义了如上的队列结构，其包含两个指针，分别指向队头和队尾，以arr为一个地址基址，方便后续的内存开辟。capacity则是数组的容量大小。

# 🍐初始化队列
```c
MyCircularQueue* myCircularQueueCreate(int k) {
        MyCircularQueue* q=(MyCircularQueue*)malloc(sizeof(MyCircularQueue));
        q->arr=(int*)malloc(sizeof(int)*(k+1));
        q->capacity=k+1;
        q->front=q->rear=0;
    return q;
}
```
传入的k代表我们要输入的`数组元素个数`，依照上面的思路我们要额外开辟一个空间方便判断队满。

# 🍊入队和出队
入队：
```c
bool myCircularQueueEnQueue(MyCircularQueue* obj, int value) {
        if((obj->rear+1)%(obj->capacity)==obj->front){
            return false;
        }
        obj->arr[obj->rear]=value;
        obj->rear=(obj->rear+1)%obj->capacity;
        return true;
}
```
入队就需要判断是否队满，所以就要用到  `(obj->rear+1)%(obj->capacity)==obj->front`这个条件。
同时需要注意插入数据时rear+1的情况，如下图的情况：
<img src='https://imgbed.link/file/19459'>
rear如果不取余操作，就会超出设置的k，那么整个回路就会出现问题了。
出队：
```c
bool myCircularQueueDeQueue(MyCircularQueue* obj) {
        if(obj->rear==obj->front){
            return false;
        }
        obj->front=(obj->front+1)%obj->capacity;
        return true;
}
```
判断队是否为空就直接判断obj->rear==obj->front即可。
front+1同上，要进行取余操作。

# 🍌取队头队尾
```c
int myCircularQueueFront(MyCircularQueue* obj) {
   if(obj->rear==obj->front){
            return -1;
        }
        return obj->arr[obj->front];
}

int myCircularQueueRear(MyCircularQueue* obj) {
    if(obj->rear==obj->front){
            return -1;
        }
        return obj->arr[(obj->rear-1+obj->capacity)%obj->capacity];
}
```
要取值就要考虑是否为空，所以直接使用obj->rear==obj->front进行判断即可。
重点在于第二个函数。
如下面的情况：
<img src='https://imgbed.link/file/19460'>
rear指向了第四个位置，但这个位置是不需要放值的，如果return obj->arr[obj->rear]，就会出现问题。所以采用取余操作。

# 🍉判空判慢与销毁
这个就没啥好说的了。
```c
bool myCircularQueueIsEmpty(MyCircularQueue* obj) {
  return obj->front==obj->rear;
}

bool myCircularQueueIsFull(MyCircularQueue* obj) {
 return (obj->rear+1)%(obj->capacity)==obj->front;
}

void myCircularQueueFree(MyCircularQueue* obj) {
       free(obj->arr);
       free(obj);
}
```




