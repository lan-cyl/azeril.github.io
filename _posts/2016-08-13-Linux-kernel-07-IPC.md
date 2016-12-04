---
layout: post
title: Linux内核分析（七）进程间通信
categories: [linux kernel ]
tags: [linux kernel, ]
description: APUE 进程间通信
---

#### 管道

管道是一个贴切的名字，跟我们生活中见到的管道一样，可以把一个进程的标准输出和另一个进程的标准输入连接起来。例如shell里经常用到的

写进程在管道的尾端写入数据，读进程在管道的首端读出数据。

管道本质上就是一个文件，前面的进程以写方式打开文件，后面的进程以读方式打开，这样前面的写完后面就可以读，于是就实现了通信。Linux系统直接将管道实现成了一种文件系统pipefs，借助VFS给应用程序提供操作接口。但是管道本身不占用磁盘或者其他外部存储空间，仅存在于内存空间。所以，Linux的管道就是一个以文件方式操作的内存缓冲区

管道分为 匿名管道、命名管道。匿名管道用于父子进程间通信，命名管道用于任何两个进程之间的通信（通过管道文件）。除此之外，这两种管道都是一样的

当管道一端被关闭后，会发生如下情况：

 1. 当读一个写端已被关闭的管道时，在所有数据都被读取后，read返回0，表示文件结束
 2. 当写一个读端已被关闭的管道时，则产生信号SIGPIPE。忽略或捕捉该信号之后，write返回-1，errno设置位EPIPE

管道的创建方法：
```c
#include <unistd.h>

/**
 * 创建匿名管道
 * @pipefd pipefd[0]是读方式打开，作为管道的读描述符
 *         pipefd[1]是写方式打开，作为管道的写描述符
 * @return 若成功 返回0；否则 返回-1
 */
int pipe(int pipefd[2]);

#include <sys.stat.h>
int mkfifo(const char* path, mode_t mode);
int mkfifoat(int fd, const char* path, mode_t mode);
```

#### XSI IPC

> 每个内核中的IPC结构都用一个非负整数的标识符（identifier）加以引用
> 标识符是IPC对象的内部名，对象的外部名为一个键（key_t 被定义为长整型）
> 在一个可能会使用共享内存的项目组中，大家可以约定一个文件名和一个项目的proj_id，然后使用ftok确定一段共享内存的key。

##### 消息队列

> 消息队列是消息的链接表，存储在内核中，由消息队列标识符标识。

操作流程：

1. msgget()创建一个新队列，或者引用一个现有队列
2. msgctl()获取、修改队列状态信息、删除队列
3. msgsnd()将任意类型的消息发送到消息队列尾端，消息都有自己的type字段
4. msgrcv()得到第一个消息/得到第一个该类型的消息等

