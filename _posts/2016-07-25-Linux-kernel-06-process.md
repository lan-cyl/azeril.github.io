---
layout: post
title: Linux内核分析（六）进程剖析
categories: [linux kernel ]
tags: [linux kernel, ]
description: linux kernel细说进程
---

> 触发任何一个事件时，系统都会将他定义为一个程序，并且给予这个程序一个ID，成为PID，同时依据启发这个程序的使用者与相关属性关系，给予这个PID一组有效的权限设定

## 进程的创建

在Linux系统中，所有的进程都是PID为1的init进程的后代。内核在系统启动的最后阶段启动init进程。该进程读取系统的初始化脚本（initscript）并执行其他的相关程序，最终完成系统启动的整个进程。

Linux提供两个函数去处理进程的创建和执行：fork()和exec()。首先，fork()通过拷贝当前进程创建一个子进程。子进程与父进程的区别仅仅在于PID（每个进程唯一），PPID（父进程的PID）和某些资源和统计量（例如挂起的信号）。exec()函数负责读取可执行文件并将其载入地址空间开始运行。

fork()使用写时拷贝（copy-on-write）页实现。内核在fork进程时不复制整个进程地址空间，让父进程和子进程共享同一个拷贝，当需要写入时，数据才会被复制，使各进程拥有自己的拷贝。在页根本不会被写入的情况下（fork()后立即exec()），fork的实际开销只有复制父进程的页表以及给子进程创建唯一的task_struct。

创建进程的fork()函数实际上最终是调用clone()函数。创建线程和进程的步骤一样，只是最终传给clone()函数的参数不同。比如，通过一个普通的fork来创建进程，相当于：clone(SIGCHLD, 0)；创建一个和父进程共享地址空间，文件系统资源，文件描述符和信号处理程序的进程，即一个线程：clone(CLONE_VM | CLONE_FS | CLONE_FILES |CLONE_SIGHAND, 0)。

在内核中创建的内核线程与普通的进程之间还有个主要区别在于：内核线程没有独立的地址空间，它们只能在内核空间运行。

#### fork和vfork的区别

fork（）与vfock（）都是创建一个进程，那他们有什么区别呢？总结有以下三点区别：
1. fork()子进程拷贝父进程的数据段，代码段； vfork()子进程与父进程共享数据段
2. fork()父子进程的执行次序不确定； vfork 保证子进程先运行，在调用exec 或exit 之前与父进程数据是共享的, 之后父进程才可能被调度运行
3. vfork()保证子进程先运行，在她调用exec 或exit 之后父进程才可能被调度运行。如果在调用这两个函数之前子进程依赖于父进程的进一步动作，则会导致死锁。

## 进程的存储空间布局

1. c程序在链接时，由链接器创建一个指向程序起始地址的启动例程

## 进程切换

Linux做进程切换的时候，把通用寄存器 如eax, ebx, 等的内容存到内核栈里，把其他几乎所有寄存器的内容存到process descriptor的task_struct里。

进程切换只发生在内核态，此时用户态进程使用的所有寄存器内容已经被保存在内核栈里了，包括指向用户栈指针地址的ss和esp寄存器对。


## 进程创建

Linux三种不同的创建进程机制：
1. Copy on Write：创建进程时，子进程拥有父进程相同的物理页，当一个进程试图改写某个物理页时，内核复制该页并分配给改写进程进行改写操作。
2. Lightweight process：共享很多内核数据结构，例如页表、打开的文件表、信号处理器
3. vfork()：共享内存地址空间，相当于创建一个线程

C库的clone()函数，它会为新的LWP创建一个栈，然后隐式的调用clone()系统调用。clone()系统调用是没有fn和arg参数的，C库的封装函数会把fn指针保存在 子进程的封装函数的返回地址 对应的栈位置里，参数arg保存在子进程的栈中fn的后面。当子进程从clone()封装函数返回的时候，CPU从栈里获取返回地址，然后执行fn(arg)

fork()系统调用是用clone()实现的，flags参数值设定SIGCHILD信号，所有的CLONE_flags都被清空，child_stack参数等于当前父进程的栈指针。这就是 写时拷贝 技术，父子进程暂时共用用户栈，

vfork()系统调用也是用clone()实现的，flags参数被设为SIGCHILD|CLONE_VM|CLONE_VFORK，child_stack参数也等于父进程的指针。

vfork函数用于创建一个新进程，而新进程的目的是exec一个新程序。父进程会等待子进程执行

clone(), fork(), vfork()系统调用最后都是用的do_fork()函数，过程如下：
1. 分配PID，会查找pidmap_array bitmap，找到可用的PID
2. 检查父进程的ptrace字段：如果为0，即父进程正在被跟踪，就要检查调试器是否要跟踪子进程(这里跟父进程指定的CLONE_PTRACE无关);如果要跟踪，且子进程不是内核线程(CLONE_UNTRACED也没被设置)，就设置CLONE_PTRACE标志。
3. 调用copy_process()复制进程描述符。
4. 如果设置了CLONE_STOPPED标示，或者子进程必须被跟踪，则子进程状态会被设置为TASK_STOPPED，然后还要加一个未决信号SIGSTOP。
5. 如果CLONE_STOPPED标示未设置，就调用wake_up_new_task()函数：
 a. 调整父子进程的调度参数
 b. 如果父子运行在同一个CPU，且父子进程不共享页表(CLONE_VM标志为空)，将把子进程插入到父进程的runqueue里父进程前面位置，子进程先执行。这样如果子进程立马刷新页表执行新的程序，就减少了父进程提前执行时写时拷贝机制带来的不必要的页拷贝。
 c. 如果不在一个CPU执行，或者共享页表(CLONE_VM标志被设置)，则把子进程插入到父进程runqueue的队尾。
6. 如果父进程被跟踪，就把子进程的PID存到当前进程的ptrace_message里，然后调用ptrace_notify()，停止当前进程，向父进程发送SIGCHILD信号。祖父进程debugger接收到SIGCHILD信号，就知道当前创建了个子进程，PID在current->ptrace_message里保存
7. 如果CLOEN_VFORK标示被设置，父进程会被加入等待队列，然后挂起直到子进程释放内存地址空间（exec或者exit）
8. 返回子进程PID，结束

## 进程终止

一个进程的结束，是由调用exit()系统调用来完成的，exit()将在内核中调起do_exit()函数，流程如下：

   将current->flag的PF_EXITING置上；
   调用del_timer_sync()来取消内核timer，当此函数返回时，可以保证没有timer handler在运行以及没有timer在queue中
   如果BSD accounting is enable, 则调用acct_process()来写accounting info
   调用__exit_mm()来释放被进程占用的mm_struct，如果没有其他进程(线程)在使用这个内存空间，则deallocate
   调用exit_sem().若process有在queue中等待的信号量(sem)，则在这里将其dequeue
   调用__exit_files(), \__exit_fs(), exit_namespace(), exit_sighand()用来减少对文件操作符以及filesystem data，process namespace，signal handler的引用计数；如果有引用计数为0，则这个对象没有被任何process使用，于是就将其移除
   随后，current->exit_code被设置，用于之后parent取得该值
   之后，调用 exit_notify()来发送一个信号给parent，同时将current->state置为TASK_ZOMBIE
   最后，调用 schedule()将当前进程换出
