---
title: CSAPP-6-存储器的层次结构
tags: [CSAPP]
categories: [系统原理,CSAPP]
comments: true
date: 2016-11-07 20:56:22 
updated: 2016-11-07 20:56:22 
description:
---

### 前戏：关于读书顺序
能够顺着读下来当然是很好的，但是呢，一方面可能读着读着感觉无趣乏味，另一方面可能书的本身书写顺序就不是很好。所以，在前后因果关系不大的情况下，完全可以跳着读，挑感兴趣或简单的读，没问题。

### 关键点
1. 时间局部性、空间局部性
2. 缓存的大概原理
3. 机械硬盘的设计、访问开销。
4. 存储器山

### 存储技术
1. 随机访问存储器（RAM, Random-Access Memory)
> 1. SRAM，静态，双稳态
> 2. DRAM，动态，敏感，存储器系统需要周期性地读写刷新存储器的每一位。

2. DRAM阵列
> 1. DRAM组成二维阵列而不是一维线性数组的一个原因是降低芯片上地址引脚的数量。例如，16个超单元组成的阵列，二维和一维分别需要2个和4个地址引脚。二维组织的缺点是，地址必须分两步发送，增加了访问时间。（一个行地址，一个列地址）。
> 2. 再加一个存储控制器和缓存行，收到行地址，读取整行放入缓存行，之后收到列地址，返回某一个超单元的数据。
![DRAM芯片高级视图](/images/csapp-6-1.jpeg)

3. 访问主存   
还记得这张老图吗？系统总线是一组并行的导线，能携带地址、数据和控制信号。但是不同总线不能直接互通，这就用到了I/O桥。   
![一个典型系统的硬件组成](/images/csapp-6-2.jpeg)
Intel系统：
> 1. 北桥：系统总线 <=> 存储器总线，连接主存
> 2. 南桥：系统总线 <=> I/O总线，连接I/O设备

### 旋转磁盘
1. 构造
![磁盘构造](/images/csapp-6-3.jpeg)
> 1. 盘片（表面）
> 2. 磁道，表面上的一组同心圆
> 3. 扇区，每个磁道被划分为一组扇区，每个扇区包含相同量的数据位(通常是512byte)。扇区之间由一些间隙分隔开，间隙中不存储数据位，同时用来标识扇区的格式化位。
> 柱面，所有盘片表面上到主轴中心距离相等的磁道集合。（不同表面上半径相同的磁道）
> 读写头，任何时刻，不同表面的磁头都位于同一个柱面上。如图:
![磁盘的动态特性](/images/csapp-6-4.jpeg)

2.  多区记录技术。<br>原本为了保持每个磁道有固定的扇区数，越往外的磁道扇区隔得越开。多区记录技术把柱面分为多个不相交的子集合，称为记录区。每个区包含一组连续的柱面。同一个区中的磁道上扇区的数量是相同的，这个数量由该区中最里面的磁道所能包含的扇区数决定。

3. 容量计算<br>
![磁盘容量计算](/images/csapp-6-5.jpeg)

4. 访问时间（access time）<br>
对扇区的访问时间主要分为三个部分：
> 1. 寻道时间：将磁头定位到目标扇区所在的磁道。这个时间依赖于磁头之前的位置和传动臂在盘面上移动的速度。通常3~9ms。
> 2. 旋转时间：找到目标所在的第一个扇区。性能依赖于当前到达的磁道上相对目标扇区的位置和磁盘的旋转速度。
> 3. 传送时间：读写扇区内容的时间。依赖于旋转速度和当前磁道的扇区数目。

 实际中，主要是寻道时间和旋转时间。传送时间大概要小2~3个数量级。
5. 磁盘控制器<br>
磁盘控制器，维护着逻辑块号（0, 1...B-1）和实际物理磁盘扇区之间的映射关系，翻译地址。（盘面，磁道，扇区）的三元组唯一标识了对应的物理扇区。

6. 存储器映射I/O<br>
CPU使用存储器映射I/O（memory-mapped I/O）的技术来向I/O设备发出命令。在使用存储器映射I/O的系统中，地址空间有一块是为与I/O设备通信保留的。每个这样的地址称为I/O端口。当一个设备连接到总线时，它与一个或多个端口相关联。