```c
/**
 * 每个队列都有个这个结构，记录消息队列的信息
 */
struct msqid_ds {
  struct ipc_perm msg_perm;   //有效用户、组ID，访问权限等
  msgqnum_t       msg_qnum;   //队列中消息数
  msglen_t        msg_qbytes; //能使用的最大字节数
  pid_t           msg_lspid;  //last send pid
  pid_t           msg_lrpid;  //last rcv pid
  time_t          msg_stime;  //last send time
  time_t          msg_rtime;  //last rcv time
  time_t          msg_ctime;  //last change time
  ...
};

#include <sys/msg.h>

/**
 * 打开一个现有队列，或创建新队列
 * @key   消息队列的外部键名
 * @flag  消息队列的ipc-perm.mode的权限位
 * return 消息队列的内部标识符
*/
int msgget(key_t key, int flag);

/**
 * 对队列进行操作的函数
 * @msqid 消息队列的内部标识符
 * @cmd   IPC_STAT  获取此队列的msqid_ds结构，并放到buf中
 *        IPC_SET   从buf中设置msg_perm.uid/msg_perm.gid/msg_perm.mode/msg_pbytes
 *        IPC_RMID  从系统中删除该消息队列，及其中的所有数据
 * @buf   保存消息队列的状态
 * return 若成功 返回0；否则 返回-1
*/
int msgctl(int msqid, int cmd, struct msqid_ds* buf);

/**
 * 将数据放到消息队列尾端：成功时更新msgid_ds结构的msg_lrpid/msg_rtime/msg_qnum+1
 * @msqid 消息队列的内部标识符
 * @ptr   待存放的数据：类型字段、非负的长度、实际的数据
 * @nbytes数据的长度
 * @flag  可以设为非阻塞：IPC_NOWAIT
 * return 若成功 返回0；否则 返回-1
*/
int msgsnd(int msqid, const void* ptr, size_t nbytes, int flag);

/**
 * 一个可能的消息结构，即ptr指向的对象
*/
struct mymesg {
  long mtype;       // positive message type
  char mtext[512];  // message data, of length nbytes
}

/**
 * 从队列中取用消息：成功时更新msgid_ds结构的msg_lrpid/msg_rtime/msg_qnum-1
 * @msqid 消息队列的内部标识符
 * @ptr   待存放的数据：类型字段、非负的长度、实际的数据
 * @nbytes数据的长度
 * @type  =0 返回队列中第一个消息
          >0 返回队列中消息类型为type的第一个消息
          <0 返回队列中消息类型值小于等于type，且类型值最小的那个消息
 * @flag  可以设为非阻塞：IPC_NOWAIT
 * return 若成功 返回0；否则 返回-1
*/
int msgrcv(int msqid, void* ptr, size_t nbytes, long type, int flag);

```

apue的最后结论是，不推荐再使用消息队列了，原因就是IPC存在的一些缺陷

1. 系统范围内起作用，且没有引用计数，删除是个问题
2. IPC结构在文件系统中没有名字，不得不增加新的系统调用
3. 不使用文件描述符，所以不能使用多路转接函数，有些功能就实现不了。

##### 信号量

> 信号量跟其他的IPC机制（管道、FIFO以及消息队列）不同，它是一个计数器，用于为多个进程提供对共享数据对象的访问

操作流程：

1. semget()创建含有nsems个信号量的信号量集，或者引用现有信号量集
2. semctl()初始化各个信号量值，获得、修改信号量集状态、删除信号量集
3. semop()原子地增加、减少信号量

```c
/**
 * 同样的，信号量也有一个维护信息的结构
 */
struct semid_ds {
  struct ipc_perm sem_perm;   //有效用户、组ID，访问权限等
  unsigned short  sem_nsems;  //信号量集合的大小
  time_t          sem_otime;  //last-semop time
  time_t          sem_ctime;  //last-change time
  ...
};

struct {
  unsigned short  semval;   // semaphore value, always >= 0
  pid_t           sempid;   // pid for last operation
  unsigned short  semncnt;  // num of processes awaiting semval > curval
  unsigned short  semzcnt;  // num of processes awaiting semval == 0
  ...
}

#include <sys/sem.h>

/**
 * 打开一个现有信号量集合，或创建新信号量集合
 * @key   信号量集合的外部键名
 * @nsems 信号量集合的大小
 * @flag  信号量集合的ipc-perm.mode的权限位
 * return 信号量集合的内部标识符
*/
int semget(key_t key, int nsems, int flag);

/**
 * 对信号量集合进行操作
 * @semid 信号量集合的内部标识符
 * @semnum指向该信号量集合中的一个成员
 * @cmd   IPC_STAT  获取此队列的semid_ds结构，并放到buf中
 *        IPC_SET   从arr.buf中设置sem_perm.uid/sem_perm.gid/sem_perm.mode
 *        IPC_RMID  从系统中删除该消息队列，及其中的所有数据
 *        GETVAL    返回成员semnum的semval值
 *        SETVAL    设置成员semnum的semval值，由arg.val指定
 *        GETPID    返回成员semnum的sempid值
 *        GETNCNT   返回成员semnum的semncnt值
 *        GETZCNT   返回成员semnum的semzcnt值
 *        GETALL    获取集合中所有的信号量值，存储在arg.array中
 *        SETALL    使用arg.array中的值设置集合中所有的信号量
 * @arg   可选参数，定义如下
 * return 若成功 返回0；否则 返回-1
*/
int semctl(int semid, int semnum, int cmd, .../* union semun arg*/);
union semun {
  int             val;      // for SETVAL
  struct semid_ds* buf;     // for IPC_STAT and IPC_GET
  unisigned short* array;   // for GETALL and SETALL
}

/**
 * 原子操作，增加或减少信号量值
 * @semid       信号量集合的内部标识符
 * @semoparray  信号量操作数组，具体结构如下
 * @nops        操作数组的长度
 * return       若成功 返回0；否则 返回-1
*/
int semop(int semid, struct sembuf semoparray[], size_t nops);
struct sembuf {
  unsigned short  sem_num;  // 信号量集合中的成员 (0, 1, ..., nsems-1)
  short           sem_op;   // 正值，表示要释放的资源数，对信号量做加操作；
                            // 负值，表示要获取的资源数；
                            // 0，表示进程希望等待该信号量值为0
  short           sem_flg;  // IPC_NOWAIT, SEM_UNDO
}
```

