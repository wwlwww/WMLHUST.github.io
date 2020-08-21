---
title: 从CAS到无锁队列.md
tags: [算法思想]
categories: [后端,算法]
comments: true
date: 2020-06-03 17:24:15
updated: 2020-06-03 17:24:15
description: 从无锁思想到CAS，再到具体实现，以无锁队列为例。
---

## 无锁算法
多个线程读写同一内存，如何做到不加锁呢？其实没有那么高大上的算法在里面，实现无锁的前提是，硬件需支持”读取-更新-写入“的原子操作，比如 Test and Set, Fetch and Add, Compare and Swap等。    
以Compare and Swap，也就是CAS为例，可以实现很多无锁的数据结构，无锁队列，无锁树，区别在于需要几次的CAS。

## CAS
`bool CAS(type* addr, type val_old, type val_new)`   
如果 addr 的值等于 val_old，就把它设置为 val_new，设置成功返回true，失败返回false。**这个比较并赋值的操作，是一个原子操作。**

## 无锁队列
我们使用一个单向链表，作为无锁队列的基础数据结构。利用CAS的原子性，来保证在push/pop，也就是在链表尾/头添加、删除节点时，不会出现多线程互相覆盖的问题。

直接看代码:

```C
// 很久没写C/CPP，语法细节忘了不少，忽略忽略

struct {
    node* tail  // 尾指针
    node* head  // 头指针
}* Q
```
```C
// Q是队列，data是待push的节点
Push(Q, data)
{
    while true 
    {
        p = Q->tail;
        if CAS(p->next, NULL, data) {
            // 如果此时p还是Q的tail，才能设置成功
            break
        }
    }
    // 更新Q的tail，如果此时tail还是p，才能设置成功。
    // 此时不用担心失败，因为如果此处不更新tail，其他线程拿到的总是旧的tail，
    // 其他线程在while循环中的CAS，会发现p->next!=NULL，就会失败, 一直处于while循环中
    CAS(Q->tail, p, data)
}
```
```C
Pop(Q)
{
    while true 
    {
        p = Q->head
        if CAS(Q->head, p, p->next) 
        {
            break
        }
    }
    return p->value
}
```

由于CAS会直接用新值覆盖旧值，为了保存旧值，所以每次都会先把旧值取出来。然后在设新值时，要判断旧值是否发生了变化。   
那么以上实现有什么问题没？
1. 问题1，死循环   
考虑一些意外的情况。对于Push，如果线程第一个CAS执行成功，在执行第二个CAS时宕掉。此时 tail 未更新，其他线程会发现tail.next总是不为空，因此就会陷入while死循环。

2. 问题2，**ABA问题**   
比如，一个线程按序执行了 pop -> push操作，而push的节点，恰巧复用了被pop节点同一块内存。因为此链表例子中，CAS比较的是内存地址，所以校验通过。而里面的值其实是发生了变化的，如果不校验里面的值，可能会认为节点未被改动。

这两个问题如何解决呢？   
1. 死循环问题
- 关键：tail节点未更新，导致CAS(p->next, NULL, data) 总是失败，因此可以让每个线程发现这个问题后，自己去更新tail节点。
   
2. ABA问题   
- 节点增加计数器，每一次更新。计数的增减操作也需要原子化。

## 总结
无锁数据结构的大致思想就是这样。   
借助CAS，一个极端的想法，所有程序都可以做成无锁的。只需要对任何一个变量的读写，都使用CAS操作，失败则从头开始。此时，虽然实现了无锁，但是效率却是降低的，因此，无锁也有它的适用场景 --- **读多写少**，因为此时CAS的冲突率比较小。   
与CAS比较像的一个机制，是自旋锁。自旋锁总是在尝试**加锁**，而CAS总是在尝试**比较-修改**，都算是**忙等**机制。

