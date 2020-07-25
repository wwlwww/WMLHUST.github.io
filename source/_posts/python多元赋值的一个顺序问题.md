---
title: python多元赋值的一个顺序问题
tags: [python]
categories: []
comments: true
date: 2020-07-12 18:50:22
updated: 2020-07-12 18:50:22
description:
---

起因是试验一个python翻转链表的代码，python的变量交换非常方便，比如交换两个元素，可以直接`a, b = b, a`，因此翻转链表，可以这么写：
```python
def reverseList(head: ListNode) :
    pre = None
    cur = head
    while cur is not None:
        cur, pre, cur.next = cur.next, cur, pre

    return pre
```
然而，这个代码是错误的。

经过单步debug，发现在第一次交换发生时，交换的结果就不对。   
不用怀疑，如果是 `a, b, c = b, c, a`，那肯定是对的。所以问题出在哪里？不会是python的一个bug吧？   
追根溯源，直接看下字节码：
```python
import dis

# 初始化链表
n0 = ListNode(0)
n1 = ListNode(1)
n2 = ListNode(2)
n3 = ListNode(3)
n0.next = n1
n1.next = n2
n2.next = n3

# 查看交换代码的字节码
pre = None
cur = n0
dis.dis("cur, pre, cur.next = cur.next, cur, pre")
```
输出是   
```python
# 0-10步，先依次取右值 cur.next, cur, pre，然后倒序
 0 LOAD_NAME     0 (cur)   # 取cur值，push入栈
 2 LOAD_ATTR     1 (next)  # 取cur.next，替换栈顶
 4 LOAD_NAME     0 (cur)   # 取cur值，push入栈
 6 LOAD_NAME     2 (pre)   # 取pre值，push入栈
 8 ROT_THREE
10 ROT_TWO                 # ROT_3和ROT_2两步的效果，就是对栈内的3个元素倒序
                           # 目前栈内从顶至底分别为[old_cur_next, old_cur, old_pre]

# 以上步骤没有问题，提前准备好要赋值的数据。
# 接下来，依次赋值左边变量，cur, pre, cur.next
12 STORE_NAME    0 (cur)   # pop栈顶，赋值给 cur (new_cur = old_cur_next)
                           # 此时栈 [old_cur, old_pre]

14 STORE_NAME    2 (pre)   # pop栈顶，赋值给 pre (new_pre = old_cur)
                           # 此时栈 [old_pre]

16 LOAD_NAME     0 (cur)   # 取cur值，push入栈
                           # 此时栈 [new_cur, old_pre]

18 STORE_ATTR    1 (next)  # 将栈顶元素(也就是cur)的next属性，赋值为栈的第二个元素
                           # new_cur.next = old_pre
```
关键在于第16步，重新load了一下cur，而此时cur已经在第12步，被赋值为old_cur_next。实际执行的是`new_cur.next = old_pre`，而我们的目标是`old_cur.next = old_pre`。因此出现了不一致。

关键在于 **多元赋值中，左边被赋值的变量是有先后关系的。先改变了cur，那么再给cur.next赋值时，cur就已经是新的cur。因此如果同时需要改变cur、cur.next的值，应该优先赋值cur.next。**

修改后的代码：
```python
def reverseList(head: ListNode) :
    pre = None
    cur = head
    while cur is not None:
        # cur, pre, cur.next = cur.next, cur, pre
        cur.next, pre, cur  = pre, cur, cur.next
    
    return pre
```