结论就是，进程间同步推荐使用记录锁，简单、快捷、进程终止时系统会管理遗留下来的锁。信号量功能花哨，速度不高；互斥量速度最快，但是在多进程共享内存中使用互斥量恢复一个终止的进程更难，而且进程共享的互斥量也没有得到普遍支持

##### 共享内存

> 共享存储段允许多个进程共享一个给定的存储区。
> 因为数据不需要在进程间复制，所以这是最快的IPC。
> 要注意同步访问共享存储区。通常信号量用于同步（如前节所述，记录锁或互斥量也可以）
> 跟mmap在实现上并没有本质区别，主要是为了在非父子关系的进程之间共享内存

操作流程：

1. shmget()创建共享存储段，这个共享存储段是在内核中的相关结构
2. shmat() 将共享存储段关联到进程地址空间中
3. shmdt() 分离该地址空间段，但不会删除的，删除都是用shmctl的IPC_RMID

```c
/**
 * 同样的，共享存储段也有一个维护信息的结构
 */
struct shmid_ds {
  struct ipc_perm shm_perm;   //有效用户、组ID，访问权限等
  size_t          shm_segsz;  //段大小
  pid_t           shm_lpid;   //pid of last shmop()
  pid_t           shm_cpid;   //pid of creator
  shmatt_t        shm_nattach;//当前附加了内存地址的数量
  time_t          shm_atime;  //last-attach time
  time_t          shm_dtime;  //last-detach time
  time_t          shm_ctime;  //last-change time
  ...
};

#include <sys/shm.h>

/**
 * 打开一个现有共享内存段，或创建新的
 * @key   共享内存段的外部键名
 * @size  共享内存段的大小
 * @flag  共享内存段的ipc-perm.mode的权限位
 * return 共享内存段的内部标识符
*/
int shmget(key_t key, size_t size, int flag);

/**
 * 对共享内存段进行操作
 * @shmid 共享内存段的内部标识符
 * @cmd   IPC_STAT  获取shmid_ds结构，并放到buf中
 *        IPC_SET   从buf中设置shm_perm.uid/shm_perm.gid/shm_perm.mode
 *        IPC_RMID  从系统中删除该共享内存段，及其中的所有数据
 * @buf   共享内存段结构信息
 * return 若成功 返回0；否则 返回-1
*/
int shmctl(int shmid, int cmd, shmid_ds* buf);

/**
 * 将共享内存段连接到进程的虚拟地址空间中；shmid_ds.shm_nattach++
 * @shmid 共享内存段的内部标识符
 * @addr  ==0 则此段连接到内核选择的第一个可用地址上
 *        !==0 && !SHM_RND 则使用该地址连接
 *        !==0 && SHM_RND  则连接到地址（addr-(addr mod SHMLBA)）
 * @flag  操作数组的长度
 * return 若成功，则返回指向共享存储段的进程空间的虚地址；否则 返回-1
*/
void *shmat(int shmid, const void* addr, int flag);

/**
 * 将共享内存段与连接的进程的地址空间分离，仅仅分离，并不删除；shmid_ds.shm_nattach--
 * @addr  连接的地址
 * return 若成功 返回0；否则 返回-1
*/
int shmdt(const void* addr);

```

