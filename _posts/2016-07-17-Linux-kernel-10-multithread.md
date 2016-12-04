---
layout: post
title: Linux内核分析（十）线程
categories: [linux kernel ]
tags: [linux kernel, ]
description: 理解linux 线程设计
---

## 一、进程？轻量进程？线程？内核线程？

欲知详情，还是manual手册靠谱啊！！！还在为进程、轻量进程、线程、用户线程、内核线程傻傻分不清楚吗？看看manual吧

### 1.1 man fork

fork通过复制调用进程来创建一个新进程作为子进程，父子进程在独立的内存空间运行。一些特点，具体看`man fork`

 - 子进程有独立的进程ID
 - 子进程的父进程ID就是调用者进程
 - 子进程不继承父进程的内存锁
 - 子进程的资源利用率和CPU时间重置为0
 - 子进程的未决信号(pending signal)初始为空
 - 子进程不继承信号量
 - 子进程不继承进程相关的记录锁(On the other hand, it does inherit fcntl(2) open file description locks and flock(2) locks from its parent)
 - 子进程不继承timers
 - 子进程不继承异步I/O相关的操作和内容
 - 等等...

> Under Linux, fork() is implemented using copy-on-write pages, so the only penalty that it incurs is the time and memory required to duplicate the parent's page tables, and create a unique task structure for the child.

> **c library/kernel differences** Since version 2.3.3, rather than invoking the kernel's fork() system call, the glibc fork() wrapper that is provided as part of the NPTL threading implementation invokes clone(2) with flags that provide the same effect as the traditional system call. (A call to fork() is equivalent to a call to clone(2) specifying flags as just SIGCHILD.) The glibc wrapper invokes any fork handlers that have been established using pthread_atfork(3).

> ——man fork:NOTES

以上是 man：NOTES 部分的内容，大意为

Linux中使用copy-on-write来实现fork调用，所需的时空开销仅仅是复制父进程的页表、创建子进程的task_struct所占用的时空，时空开销已经很小了，我已经炒鸡轻量咯。

c函数库跟内核在fork上的实现的不同点：Linux 2.3.3之后，glibc的fork()不是简单的调用fork()系统调用来实现，而被封装为 **“调用clone(2)来实现线程的NPTL”** 的一部分，其中clone()调用具有flags参数，能产生跟传统系统调用fork()一样的效果。(当flags参数为SIGCHLD时，fork()系统调用跟clone()系统调用一样) glibc使用pthread_atfork(3)来实现所有的fork调用。

### 1.2 man clone(2)

```c
/* glibc封装的函数原型 */

# include <sched.h>
/**
* @fn(arg) 子进程创建后执行的方法
* @child_stack 子进程使用的栈地址
* @flags The low byte contains the number of the termination signal sent to the parent when the child dies,
*        Also be bitwise-of'ed * with zero or more of the following constants, in order to specify
*        what is shared between the calling process and the child process: for detail, see man clone(2)
* @arg 子进程要执行的方法的参数
*/
int clone(int (*fn)(void *), void *child_stack,
          int flags, void *arg, ...
        /* pid_t *ptid, struct user_desc *tls, pid_t *ctid */);


/* 系统调用的原型 */

long clone(unsigned long flags, void *child_stack,
           void *ptid, void *ctid,
           struct pt_regs *regs);
```

> Unlike fork(2), clone() allows the child process to share parts of its execution context with the calling process, such as **the memory space**, the table of **file descriptors**, and the table of **signal handlers**.

> One use of clone() is to **implement threads**: multiple threads of control in a program that run concurrently in a shared memory space.

>——man clone(2):DESCRIPTION

大意就是：clone()可以通过参数设置共享进程的执行上下文(内存空间、文件描述符、信号)，NPTL通过clone()来实现线程。所以到目前为止，Linux都是使用进程来实现线程的。Linux使用的线程库NPTL，也可以说是对clone实现的封装，以此带来了可移植性。

### 1.3 Linux kernel development

《Linux 内核设计和实现》3.4节《The Linux Implementation of Threads》写的很清楚呢：

> Linux has a unique implementation of threads. To the Linux kernel, there is no concept of a thread. Linux implements all threads as standard processes. The Linux kernel does not provide any special scheduling semantics or data structures to represent threads. Instead, a thread is merely a process that shares certain resources with other processes. Each thread has a unique task_struct and appears to the kernel as a normal process--threads just happen to share resources, such as an address space, with other processes.