7. 直接存储器访问技术（Direct Memory Access，DMA）<br>
由于磁盘太慢，比如旋转磁盘访问时间都在毫秒级别。比如1GHz的处理器时钟周期是1ns，1ms可以执行100万条指令。所以，在读磁盘数据的时候，如果什么都不做是一种极大的浪费。DMA技术：
> 在磁盘控制器收到来自CPU的命令后，它将逻辑块号翻译成一个扇区地址，读该扇区的内容后，将这些数据**直接传送到内存**，不需要CPU的干涉。**设备可以自己执行读或写总线事务，这个过程称为DMA。** DMA传送完成后，磁盘控制器给CPU发送一个中断信号来通知CPU。

### 局部性
1. 时间局部性：重复引用同一个变量的程序有良好的时间局部性。

2. 空间局部性：对于具有步长为k的引用模式的程序，步长越小，空间局部性越好。

3. 循环，循环本身在CPU里的运行效率是比一般的非循环指令要高的，但是由于一段程序里，循环部分占用的绝对值时间比较大，因此给人一种错觉就是，循环比较慢，这是两个不同的角度。

### 高速缓存器
1. 基本构造<br>
（S, E, B, m）,m是地址w的位长度。
> 1. S，S=2^s个组。
> 2. E，每组E个高速缓存行。
> 3. B，每个缓存行作为一个数据块，有B=2^b个字节。地址w的最后b位是块偏移。设计得真是巧啊，配合组号，正好可以把一段连续内存地存放在连续的缓存块里。
> 4. 1位有效位，指明该行是否有效。
> 5. t位标记位，t=m-b-s。唯一标识一个缓存行（数据块)

 容量计算：<img src="http://chart.googleapis.com/chart?cht=tx&chl=\Large C=S*E*B" style="border:none;"><br>
高速缓存确定一个请求是否命中，然后取出被请求的字的过程，分为三步：1）组选择，2）行匹配，3）字抽取。当且仅当设置了有效位，而且标记位与w地址中的标记位相匹配时才算命中。
![高速缓存结构](/images/csapp-6-6.jpeg)
2. 分类
> 1. 直接映射高速缓存，每个组只有一行。
> 2. 组相联高速缓存，每个组不止一行。
> 3. 全相联高速缓存，只有一个组，所有的缓存行都在一个组里。

3. 高速缓存的写
 - 缓存命中
> 1. 直写。立即将w的高速缓存块写到低一层的存储器中。缺点是每次写都会引起总线流量。通常是非写分配的形式。
> 2. 写回。尽可能的推迟存储器更新，只有当替换算法要驱逐已经更新过的块时，才把它写到低一层的存储器里。优点是，写回能显著地减少总线流量。缺点是，它增加了缓存的复杂性，每个缓存行必须维护一个额外的修改位（dirty bit），标明该行是否被修改过。通常是写分配的形式。

 - 不命中
 > 1. 写分配，加载低一层的块到高速缓存中，然后更新这个高速缓存块。写分配视图利用写操作的空间局部性（下一次写操作可能是下一个地址的内容）。缺点是：每次写不命中都会导致一个块从低一层传送到高速缓存。
 > 2. 非写分配，避开高速缓存，直接把这个字写到低一层中。

 其实就两种情况，写到低一层存储器的时候，是否要经过高速缓存。

4. 计算缓存的不命中率<br>
 - 最根本的原理是根据缓存的参数，计算出来地址里各个部分分别占几位，然后就每次访问的地址进行计算，观察是否在缓存中。

 - 简单点的，S、E、B其实是特别巧的，低b位就是块偏移，然后是s位的组号。同一组的数据都有相同的组号。算下来，内存中的每一块会按顺序正好填充到缓存中。所以只需要看缓存是否能够存下这整块数据，若能，就只有第一次访问的时候会存在冷不命中，此时，不命中率=组数/访问数。若不能，则看情况了，尤其是访问步长不为1的时候。

5. 存储器山 的概念
读速度是时间和空间局部性的二维函数。时间局部性的表现一是可以通过程序优化，二是可以加大内存，目的是使得一段程序的运行数据全部都在缓存里命中。