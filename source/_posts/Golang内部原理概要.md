---
title: Golang部分原理概要
tags: [golang]
categories: [后端]
comments: true
date: 2020-08-18 15:33:30
updated: 2020-08-18 15:33:30
description:
---
### 1. 协程实现与调度
协程，可以当做线程的一个个任务，不过线程会对协程进行调度，类似操作系统对线程做的调度。协程对比线程的优势在于：
- 协程切换完全在用户空间进行，线程切换需要在内核空间完成
- 协程切换相比线程，需要切换的东西比较少，协程切换主要包括cpu寄存器；而线程出了cpu寄存器，还有线程私有的栈和寄存器等等。
   
Golang里的GPM协程调度模型非常著名，简单地如下图所示：
![GPM调度模型](/images/golang_mpg.png)  
P充当了物理线程M和协程G之间的协调者，也可以说是对物理线程资源的一个封装，形成逻辑线程。
每个P关联一个等待被调度运行的G队列，P会依次执行。然而正常情况下，协程之间并不会出现抢占，而是依次排队执行。发生调度的时机如下所示：
- 主动挂起
- 系统调用发生阻塞
- 协作式调度：在发生方法调用时，会检查标志
- 系统监控

这里还有很多细节，比如如果某个P上面的G队列空了，然而其他P上的G都还是满的，这种时候怎么办？GoMaxProcs设置为CPU核数一定性能最好吗？  

### 2. GC
Golang的另一个优点在于，不用手动管理内存，比较像Java，Java运行时需要JVM，Golang不需要单独的VM，而是在编译时，内嵌一个小的Runtime，来处理GC、调度等事务。

整个GC大致分为4个阶段：  
> 1. Mark Prepare，**需要STW**，标记阶段准备工作，如标记根对象、开启写屏障和辅助GC等
> 
> 2. Marking, 并发标记，**不需STW**，采用**三色标记法**，多个GC协程和多个用户协程可以并行运行。初始大概25%的P用于mark，逐个包括全局指针和G栈上的指针（扫描栈时，需要停止该G）。这部分工作叫gcBgMarkWorker。  
> 如果保证marking过程中，其它G分配堆内存太快，导致mark跟不上allocate的速度，其它G还会被征用，配合做一部分mark的工作，称为mutator assists。  
> 在marking期间，每次G分配内存，就会更新自己的gcAssistBytes，这个和G要帮忙mark的内存大小成正比。也就是mutator assist的工作量。
>
> 3. Mark Termination，标记结束，**需要STW**，关闭写屏障，重新扫描发生变化的对象，防止误回收。
> 4. Sweeping，并发清理，**不需STW**。清理回收的开销，被平摊到每次G的内存分配操作中，直到所有sweep任务完成。所以在go trace里看到sweep是和G并行运行的。另外go trace里的GC时间段，就不包括这部分时间。

随着Golang对GC的不断优化，当前的STW时间是非常短的，一般都在2ms以内。

非常推荐使用GoTrace观察一下实际运行中，Golang的协程调度表现，能够更好地理解调度与GC等过程。也可以打开gctrace开关，观察运行过程中的gc行为。

### 3. defer原理
每个goroutine会保存一个defer调用链，在方法执行结束时，取出链头，判断是否当前方法内设置的defer，是就执行，否则终止执行。

可以从编译期和运行期两个角度理解：  
- 编译期：defer在会被转换为deferproc()方法，并在return前插入deferreturn()方法。
- 运行期：  
  deferproc()方法即会在goroutine的defer调用链头插入一个_defer结构体，代表当前被defer的方法。  
  deferreturn()会依次读取、执行goroutine的defer调用链。

**注意Golang方法调用使用值传递，因而被defer方法的参数在defer()时已经确定。**

### 4. panic与recover原理
同defer，每个goroutine维护一个自己的panic链表，发生panci后，会在panic链表插入一项。而recover则从panic链表依次读取处理。

