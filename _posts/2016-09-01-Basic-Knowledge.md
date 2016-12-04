---
layout: post
title: 计算机基础知识总结
categories: [Basic ]
tags: [Basic, ]
description: 计算机基础知识总结，面试知识点整理
---

## 数据库相关

#### 1.Mysql的查询过程？

 - 应用程序的数据访问层请求DataSource来获取一个数据库连接
 - DataSource使用数据库驱动来打开一个连接
 - 创建数据库连接，同事打开一个TCP socket
 - 应用程序进行数据库读写
 - 关闭连接
 - 关闭socket

#### 2.MySQL 对于千万级的大表要怎么优化？

 1. 优化sql和索引
 2. 加缓存，redis、memcached
 3. 主从复制，做读写分离
 4. 分区
 5. 垂直拆分：将多字段的表 根据模块耦合度 拆分成多个小表，就可以做分布式
 6. 水平切分：按hash值、固定大小、按时间 切分成多个表

#### 3.Myisam和InnodB比较？

基本的差别是：Myisam类型不支持事务等高级处理，而Innodb支持

 1. InnoDB不支持全文索引，可以用插件
 2. InnoDB不保存表的具体行数，而Myisam保存，但是有where条件时，操作都一样，都需遍历整个表计算行数
 3. 对于 AUTO_INCREMENT类型的字段，InnoDB必须包含只有该字段的索引，Myisam可以和其他字段一起建立联合索引
 4. Delete from table时，InnoDB不会重新建立表，而是一行一行的删除
 5. InnoDB支持行锁，但是仅仅能确定扫描范围的情况下。如果Mysql不能确定要扫描的范围，InnoDB同样会锁全表，例如 update table set num=1 where name list '%anna%'

#### 4.Mysql的性能优化tips？

 1. 优化查询，来使用查询缓存。例如where signup_data >= curdate() 改为 today=data("Y-m-d"); where signup_data > 'today';像now()/rand()之类的sql函数都不会开启查询缓存，因为函数返回值是易变的
 2. EXPLAIN你的SELECT查询，可以看到查询语句是怎样执行的
 3. 当只有一行数据时使用LIMIT 1，这样Mysql引擎会在找到一条数据之后停止搜索
 4. 为搜索字段建索引
 5. 两个表中Join的字段是被建过索引的，且类型一致
 6. 千万不要 ORDER BY RAND()
 7. 避免 SELECT * ，从数据库读出越多数据，查询就会变得越慢，特别是数据库服务器和WEB服务器时两台独立主机时，还会增加网络负载
 8. 为每张表设置一个ID，最好是 UNSIGNED INT型，并设置为 AUTO_INCREMENT。因为很多操作需要使用主键
 9. 使用 ENUM 而不是VARCHAR
 10. 从 PROCEDURE ANALYSE()取得建议
 11. 尽可能使用 NOT NULL，简化程序的判断逻辑，NULL值也是需要存储空间的
 12. 当一个相同的查询被使用多次的时候，使用 Prepared Statements
 13. 把 IP地址存成 UNSIGNED INT
 14. 固定长度的表会更快，可以使用垂直分割技术，将一个表分割为 定长的，和不定长的两个表
 15. 垂直分割，将不经常查询的字段单独放出来，将经常更新的字段单独放出来 这样就不会影响其他数据的缓存有效性。要确保分出来的数据不会被经常性的Join起来
 16. 拆分大的 DELETE或 INSERT语句。这两个操作会锁表，造成整个数据库的阻塞，堆积大量的http请求、数据库链接请求、打开文件等，造成服务器当机。可以使用LIMIT条件每次执行一定的语句
 17. 越小的列会越快
 18. 选择正确的存储引擎：Myisam适合大量读、少量写；InnoDB支持 行锁、事务，适合写操作比较多的应用
 19. 使用一个对象关系映射器 ORM，例如Mybatis

#### 5.数据库事务并发问题

事务的特点：ACID

多个事务同时访问数据库时，会发生如下5类问题：3类数据读问题（脏读、不可重复读、幻读）2类数据更新问题（第一类丢失更新、第二类丢失更新）

 1. 脏读（dirty read）：A事务读取B事务尚未提交的更改数据并在这个基础上操作。如果B事务回滚，那么A事务读到的数据根本不是合法的，成为脏读。
 2. 不可重复读（unrepeatable read）:A事务读取B事务已经提交的更改（或删除）数据时，在B事务提交前后A事务读取的结果不一致。———加行级锁，数据库默认
 3. 幻读（phantom read）：B事务新增一条记录的情况下，A事务在B事务提交前后读取到的数据条数不一致。——加表锁，开销太大，一般都不加
 4. 第一类丢失更新：A事务撤销时，把B事务提交的数据修改覆盖掉。
 5. 第二类丢失更新：A事务提交时，把B事务提交的数据修改覆盖掉。