> This approach to threads contrasts greatly with operating systems such as Microsoft Windows or Sun Solaris, which have explicit kernel support for threads (and sometimes call threads lightweight processes). The name "lightweight process" sums up the difference in philosophies between Linux and other systems. To these other operating systems, threads are an abstraction to provide a lighter, quicker execution unit than the heavy process. **To Linux, threads are simply a manner of sharing resources between process**(which are already quite lightweight).

> For example, assume you have a process that consists of four threads. On systems with explicit thread support, one process descriptor might exist that, in turn, points to the four different threads. The process descriptor describes the resources they alone posses. Conversely, in Linux, there are simply four processes and thus four normal task_struct structures. The four processes are set up to share certain resources. The result is quite elegant.

大意就是：

其他操作系统的进程设计的很笨重，因此使用线程来达到轻量的目的，所以其他操作系统要单独设计线程。

Linux根本没有线程、没有轻量进程，只有进程的实现，因为我的进程已经够轻量的，我本身就是轻量的，数据结构只有task_struct，Linux的线程只是进程间共享资源的一种简单方式。

再说一遍，Linux的线程只是进程间共享资源的一种简单方式。

所以Linux只实现了进程，线程是对共享资源的进程的称呼，都是通过clone()系统调用创建的。

### 1.4 Kernel Threads

内核线程是内核在内核空间做的操作。

跟进程的区别就是没有地址空间，只在内核空间做操作（然后还是使用的clone()系统调用，还是用的task_struct数据结构，本质上还是进程）

内核线程只能由内核线程创建。

> It is often useful for the kernel to perform some operations in the background. The kernel accomplishes this via kernel threads--stadard processes that exist solely in kernel space. The significant difference between kernel threads and normal processes is that kernel threads do not have an address space. (Their mm pointer, which points at their address space, is NULL) They operate only in kernel space and do not context switch into user space. Kernel threads, however, are schedulable and preemptable, the same as normal processes.

> Linux delegates serveral tasks to kernel threads, most notably the *flush* tasks and the *ksoftirqd* task. You can see the kernel threads on your Linux system by running the command `ps -ef`. There are a lot of them! Kernel threads are created on system boot by other kernel threads. Indeed, a kernel thread can be created only by another kernel thread. The kernel handles this automatically by forking all new kernel threads off of the *kthread* kernel process. The interface, declared in <linux/kthread.h>.

> The new task is created via the *clone()* system call by the *kthread* kernel process. The process is created in an unrunnable state; it will not start running until explicitly woken up via *wake_up_process()*. A process can be created and made runnable with a single function *kthread_run()*.

### 1.5 小结

线程有两大优势：1.共享资源 2.轻量

Linux的进程已经足够轻量了，所以我只要再实现下共享资源就OK了。所以Linux的 *clone()*系统调用实现了通过参数指定共享资源，这样创建的共享资源的进程就是你们所说的线程了吧，呜哈哈！！

够优雅吧！这样以来，我就不用单独设计线程数据结构、不用单独对线程做调度，还可以实现你的线程的一切功能，爽歪歪啊。

NPTL是Linux现在使用的线程库，内部也是使用clone()系统调用。内核线程也就是没有用户空间的只能在内核空间活动的进程。一切皆是进程！！！

## 二、Linux线程的实现

### 2.1 POSIX threads

POSIX.1定义了多线程编程的接口，即POSIX threads, or Pthreads. 每个进程可以有多个线程，这些线程共享内存空间，但是各个线程有自己的线程栈。

POSIX.1还要求线程共享一些其他属性（那些进程范围内的属性，而不是每个线程的属性）：

 - process ID
 - parent process ID
 - process group ID and session ID
 - controlling terminal
 - user and group IDs
 - open file descriptors
 - record locks
 - signal dispositions
 - file mode creation mask
 - current directory and root directory
 - interval timers and POSIX timers
 - nice value
 - resource limits
 - measurements of the consumption of CPU time and resources

POSIX.1也指明了线程独有的属性：

 - thread ID (guaranteed to be unique only within a process)
 - signal mask
 - the errno variable
 - alternate signal stack
 - real-time scheduling policy and priority

另外，Linux还给每个线程加了这两个属性：

 - capabilities
 - CPU affinity

### 2.2 Linux implementations of POSIX threads

