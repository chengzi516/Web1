---
title: 【leetcode】双指针巧解判断链表是否有环的问题
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/1610173947-QjrHlC-LeetCode.png
tags:
  - 刷题
  - c语言
abbrlink: 17edd14a
categories: 
- 力扣
swiper_index: 7
date: 2023-03-09 19:29:52
---
> 本章讲述的例题来自leetcode [141.环形链表1](https://leetcode.cn/problems/linked-list-cycle/description/),[142.环形链表2](https://leetcode.cn/problems/linked-list-cycle-ii/description/)，旨在利用双指针解决此类型的问题。


# 💻141.环形链表1
给定一个链表的头节点  head ，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。

不允许修改 链表。

 

示例 1：
<img src='https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png'>


输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
示例 2：
<img src='https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png'>


输入：head = [1,2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。
示例 3：

<img src='https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png'>

输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。


解法：本题可以思考转换为`追击问题`，设定两个指针，一个走得快，一个走得慢，当链表存在`闭环`时，快指针就会在环中不断的循环，而慢指针此时就有机会追上快指针。当二者相遇，就证明此链表为环形的链表。
那么问题又来了，快慢指针该如何设定走的速度呢？如果快指针每次走2，慢指针走1，他们是否会在环中相遇呢？将快指针的速度调到一次走3，一次走4呢？
在这里我们可以尝试以归纳的思想证明以下。
先假定两指针的速度分别为2，1。
若链表`存在环`，那么fast必定会先进入环中，当slow开始入环，设定其二指针相隔距离为`N`,此时问题就已经演变为了fast指针去追击slow指针，且二者相隔N。
fast一次走两步，slow一次走一步，也就意味着`N会不断的减1`，那么`N一定会减小到0`，当N为0时，快指针也就追上了慢指针。
若fast一次走`三步`呢？
N,N-2,N-4,N-6··············，如果N是一个`偶数`，那么N也是可以为0的，意味着相遇，那如果N是`奇数`呢？必定会得到-1，也就是`fast走快了一步`，走到了slow的前面去了。此时fast又得开始一段新的追击,假设环的长度为C，那么`新的N就等于C-1`,此时能否追上就要考虑C-1是偶数还是奇数了。

参考代码如下：
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
bool hasCycle(struct ListNode *head) {
    struct ListNode* slow =head, *fast=head;
    while(fast && fast->next)
    {
        slow=slow->next;
        fast=fast->next->next;
        if(slow==fast)
        {
            return true;
        }
        
    }
    return false;
}
```

# 🎥142.环形链表2
给定一个链表的头节点  head ，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。

不允许修改 链表。

 

示例 1：

<img src='https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png'>

输入：head = [3,2,0,-4], pos = 1
输出：返回索引为 1 的链表节点
解释：链表中有一个环，其尾部连接到第二个节点。
示例 2：
<img src='https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png'>


输入：head = [1,2], pos = 0
输出：返回索引为 0 的链表节点
解释：链表中有一个环，其尾部连接到第一个节点。
示例 3：

<img src='https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png'>

输入：head = [1], pos = -1
输出：返回 null
解释：链表中没有环。

思路：此题算是上一道题的一个延深，直接上结论吧：
> 让一个指针从`起始`位置开始遍历链表，同时让一个指针从`判环时相遇点`的位置开始绕环运行，两个指针每次都是`只走一步`，最终都会在`入口`处的位置相遇。

这个结论也就意味着，只需要在上一道题所求出的那个判环相遇点开始与起点指针同步运行，二者一定会在环的入口处相遇。

  *如何证明?*
因为上题我们已经证明了fast走两步，slow走一步，就一定可以相遇的结论，那么我们本题也设定在相同的条件下进行一个假设归纳。
<img src='https://imgbed.link/file/18235'>
设到入口处前的长度为l，环长c，相遇位置和入环口间距x。
那么slow指针走了多少步呢？l+x还是l+x+n*c呢？
答案是`l+x`，因为我们已经证得了快慢指针一定会相遇，所以慢指针不可能在环中走`超过一圈`。
fast指针呢？它走了l+x+c还是l+x+n*c呢？
有的人可能看到上面给出的那幅图，就会觉得fast在环中走了一圈，然后与slow相遇。
那如果是下面这张图呢？
<img src='https://imgbed.link/file/18236'>
fast就走了不止一圈了。所以正确的答案就是`l+x+n*c`。
但是不管是错误的推断成l+x+c还是l+x+n*c，得到的结果都是`一致`的。
因为fast走的路程是slow的两倍，所以可以联立出以下的式子：
$l+n*c+x=2*(l+x)$ ，解得： $l=n*c-x$
可能初看不是很理解，可以将n带入成1，2，3，比如n为1时，l=c-x，那么此时两个指针分别从链表头和相遇点开始移动，就一定能在环的入口处相遇。n为2，l=2*c-x，从相遇点开始移动的指针绕着环转了2圈-x的路程，也恰好到达入环处。
所以代码如下：
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode *detectCycle(struct ListNode *head) {
    struct ListNode* fast=head;
    struct ListNode* slow=head;
    while(fast&&fast->next){
        fast=fast->next->next;
        slow=slow->next;
        if(fast==slow){
            struct ListNode* begin=head;
            struct ListNode* meet=slow;
            while(begin!=meet){
                begin=begin->next;
                meet=meet->next;
            }
            return meet;
        }

    }
    return NULL;
}
```

