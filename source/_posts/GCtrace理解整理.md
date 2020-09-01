---
title: GCtrace理解整理
tags: [golang]
categories: [后端]
comments: true
date: 2020-08-25 11:37:28
updated: 2020-08-25 11:37:28
description:
---
之前定位问题时，找了很多官方、网上的一些资料，现在整理出来：

export GODEBUG=gctrace=1，可以打开gc trace开关，具体信息会输出到stderr。
这是一个gc trace的一条具体信息。
```
gc 1405 @6.068s 11%: 0.058+1.2+0.083 ms clock, 0.70+2.5/1.5/0+0.99 ms cpu, 7->11->6 MB, 10 MB goal, 12 P
```
可以分为几部分：
1. 整体指标  gc 1405 @6.068s 11%  
- gc 1405，表示第1405次GC  
- @6.068s, 表示程序启动至今，执行了6.068s  
- 11%，本次GC总共占用了11%的CPU  

2. 墙上时间  0.058+1.2+0.083 ms clock  
- 0.058，Mark Prepare阶段的时间，STW状态  
- 1.2，Marking阶段的时间，就是并发mark阶段  
- 0.083，Mark Termination阶段耗时，STW状态  
需要了解[Golang GC的过程](https://acac.fun/2020/08/18/Golang%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86%E6%A6%82%E8%A6%81/)

3. CPU时间 0.70+2.5/1.5/0+0.99 ms cpu  
- 0.70，Mark Prepare阶段STW的CPU时间  
- 2.5，Marking阶段，mutator assist的CPU时间  
- 1.5，Marking阶段，background gc time，包括 delicated 和 fractional   
- 0，markding阶段，idle gc time  
- 0.99，Mark Termination阶段CPU时间  

4. 内存  7->11->6 MB, 10 MB goal
- 7，mark之前的堆大小
- 11，mark之后的堆大小，意思是在marking过程中，又有4M的堆分配
- 6，mark结束后，被标记为存活的堆内存，意味着下次GC的目标是12Mb
- 10MB goal，下次堆内存达到10MB时，执行GC

5. 12 P
- P的数量，也就是GoMaxProcs大小

6. 其他，GC forced
- 如果两分钟没有GC，就会强制执行一次GC，此时打印GC forced