mmap文件映射能将同一文件映射到两个无关进程的地址空间，通过文件实现进程通信

mmap的匿名文件映射只能在父子进程间共享，因为没有办法将地址传给其他无关进程，因此引入共享内存。这两种方式在内核底层都是使用tmpfs方式实现的。

XSI 的key+projid的命名方式不够UNIX，没用遵循一切皆文件的设计理念。因此出现了POSIX 标准的进程通信机制，他们使用文件描述符的方式进行管理，因此可以结合select、poll这样的IO异步事件驱动机制做一些更高级的功能

#### POSIX 共享内存

POSIX 共享内存本质上就是mmap对文件的共享方式映射，只不过映射的是tmpfs文件系统上的文件，而不是普通的磁盘文件。

Linux提供的POSIX共享内存，实际上就是在/dev/shm下创建一个文件，并将其mmap之后映射其内存地址即可。

相关函数：
```c
#include <sys/mman.h>
#include <sys/stat.h>        /* For mode constants */
#include <fcntl.h>           /* For O_* constants */

// 创建或者访问一个已经创建的共享内存，就是open系统调用的封装
int shm_open(const char* name, int oflag, mode_t mode);

int shm_unlink(const char* name);
```

映射共享内存地址使用mmap，解除映射使用munmap。使用ftruncate设置共享内存大小，实际上就是对tmpfs的文件进行指定长度的截断。使用fchmod、fchown、fstat等系统调用修改和查看相关共享内存的属性。close调用关闭共享内存的描述符。实际上，剩下的都是标准的文件操作。

#### POSIX 信号量

SUSv4将POSIX信号量放到了基本规范里。而消息队列和共享存储接口依然是共享的

POSIX信号量相对于XSI信号量的优势：

1. POSIX信号量考虑了更高性能的实现
2. POSIX信号量接口更简单：没有信号量集，并且使用文件操作的方式对该接口进行模式化
3. 删除信号量时也能像文件那样，最后一个引用释放时删除

命名信号量的创建和销毁：

```c
#include <semaphore.h>

/**
 * 创建一个新的命名信号量，或者使用一个现有的
 * @name  信号量的名字
 * @oflag 0 使用现有的；O_CREAT|O_EXCL 创建标志
 * @mode  指定访问权限位
 * @value 信号量的初始值
 * return 若成功，返回指向信号量的指针；否则 返回SEM_FAILED
 */
sem_t *sem_open(const char* name, int oflag, ...
                /* mode_t mode, unsigned int value*/ );

/**
 * 释放信号量相关的所有资源，不改变信号量值。进程退出时也会自动关闭，类似文件
 */
int sem_close(sem_t* sem);

/**
 * 使用名字来销毁一个命名信号量，会等到所有引用关闭
 */
int sem_unlink(const char* name);

```

未命名信号量的创建和销毁：

```c
#include <semaphore.h>

/**
 * 创建一个新的命名信号量，或者使用一个现有的
 * @sem     信号量，如果要在多个进程中使用，确保sem指向两进程共享的内存范围
 * @pshared 非0 表示在多个进程中使用该信号量
 * @value   信号量的初始值
 * return   若成功，返回0；否则 返回-1
 */
int sem_init(sem_t* sem, int pshared, unsigned int value);

/**
 * 销毁信号量，销毁之后就不能在使用了
 */
int sem_destrory(sem_t* sem);

```

信号量的操作：

```c
#include <semaphore.h>

// 获取信号量
int sem_trywait(sem_t* sem);
int sem_wait(sem_t* sem);
int sem_timedwait(sem_t* restrict sem,
                  const struct timespec* restrict tsptr);

// 释放信号量，对信号量做加操作
int sem_post(sem_t* sem);

int sem_getvalue(sem_t* restrict sem, int* restrict valp);

```

#### 进程间通信小结

1. 全双工管道优于消息队列
2. 记录锁优于信号量
3. 共享内存仍然有自己的用途。虽然mmap可以实现同样的功能

### 网络IPC：套接字

> 同样的接口，既可以用在计算机间通信，也可以用在计算机内通信