至今，总共出现了两种线程库的实现：

**LinuxThreads** 这是最初的实现，从glibc 2.4之后，就不在使用了。

最初，LinuxThreads项目使用 *clone()* 系统调用来完全在用户空间模拟对线程的支持。不幸的是，这种方法有一些缺点，尤其是在信号处理、调度和进程间同步原语方法都存在问题。另外，这个线程模型也不符合POSIX的要求。

**NPTL (Native POSIX Threads Library)** 这是现代的Pthreads实现。对比LinuxThreads，NPTL更加满足POSIX.1的标准，并且能更好的创建大量线程。NPTL开始于glibc 2.3.2，需要Linux 2.6内核的特性（内核对task_struct结构做了调整，添加了对线程组概念的变量支持）。

这两种实现都是“一对一”的线程模型，即一个线程对应一个内核调度实体 task_struct。这个模型最大的好处是线程调度由内核完成了，而其他线程操作（同步、取消）等都是核外的线程库函数完成的。线程实现利用 *clone()* 系统调用，同步原语(mutexes, thread joining, and so on)使用 *futex()* 系统调用。

linux上的线程就是基于轻量进程, 由用户态的pthread库实现的。使用pthread以后, 在用户看来, 每一个task_struct就对应一个线程, 而一组线程以及它们所共同引用的一组资源就是一个进程。但是, 一组线程并不仅仅是引用同一组资源就够了, 它们还必须被视为一个整体。

对此, POSIX标准提出了如下要求:

 1. 查看进程列表的时候, 相关的一组task_struct应当被展现为列表中的一个节点;
 2. 发送给这个"进程"的信号(对应kill系统调用), 将被对应的这一组task_struct所共享, 并且被其中的任意一个"线程"处理;
 3. 发送给某个"线程"的信号(对应pthread_kill), 将只被对应的一个task_struct接收, 并且由它自己来处理;
 4. 当"进程"被停止或继续时(对应SIGSTOP/SIGCONT信号), 对应的这一组task_struct状态将改变;
 5. 当"进程"收到一个致命信号(比如由于段错误收到SIGSEGV信号), 对应的这一组task_struct将全部退出;
 6. 等等(以上可能不够全);

### 2.3 关于NPTL

NPTL实现了前面提到的POSIX的全部5点要求。 但是，实际上，与其说是NPTL实现了，不如说是linux内核实现了。还有如下一点POSIX.1没能实现：

 - Threads do not share a common nice value

在linux 2.6中，内核有了线程组的概念，task_struct结构中增加了一个tgid(thread group id)字段。

如果这个task是一个"主线程"，则它的tgid等于pid，否则tgid等于进程的pid(即主线程的pid)。

在 *clone()* 系统调用中, 传递CLONE_THREAD参数就可以把新进程的tgid设置为父进程的tgid(否则新进程的tgid会设为其自身的pid).

类似的XXid在task_struct中还有两个：task->signal->pgid保存进程组的打头进程的pid、task->signal->session保存会话打头进程的pid。通过这两个id来关联进程组和会话。

有了tgid, 内核或相关的shell程序就知道某个tast_struct是代表一个进程还是代表一个线程, 也就知道在什么时候该展现它们, 什么时候不该展现(比如在ps的时候, 线程就不要展现了)。

而getpid()(获取进程ID)系统调用返回的也是tast_struct中的tgid, 而tast_struct中的pid则由gettid()系统调用来返回。

在执行ps命令的时候不展现子线程，也是有一些问题的。比如程序a.out运行时，创建 了一个线程。假设主线程的pid是10001、子线程是10002（它们的tgid都是10001）。这时如果你kill 10002，是可以把10001和10002这两个线程一起杀死的，尽管执行ps命令的时候根本看不到10002这个进程。如果你不知道linux线程背 后的故事，肯定会觉得遇到灵异事件了。

为了应付"发送给进程的信号"和"发送给线程的信号", task_struct里面维护了两套signal_pending, 一套是线程组共享的, 一套是线程独有的。

通过kill发送的信号被放在线程组共享的signal_pending中, 可以由任意一个线程来处理; 通过pthread_kill发送的信号(pthread_kill是pthread库的接口, 对应的系统调用中tkill)被放在线程独有的signal_pending中, 只能由本线程来处理。

当线程停止/继续, 或者是收到一个致命信号时, 内核会将处理动作施加到整个线程组中。