#### 6.数据库的悲观锁和乐观锁

 - 悲观锁：假设数据肯定会冲突，所以在数据开始读取的时候就把数据锁住。
 - 乐观锁：认为数据一般不会造成冲突，所以在数据提交时才会检查数据是否冲突，如果发生冲突则返回错误信息，让用户决定如何去做。
 - 乐观锁的三种实现方式：（都是CAS操作）
	1. 整个数据copy到应用中，提交时对比当前数据的数据和copy的数据是否一致，由于double型不可比，所以一般不用。
	2. 采用版本戳，数据表增加一个新的column，比如是number型的，每次更新时都加一，通过对比number来判断一致性。
	3. 采用时间戳，跟第2中类似，不过数据类型是timestamp型的。

#### 7.Innodb默认使用乐观锁还是悲观锁

默认使用乐观锁，MVCC（多版本并发控制）来处理不可重复读和幻读。Innodb会在每行数据后添加两个额外的隐藏值来实现MVCC。（参见http://tech.meituan.com/innodb-lock.html）

#### 8.Mysql和mongodb索引原理（说一下B+tree和B-tree区别，为什么为什么一个用b+一个用b-，为什么索引用btree不用平衡二叉树）

B+树的非叶子节点都是索引结构，Data全部在叶节点保存。优点1：每个索引节点可以包含更多子节点，一般上千个，降低B树的深度，较少磁盘I/O次数；优点2：叶子节点用指针串起来，可以完成区间访问

平衡二叉树是通过旋转来保持平衡的，而旋转是对整棵树的操作，若部分加载到内存中则无法完成旋转操作。其次平衡二叉树的高度相对较大为 log n（底数为2），这样逻辑上很近的节点实际可能非常远，无法很好的利用磁盘预读（局部性原理），所以这类平衡二叉树在数据库和文件系统上的选择就被 pass 了。

