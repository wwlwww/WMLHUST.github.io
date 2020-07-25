---
title: CSAPP-8.3-异常控制流3
tags: [CSAPP]
categories: [系统原理,CSAPP]
comments: true
date:  2016-11-28 10:57:58
updated:  2016-11-28 10:57:58
description:
---
## 信号
### 简介
![这里写图片描述](/images/csapp-8.3-1.jpeg)

每种信号类型都对应某种系统事件。底层的硬件异常是由内核异常处理程序处理的，正常情况下，堆用户进程而言是不可见的。信号提供了一种机制，统治用户进程发生了这些异常。当一个子进程终止或者停止时，内核会发送一个SIGCHLD信号给父进程。

### 发送信号
1. 内核更新目的进程上下文中的某个状态，来给目的进程发送信号。一个进程可以给它自己发信号。向进程发信号都是基于进程组（process group）的概念。

3. 进程组。每个进程都只属于一个进程组，进程组由一个正整数**进程组ID, pgid**标识。shell也为每个作业（job）都创建一个独立的进程组，pgid是新进程的pid。如果这个新进程再继续创建子进程，默认地，一个子进程和它的父进程同属于一个进程组。一个进程可以通过函数改变自己活其他进程的进程组。（权限？）
 ```c
 #include <unistd.h>

 pid_t getpgrp(void);   //返回调用进程的进程组ID

 int setpgid(pid_t pid, pit_t pgid);    //设置一个进程的进程组id，成功返回0，失败-1.
 ```
4. kill命令可以给其他进程发送任意的信号。

 ```shell
 kill -9 1234          //给进程1234发送信号9（SIGKILL）
 kill -9 -1234         //负的pid，表示给1234这个进程组，的每个进程发信号9
 kill -9 0             //0，给当前进程所在进程组，的每个进程发信号9
 ```
 ```c
 #include <sys/types.h>
 #include <signal.h>

 /* 进程可以调用kill函数发送信号，包括给他们自己 */
 int kill(pid_t pid, int sig);
 ```
 ```c
 #include <unistd.h>

 /* 调用alarm函数可以给它自己发送SIGALRM信号 */
 unsigned int alarm(unsigned int sig);
 ```

### 接收信号
1. 接收信号。内核为每个进程在pending位向量维护着待处理信号的集合，而在blocked位向量中维护着被阻塞的信号集合。所以在任何时刻一种类型的信号只会被接收一次，在处理它的时候，会先把该类型的信号block。进程可以忽略信号，也可以捕获这个信号，执行信号处理程序。

2. 当内核从一个异常处理程序返回（进程调度也属于一种异常？定时器中断？），准备把控制传递给某个进程p时，它会检查进程p的未被阻塞的待处理信号集合（pending & ~blocked）。如果这个集合不为空，那么内核选择集合中的某个信号k（通常是编号最小的信号，所以Linux信号编号还是特意的呢，编号越小，优先级越高），并进入k的处理程序。

 Ps:在block的时候，来的信号会不会标记到pending里？

 答案：会的。执行信号的处理动作称为信号递达（Delivery），信号从产生到递达之间的状态，称为信号未决（Pending）。进程可以选择阻塞（Block）某个信号。**被阻塞的信号在产生时将保持在未决状态**，直到进程解除对此信号的阻塞，才执行递达的动作。注意，阻塞和忽略是不同的，只要信号被阻塞就不会递达，而忽略是在递达之后可选的一种处理动作。忽略也可以说是信号处理程序的一种。

3. 信号的默认处理程序是如下中的一种：
 - 进程终止

 - 进程终止并转储存储器（dump core）

 - 进程暂停，直到被SIGCONT信号重启

 - 进程忽略该信号

4. 修改信号的默认处理程序（忽略，恢复默认行为，自定义）

 ```c
 #include <signal.h>

 typedef void (*sighandler_t)(int);

 sighandler_t signal(int signum, sighandler_t handler);
 ```

5. 待处理信号被阻塞。Unix信号处理程序通常会阻塞当前处理程序正在处理的类型的信号。比如正在执行SIGINT处理程序时，如果再来一个SIGINT信号，只会在pending里置1（感觉此时应该已经为1了）。

6. 系统调用可以被中断。像read、wait和accept这样的系统调用会潜在地阻塞进程一段时间，称为慢速系统调用。在某些操作系统中，若正在执行慢速系统调用的时候收到了信号，当从信号处理程序返回时，原来的慢速系统调用可能不会继续，然而由于被信号中断，系统调用的任务并未完成。Linux系统会自动重启被中断的系统调用。

7. 阻塞和取消阻塞信号

 ```c
 #include <signal.h>

 /* 改变blocked向量的值，若oldset!=null，会用来保存以前blocked向量的值 */
 int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);

 /* 初始化set为空集 */
 int sigemptyset(sigset_t *set);

 /* 初始化set全为1，每个信号都填入blocked向量 */
 int sigfillset(sigset *set);

 /* 添加、删除signum到set */
 int sigaddset(sigset_t *set, int signum);
 int sigdelset(sigset_t *set, int signum);

 /* set中对应signum是否置1 */
 int sigismember(const sigset_t *set, int signum);
 ```

## 非本地跳转
C语言提供了一种用户级异常控制流形式，称为非本地跳转（nonlocal jump)。它将控制直接从一个函数转移到另一个当前正在执行的函数，而不需要经过正常的调用-返回序列。涉及到的两个函数分别是setjmp和longjmp。
```c
#include <setjmp.h>

int setjmp(jmp_buf env);
int sigsetjmp(sigjmp_buf env, int savesigs);

void longjmp(jmp_buf env, int retval);
void siglongjmp(sigjmp_buf env, int retval);
```
1. setjmp函数在env缓冲区中保存当前的调用环境，以供后面longjmp使用，并返回0，调用环境包括程序计数器、栈指针和通用目的寄存器。

2. longjmp函数从env缓冲区中恢复调用环境，然后从最近一次setjmp的env中恢复，恢复的时候相当于setjmp函数的返回，只是返回值非0（longjmp的第二个参数）。longjmp函数只有调用，没有返回。此外，setjmp函数调用一次，可以返回多次。

3. 非本地跳转的一个重要应用是允许从一个深层嵌套的函数调用中立即返回。如果再一个深层嵌套的函数中发现了错误，可以调用longjmp(env, ret)，直接返回到setjmp处，然后根据返回的ret值判断是什么错误，调用什么样的错误处理程序。类似java的try...catch语句。

## 操作进程的工具
1. strace: 打印一个正在运行的程序和它的子进程调用的每个系统调用的轨迹。

2. pmap: 显示进程的存储器映射

3. /proc：一个虚拟文件系统，以ASCII文本格式输出大量内核数据结构的内容，用户程序可以读取这些内容。比如 cat /proc/loadavg，观察linux系统上当前的平均负载。