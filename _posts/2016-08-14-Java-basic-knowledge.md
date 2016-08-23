---
layout: post
title: Java基础知识总结
categories: [Java ]
tags: [Java, ]
description: Java基础知识总结，面试知识点整理
---

## Java基础

1.八种基本数据类型、大小、封装类

byte 1\short 2\int 4\long 8\float 4\double 8\boolean 1\char 2

2.Switch能否用string做参数？

jdk7开始支持，底层使用str.hashCode()比较；byte\short\char\int\enum\String

3.equals()与==的区别？

4.Object有哪些公用方法？

wait\notify\equals\hashCode\toString

5.Java的四种引用，强弱软虚，用到的场景？

强引用 声明的对象都是强引用的，绝不会被GC回收；软引用 用SoftReference声明，一般用于Cache，内存不足时才会考虑回收；弱引用 用WeakReference声明，常用于Debug、内存监视程序，不影响正常GC；虚引用跟没有引用一样

6.ArrayList、LinkedList、Vector的区别

链表实现方式和线程安全

7.String、StringBuffer、StrinbBuilder的区别

String是final修饰的不可变对象；StringBuffer是线程安全的

8.Map、Set、List、Queue、Stack的特点与用法

9.Collection包结构，Arrays与Collections的区别？

Arrays都是对数组的操作，排序、比较、查找、填充、复制分割等；Collections都是对List Map的操作，排序、查找、反转、交换、填充、复制、最大最小值、得到同步结构等

10.Excption与Error包结构。OOM你遇到过哪些情况，SOF你遇到过哪些情况？

Throwable包含两个子类: Error 和 Exception；OOM内存不足，内存泄露或内存溢出，调整JVM内存块大小Xmx,Xms；SOF栈溢出，调整栈的允许大小Xss；常量池溢出、方法区溢出

11.Java面向对象的三大特征及含义

12.Java多态的实现原理？

强制多态、重载的多态、参数的多态、包含的多态；可以单独开篇文章了[【解惑】Java动态绑定机制的内幕](http://hxraid.iteye.com/blog/428891);[jvm specification ](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-5.html)

13.HashMap遍历中删除？

keySet、entrySet，只有entrySet.iterator可以，原因是只有该删除方法同步乐modCount=exceptedModCount；同样的遍历中也不能增加，因为增加只有put方法，之后的nextEntry()会报错；同样的List也只能通过Iterator才能正确remove。设置modCount和expectedModCount的目的是为了检查iterator的有效性，keySet和hashMap的remove可以删除任意节点，而iterator只能删除当前节点

14.try里return，finally再return，哪个会返回，字节码什么样的？

返回finally里的，编译为字节码时，直接忽略了try和catch里的返回，使用的finally里的返回

15.方法锁、对象锁、类锁

16.ThreadLocal的设计理念与作用？

aupe说，1有时候需要维护per-thread的数据 2他提供了让基于进程的接口适应多线程的机制

对Java来说就是1的作用吧，比如数据库线程池，线程类里存在有状态的全局变量session，可以放到ThreadLocal里，来实现线程安全的访问；Spring也会对一些Bean（LocaleContextHolder，RequestContextHolder）中非线程安全状态这样处理

17.wait()和sleep()的区别？

 - wait用于线程同步，是Object的方法，需要notify唤醒；sleep是Thread的方法，等待超时或者interrupt
 - wait必须用于synchronized块内，否则抛出IllegalMonitorStateException，会释放锁；sleep可以用于任何地方
 - wait也可加等待超时时间参数，wait notify需要在获得锁后调用；

18.ThreadPool用法与优势？

降低资源消耗、提高响应速度、提高线程可管理性

19.线程同步的方法：synchronized、lock、reentrantLock、readWriteLock等

20.写出生产者消费者模式

用个LinkedBlockingQueue就可以

21.Concurrent包里的其他东西：BlockingQueue、CountDownLatch、Barries等等

22.Java IO与NIO？

nio面向缓存、可以设置非阻塞、使用selector多路选择

23.JDK 1.7与1.8新特性？

[新特性](http://blog.chinaunix.net/uid-29618857-id-4416835.html)

24.设计模式：单例、工厂、适配器、责任链、观察者、策略等等。

10.数据库事务并发问题

多个事务同时访问数据库时，会发生如下5类问题：3类数据读问题（脏读、不可重复读、幻读）2类数据更新问题（第一类丢失更新、第二类丢失更新）
 1. 脏读（dirty read）：A事务读取B事务尚未提交的更改数据并在这个基础上操作。如果B事务回滚，那么A事务读到的数据根本不是合法的，成为脏读。
 2. 不可重复读（unrepeatable read）:A事务读取B事务已经提交的更改（或删除）数据时，在B事务提交前后A事务读取的结果不一致。———加行级锁，数据库默认
 3. 幻读（phantom read）：B事务新增一条记录的情况下，A事务在B事务提交前后读取到的数据条数不一致。——加表锁，开销太大，一般都不加
 4. 第一类丢失更新：A事务撤销时，把B事务提交的数据修改覆盖掉。
 5. 第二类丢失更新：A事务提交时，把B事务提交的数据修改覆盖掉。

11.数据库的悲观锁和乐观锁

 - 悲观锁：假设数据肯定会冲突，所以在数据开始读取的时候就把数据锁住。
 - 乐观锁：认为数据一般不会造成冲突，所以在数据提交时才会检查数据是否冲突，如果发生冲突则返回错误信息，让用户决定如何去做。
 - 乐观锁的三种实现方式：（都是CAS操作）
	1. 整个数据copy到应用中，提交时对比当前数据的数据和copy的数据是否一致，由于double型不可比，所以一般不用。
	2. 采用版本戳，数据表增加一个新的column，比如是number型的，每次更新时都加一，通过对比number来判断一致性。
	3. 采用时间戳，跟第2中类似，不过数据类型是timestamp型的。

12.Innodb默认使用乐观锁还是悲观锁

默认使用乐观锁，MVCC（多版本并发控制）来处理不可重复读和幻读。Innodb会在每行数据后添加两个额外的隐藏值来实现MVCC。（参见http://tech.meituan.com/innodb-lock.html）


tomcat使用线程池、异步io来监听客户端socket请求：

```
TaskThread.run();
ThreadPoolExecutor.runWorker();
NioEndpoint.SocketProcessor.run();
Http11NioProtocal.Http11ConnectionHandler
Http11NioProcessor.process();
tomcat根据web.xml配置的filter进行字符过滤：
EXTCharacterEncodingFilter.doFilterInternal();
tomcat根据web.xml配置的servlet进行请求处理：
DispatcherServlet.service();
DispatcherServlet.doDispatch();
spring-mvc根据配置的拦截器interceptors做权限拦截：
SecurityInterceptor.preHandler();
```

15.一个网站有很大的访问量，有什么办法来解决？

主要从架构层面解决：
	1. 使用服务器集群，如tomcat集群；
	2. 使用缓存，如memcache，redias，cassandra分布式缓存；
	3. 使用数据库集群，如mysql集群，oracle数据库集群。
通过分布式解决 访问量、数据量大的问题。

参考：

* [JDK8 新特性](http://blog.chinaunix.net/uid-29618857-id-4416835.html)
* [面试心得与总结---BAT、网易、蘑菇街](http://sanwen8.cn/p/1fcgrN7.html)
* [Java研发方向如何准备BAT技术面试](http://www.jianshu.com/p/05f42258850b)