## 三、线程使用

```c
/**
* man pthread_create 里的例子
* compile: gcc pthread_create_test.c -o pthread_create_test -lpthread
* run: ./pthread_create_test -s:8192 wuhahahha
*/
#include <pthread.h>
#include <string.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <ctype.h>

#define handle_error_en(en, msg) \
       do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)

#define handle_error(msg) \
       do { perror(msg); exit(EXIT_FAILURE); } while (0)

struct thread_info {    /* Used as argument to thread_start() */
   pthread_t thread_id;        /* ID returned by pthread_create() */
   int       thread_num;       /* Application-defined thread # */
   char     *argv_string;      /* From command-line argument */
};

/* Thread start function: display address near top of our stack,
  and return upper-cased copy of argv_string */

static void *
thread_start(void *arg)
{
   struct thread_info *tinfo = arg;
   char *uargv, *p;

   printf("Thread %d: top of stack near %p; argv_string=%s\n",
           tinfo->thread_num, &p, tinfo->argv_string);

   uargv = strdup(tinfo->argv_string);
   if (uargv == NULL)
       handle_error("strdup");

   for (p = uargv; *p != '\0'; p++)
       *p = toupper(*p);

   return uargv;
}

int
main(int argc, char *argv[])
{
    int s, tnum, opt, num_threads;
    struct thread_info *tinfo;
    pthread_attr_t attr;
    int stack_size;
    void *res;

    /* The "-s" option specifies a stack size for our threads */

    stack_size = -1;
    while ((opt = getopt(argc, argv, "s:")) != -1) {
        switch (opt) {
        case 's':
            stack_size = strtoul(optarg, NULL, 0);
            break;

        default:
            fprintf(stderr, "Usage: %s [-s stack-size] arg...\n",
                    argv[0]);
            exit(EXIT_FAILURE);
        }
    }

    num_threads = argc - optind;

    /* Initialize thread creation attributes */

    s = pthread_attr_init(&attr);
    if (s != 0)
        handle_error_en(s, "pthread_attr_init");

    if (stack_size > 0) {
        s = pthread_attr_setstacksize(&attr, stack_size);
        if (s != 0)
            handle_error_en(s, "pthread_attr_setstacksize");
    }
    /* Allocate memory for pthread_create() arguments */

    tinfo = calloc(num_threads, sizeof(struct thread_info));
    if (tinfo == NULL)
       handle_error("calloc");

    /* Create one thread for each command-line argument */

    for (tnum = 0; tnum < num_threads; tnum++) {
       tinfo[tnum].thread_num = tnum + 1;
       tinfo[tnum].argv_string = argv[optind + tnum];

       /* The pthread_create() call stores the thread ID into
          corresponding element of tinfo[] */

       s = pthread_create(&tinfo[tnum].thread_id, &attr,
                          &thread_start, &tinfo[tnum]);
       if (s != 0)
           handle_error_en(s, "pthread_create");
    }

    /* Destroy the thread attributes object, since it is no
      longer needed */

    s = pthread_attr_destroy(&attr);
    if (s != 0)
       handle_error_en(s, "pthread_attr_destroy");

    /* Now join with each thread, and display its returned value */

    for (tnum = 0; tnum < num_threads; tnum++) {
       s = pthread_join(tinfo[tnum].thread_id, &res);
       if (s != 0)
           handle_error_en(s, "pthread_join");

       printf("Joined with thread %d; returned value was %s\n",
               tinfo[tnum].thread_num, (char *) res);
       free(res);      /* Free memory allocated by thread */
    }

    free(tinfo);
    exit(EXIT_SUCCESS);
}

```

## 参考文章

1. 《Linux kernel development》

2. [Linux 线程实现机制 IBM2003](http://www.ibm.com/developerworks/cn/linux/kernel/l-thread/)

3. [Linux 线程模型的比较：LinuxThreads 和 NPTL IBM2006](http://www.ibm.com/developerworks/cn/linux/l-threading.html)

4. [linux线程浅析](http://blog.chinaunix.net/uid-28541347-id-4406541.html)

5. [Linux进程、线程模型，LWP，pthread_self()](http://blog.csdn.net/tianyue168/article/details/7403693)

6. [fork, vfork和 clone实现分析 这个挺清晰](http://blog.chinaunix.net/uid-28541347-id-4598097.html)