同样可以从编译期和运行期理解panic和recover。
- 编译期：panic()和recover()，会分别转换为gopanic()和gorecover()。  
- 运行期：  
  gopanic()，说明发生了panic，会插入panic到当前goroutine的panic链，然后循环执行defer链，此时即使非本方法的defer方法，也会被执行。如果执行完defer方法后，未recover，最终将终止整个进程。  
  gorecover()，一般recover都是放在defer里（放在defer外也不会生效，因为如果发生了panic，只有defer代码会被执行）。gorecover()从panic链获取待处理的panic，处理完后return会正常流程。

因此，总结一下，panic如果不recover，会导致整个进程退出，而recover只能捕获自己goroutine的panic。所以，**一个稳定的服务，要保证每个goroutine，都有defer recover()兜底。** 当然，在recover中处理panic时，也要有响应的告警通知机制，以便开发及时发现处理。

### 5. interface原理
良好的interface设计，能够减少模块之间的耦合，隔离具体实现细节。  
interface分为`iface`和`eface`，其中`eface`指不含method的interface。而face内部都有类似_type和_data两个指针，分别指向类型信息和具体对象信息。对于含有method的struct对象，_data里也记录了interface的方法列表。  
方法之间传递时，struct与interface转换的合法性检测，发生在编译期。

golang内部，如果interface == nil，要求_type和_data同时为nil。

### 6. channel原理
golang的一个设计哲学是，不要通过共享内存来通信，而应该通过通信来共享数据。而goroutine之间就是使用channel进行高效的协程间通信。  
由channel的FIFO特性，很容易想到队列，其实channel本身就是个队列。  
channel内部实现的结构体如下：
```golang 
// runtime/chan.go
type hchan struct {
	qcount   uint           // 环形缓冲区中的元素个数
	dataqsiz uint           // 环形缓冲区的容量
    buf      unsafe.Pointer // 环形缓冲区的指针
	elemsize uint16         // 元素大小
	closed   uint32         // chan是否关闭
	elemtype *_type // 元素类型
	sendx    uint   // 环形缓冲区的写入index
	recvx    uint   // 环形缓冲区的读取index
	recvq    waitq  // 由于channel是空，读取阻塞的G队列
	sendq    waitq  // 由于channel已满，写入阻塞的G队列

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex  // 互斥锁
}
```
简洁明了，channel内部是使用一个环形缓冲区和一个互斥锁实现其队列机制，在此基础上做了一些优化，如：
- 如果chan是空的，那么读取的G就会阻塞，那么下次写的G过来，可以不经过环形缓冲区，直接交给读取阻塞的G，从而提升效率
- 写chan阻塞同理  

### 7. 定时器   
**千万别以为定时器是精确触发的！！！** 如果定期间隔 <10ms，很多定时器的精确度都不够，包括golang的实现。  
golang使用四叉堆来维护众多的timer。对定时器的检查有两个时机：
- G调度时
- 监控线程执行时  

检查时，只需检查堆顶的timer（因为堆顶的timer是最先到期的），如果到触发时间，则起一个goroutine，执行对应回调函数。而goroutine的调度时间，也是不可控的，因此不能精确依赖定时器的触发时间。

还有一些其他的定时器实现方式，比如nginx使用红黑树、linux内核使用时间轮等。   

### 8. 内存分配
内存分配
先说一下给对象 object 分配内存的主要流程：

1. object size > 32K，则使用 mheap 直接分配。
3. object size > 16 byte && size <=32K byte 时，先使用 mcache 中对应的 size class 分配。
4. object size < 16 byte，使用 mcache 的小对象分配器 tiny 直接分配。（其实 tiny 就是一个指针，暂且这么说吧。）
5. 如果 mcache 对应的 size class 的 span 已经没有可用的块，则向 mcentral 请求。
6. 如果 mcentral 也没有可用的块，则向 mheap 申请，并切分。
7. 如果 mheap 也没有合适的 span，则想操作系统申请。

### 9. 相关资料
- 也谈goroutine调度器：https://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/
- Go 语言设计与实现：https://draveness.me/golang/ ，直接解读runtime源码
- 关于Golang GC的一些误解：https://zhuanlan.zhihu.com/p/77943973
- GC In Go: https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html ，系列的三篇文章都值得一读