参看：[MySQL索引原理及慢查询优化](http://tech.meituan.com/mysql-index.html)

[MySQL索引背后的数据结构及算法原理](http://blog.codinglabs.org/articles/theory-of-mysql-index.html)

[从 MongoDB及 Mysql谈 B/B+树](http://blog.csdn.net/wwh578867817/article/details/50493940)

#### 9.数据库连接池？

连接池能带来效率的明显提升，较少连接创建销毁的时间、tcp socket创建销毁的时间等。但是连接池的参数配置又带来额外的复杂性

参看：[数据库连接池简析](http://it.deepinmind.com/db/2014/05/04/the-anatomy-of-connection-pooling.html)

#### 10.一致性hash算法

参看：http://blog.csdn.net/cywosp/article/details/23397179

#### 11.一个网站有很大的访问量，有什么办法来解决？

主要从架构层面解决：

	1. 使用服务器集群，如tomcat集群；
	2. 使用缓存，如memcache，redias，cassandra分布式缓存；
	3. 使用数据库集群，如mysql集群，oracle数据库集群。

通过分布式解决 访问量、数据量大的问题。

## 网络相关

#### 1.DNS查询过程？

工具软件dig可以显示整个查询过程：

 - 向本机的DNS服务器发送DNS请求
 - DNS服务器对域名地址进行分级查询：主机名.次级域名.顶级域名.根域名
 - DNS服务器首先通过“根域名服务器(世界上一共有十三组)”查到“顶级域名服务器”的NS记录和A记录
 - 再通过“顶级域名服务器”查到“次级域名服务器”的NS记录和A记录
 - 再通过“次级域名服务器”查出“主机名”的IP地址

每一级的查询都是发出一批UDP请求，比如查询“顶级域名服务器”时，同时向13个"根域名服务器"发送DNS请求

参见：http://www.ruanyifeng.com/blog/2016/06/dns.html

#### 2.HTTP1.1和HTTP1.0的区别？2.0？

 - 持久连接：一个tcp连接上可以传送多个请求，每个单独网页仍然使用各自的连接。客户端不用等待上次请求结果返回就可以发送下一次请求，但服务端必须按照接收到客户端请求的顺序依次回送响应结构，以保证客户端能区分响应内容
 - host域：使用虚拟主机技术，这样可以区分一台主机上的多个应用
 - Transfer Codings：分块传输，以0长度的块结尾。避免缓存所有的数据来计算Content-Length值
 - Range和Content-Range：允许请求资源的某个部分，偏移值+长度
 - Status 100：增加状态码100（Continue），解决包含大量内容的请求被服务端拒绝访问时，浪费的带宽。客户端在发送request body之前，先用request header试探一下server，看server要不要接收request body，再决定要不要发request body。401（Unauthorized）、100（Continue）
 - Cache：Http会使用Expire头域来判断资源的fresh或stale，并使用条件请求来判断资源是否仍然有效。1.1增加了超时机制，重新判断资源有效性。
 增加了一些请求方法

参考：http://blog.csdn.net/hguisu/article/details/8608888

HTTP的头部字段解析：http://www.cnblogs.com/xcsn/p/4308228.html

#### 3.HTTP的状态码含义？

1.信息，2.成功，3.重定向，4.客户端错误，5.服务器端错误

参考：http://www.w3school.com.cn/tags/html_ref_httpmessages.asp

#### 4.TCP、UDP和IP报文的格式？

http://blog.csdn.net/kernel_jim_wu/article/details/7447377

#### 5.TCP的慢启动、快重传？

tcp有一个接收窗口用于流量控制，窗口为1是停等协议，效率低下，如果窗口太大，那么可能会出现网络拥塞。所以拥塞控制依赖于拥塞窗口（cwnd）来进行控制。慢启动有点类似于车辆启动的时候，具体待阐述，参考：http://blog.csdn.net/loverooney/article/details/38323907

#### 6.多播协议、多播协议的应用场景？

一点对多点，节省网络带宽；IP多播广泛应用在网络音频、视频中。依赖IP多播地址，是D类地址。

#### 7.TCP连接三次握手、断开四次挥手的过程？

三次握手：客户端发起请求（序列号）-服务器回应ack并给出自己的序列号-客户端回应ack。

断开四次挥手：一端发起断开（FIN），另外一端回应ack；这样一端的写就关闭了；另外一端发起断开请求（FIN），本端回应ack。这样双方都关闭了连接。因为一端关闭了写，但是另外一端可能还有数据要发送，所以两端可能不同时关掉连接，就出现了4次挥手的过程。

time_wait状态：tcp 4次挥手的过程中，主动关闭连接的一方，在收到对方的FIN包后会进入time_wait状态(2*MSL的时间），如果在这个时间段内，没有再次收到对面的FIN包，就表示ack已经成功到达。假想，如果没有这个状态，而是直接就是close状态的话，对面没有收到ack，重发FIN包，这边会发送RST包，导致server端收到rst包报错，影响程序进程； 并且假设没有time_wait状态，新的连接请求过来，老的关闭报文就会对新的连接产生干扰

参看：http://www.firefoxbug.com/index.php/archives/2795/

#### 8.ping的实现？

Ping类似于一个回声系统，发送一个请求、对端如果收到回应一个报文就可以判断了。这里采用ICMP（网际控制报文协议）来实现。发送一个ICMP的回送请求报文，对端如果收到，回应一个ICMP的回送请求和回送应答报文。利用的是IP的点对点的协议，并没有端到端，不需要端口这些，使用的是原始套接字(RAW)

具体实现可以参考：http://blog.csdn.net/zzucsliang/article/details/41407387

#### 9.close和shutdown

shutdown是优雅的关闭，会让所有数据发送完

参考：http://blog.liyiwei.cn/socket-%E7%BC%96%E7%A8%8B%E4%B8%ADclose%E5%92%8Cshutdown%E7%9A%84%E5%8C%BA%E5%88%AB/

## 操作系统相关

#### 1.阐述一下系统调用？

 - Linux系统调用通过中断来实现
 - 系统调用使用中断号0x80
 - 系统调用号  存入寄存器%eax，通过系统调用表查得系统调用入口地址
 - 系统调用参数 存入%ebx，%ecx，%edx，%esi，%edi和%ebp这6个寄存器
 - 从TSS段中的sp0获取进程的内核栈的栈顶指针（每个进程都有一个内核栈）
 - 由控制单元将当前eflags, cs, ss, eip, esp寄存器的值存入内核栈
 - 把内核代码选择符写入cs寄存器，内核栈指针写入esp寄存器，内核入口点的线性地址写入eip寄存器
 - 这样就完成了内核态的切换。回到用户态之前也要恢复相关寄存器的值

可以看出系统调用时间花费主要在：1.查询中断向量表、系统调用表 得到系统调用函数 2.保存和恢复寄存器值

参看：http://blog.csdn.net/chosen0ne/article/details/7721550

#### 2.进程与线程的区别？

 - 进程是资源分配的单位，线程时任务调度的单位
 - 进程有相应的内存空间，线程共享进程的内存空间
 - 单线程遇到阻塞会卡死，影响交互
 - 多线程能充分发挥多核心的计算能力
 - 简化程序结构，使程序便于维护
 - 与进程相比，线程的创建和切换开销更小

 参看：http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html

#### 3.Linux下面的进程间通信方式？

 - 管道
 - 命名管道
 - 信号
 - 信号量集
 - 消息队列：结构化数据、数据量大
 - 共享内存：快，和互斥量或文件锁结合达到进程间同步和互斥
 - socket：不同主机间的进程进行通信。socket域套接字也可用于同一主机内不同进程通信，netstat就能看到好多

 参看：http://www.cppblog.com/jerryma/archive/2011/08/03/152348.aspx

#### 4.线程之间同步方式？

 - 互斥锁
 - 读写锁
 - 条件变量
 - 屏障
 - 记录锁

#### 5.孤儿进程？

父进程先于子进程退出，则子进程成为孤儿进程，1号进程init作为其养父进程。init会定期检查子进程的状态，清除僵尸进程，从而避免系统内充斥大量的僵尸进程，占用进程号。

#### 6.内存分配策略？

伙伴系统

#### 7.交换空间的作用？

`cat /proc/sys/vm/swappiness` 指定一个比例，比例越大，越积极使用交换空间。操作系统根据空闲内存量，及swappiness值决定使用交换空间

#### 8.Linux缓冲区设计？

LRU具体实现，

#### 9.重点部分

文件系统、进程管理、进程切换、内存管理这几个部分，其中文件管理和内存管理尤为重要。包括 vfs 虚拟文件系统，一个完整的文件操作过程比如 read 、 write 等等，文件映射 mmap ，共享内存等等可能需要花一点时间理解这些东西。

## Linux shell命令笔记

#### 1.linux下查看文件第几行内容的方法？

```
.输出一个文件的第4行
sed -n '4p' ufile
awk 'NR==4' ufile
head -4 file|tail -1
```

#### 2.一个端口的TCP进程结束了，一段时间之内，该端口还会不会被分配？

除非指定端口，否则不会。端口是+1指定的

#### 3.服务器端经常用到的shell命令？

- free -m              (查看内存使用情况）
- fdisk -l             (查看硬盘分区信息）
- du -sh /bin/         (查看目录的磁盘占用）
- iostat               (查看硬盘的IO性能）
- uptime               (查看服务器的平均负载）
- vmstat               (查看服务器的整体性能）
- netstat              (统计与网络相关的参数）

参考：http://blog.jobbole.com/15430/

http://www.cnblogs.com/linzhenjie/archive/2013/01/14/2859085.html

#### 4.找出当前目录下文件最大的前十个？

du -s * | sort -nr | head

#### 5.查找指定后缀文件并且这些文件中包含特定的字符串的文件？

find . -name “\*.c” | xargs grep -H “hello”

#### 6.找出文件中指定字符串的前后5行？

grep -i “main” -C 3 sprintf.c

主要有 grep 、 awk 、 sed 、 ss 、 top 、 find 等等还有一些性能调优的命令


## 问题

#### Tomcat的session和cookie对比？

1. cookie保存在客户端、session保存在服务器
2. cookie可以设置有效期，保存在客户端的磁盘文件里，或者不设置有效期只在该浏览器窗口有效；session默认20分钟不使用就清除
3. session使用jsessionid标示，可以通过url或者cookie传输jsessionid；因此别的客户端拿到jsessionid也是可以访问session的，这样就出现了session劫持
4. Tomcat使用ConcurrentHashMap存放session
5. 使用Tomcat集群的话就要考虑session一致性的问题，可以使用同一台主机存放session

参看：http://www.cnblogs.com/chenpi/p/5434537.html

#### 如何做负载均衡？

#### 如何处理高并发？对一个表的高并发写入？分布式的Mysql如何保证一致性？

#### HashMap并发写有啥问题？

#### int和Integer的区别？String的比较？

#### 了解Tomcat和nginx？

Linux 内核的知识、Java 基础知识也差不多了，自己丢个 blog也没人看，还是转到 osc吧。最近在看 osc上的两个开源 Java Web框架：jfinal、smart-framework，还见识了叼叼的 Angularjs、shiro。围绕这 Web框架，自己写点东西出来先。

osc 地址：https://my.oschina.net/u/3026036/blog
