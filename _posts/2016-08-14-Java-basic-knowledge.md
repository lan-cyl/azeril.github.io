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

7.String、StringBuffer、StringBuilder的区别

String是final修饰的不可变对象；StringBuffer是线程安全的

8.Map、Set、List、Queue、Stack的特点与用法

9.Collection包结构，Arrays与Collections的区别？

Arrays都是对数组的操作，排序、比较、查找、填充、复制分割等；Collections都是对List Map的操作，排序、查找、反转、交换、填充、复制、最大最小值、得到同步结构等

10.Excption与Error包结构。OOM你遇到过哪些情况，SOF你遇到过哪些情况？

Throwable包含两个子类: Error 和 Exception；OOM内存不足，内存泄露或内存溢出，调整JVM内存块大小Xmx,Xms；SOF栈溢出，调整栈的允许大小Xss；常量池溢出、方法区溢出

11.Java面向对象的三大特征及含义

12.Java多态的实现原理？

看27条

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

如何正确停止一个线程？

* 1. 使用退出标志，线程检查到退出标志时，正常return
* 2. 使用stop方法强行退出，废弃方法不推荐
* 3. 使用interrupt方法中断线程

参见：http://www.cnblogs.com/greta/p/5624839.html

18.ThreadPool用法与优势？

降低资源消耗、提高响应速度、提高线程可管理性

线程池原理？

线程池中的线程数少于coreThreadPoolSize时，创建新的线程来执行任务；多于coreThreadPoolSize时，便将任务放入缓存队列；当缓存队列满时，创建新的线程来处理；当线程数大于maxThreadPoolSize时，拒绝服务

线程池的实现类是ThreadPoolExecutor->AbstractExecutorServer->ExecutorServer->Executor，当时Jdk封装了Executors工具类，提供了几个工厂方法，用户生成不同类型的ThreadPoolExecutor线程池。

参看：http://www.importnew.com/19011.html

19.线程同步的方法：synchronized、lock、reentrantLock、ReentrantReadWriteLock等

_ReentrantLock源码，使用BlockingQueue作为模板方法模式_

参看：http://www.raychase.net/1912  
参看：http://www.cnblogs.com/dolphin0520/p/3923167.html

20.写出生产者消费者模式

用个LinkedBlockingQueue就可以

21.Concurrent包里的其他东西：BlockingQueue、CountDownLatch、CycleBarries等等

22.Java IO与NIO？

nio面向缓存、可以设置非阻塞、使用selector多路选择

aio将实际的读写操作交给操作系统，等待读写完毕再通知进程

23.JDK 1.7与1.8新特性？

[新特性](http://blog.chinaunix.net/uid-29618857-id-4416835.html)

阻塞、非阻塞，同步、异步的区别：http://blog.csdn.net/hguisu/article/details/7453390

24.设计模式：单例、工厂、适配器、责任链、观察者、策略、代理模式等等。

25.虚拟机类加载过程

   加载   从各个地方获取输入流
-> 验证   文件格式？元数据 字段\方法是否于父类冲突？字节码里的数据流 控制流合法？符号引用合法？，要先加载父类
-> 准备   为类变量分配内存(方法区)，初始为0值。常量初始化对应值
-> 解析   常量池中的符号引用替换为直接引用，仅针对invokestatic和invokespecial指令引用的非虚函数（静态、私有、构造、父类方法）
          在运行时常量池中记录直接引用，并把对应常量标示为已解析状态。invokedynamic指令不受解析状态影响，实际运行到这条指令时才动态解析，叫动态链接
          类与接口的解析：如果是数组先加载数组元素类型，否则直接加载元素类型，最后判断元素类型的访问权限
          字段解析：先加载所属的类或接口，判断是否包含该字段，否则自下而上的搜索各个接口，还没有的话自下而上的搜索父类字段，最后也是权限验证。虚拟机实现的更加严格，如果字段同时出现在接口和父类中、或者自己或父类的多个接口中，则拒绝编译
          类方法解析：先加载所属的类或接口，如果是个接口就抛出错误。在类中查找匹配的方法，没有就递归的去父类里找，还没有就去接口里找抛出AbstractMethodError异常，都没有就抛出NoSuchMethodError。最后验证权限。
          接口方法解析！！！接口方法，有个卵用？？？难道不是指向实现类的方法
-> 初始化 编译器将类变量的赋值语句，静态代码块 按照源代码中的顺序自动生产<clinit>方法；父类的<clinit>先执行。

26.类加载器

通过一个类的全限定名来获取描述此类的二进制字节流。双亲委派模型，先交给父类加载器加载

27.虚拟机字节码执行引擎？

解析：就是类加载过程中的解析阶段，将非虚方法的符号引用替换为直接引用，非虚方法在运行期是不会改变的

静态分派：所有依赖静态类型来定位方法执行版本的分派动作，一般用于方法重载(Overload)。发生在编译阶段

动态分派：跟重写(Override)紧密关联。invokevirtual指令需要两个操作数，一个取自操作数栈顶的对象指针，一个是指令后面跟的符号引用。1.找到对象的实际类型 2.在类型中找到与符号引用的描述符和简单名称都相符的方法，判断权限并返回这个方法的直接引用，结束 3.在父类中进行搜索和验证

动态分派的实现：方法区建立虚方法表，存放各个方法的实际入口地址，具体可看《深入理解JVM》的P257的 图8-3 方法表结构

28.


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

## 参考：

* [JDK8 新特性](http://blog.chinaunix.net/uid-29618857-id-4416835.html)
* [面试心得与总结---BAT、网易、蘑菇街](http://sanwen8.cn/p/1fcgrN7.html)
* [Java研发方向如何准备BAT技术面试](http://www.jianshu.com/p/05f42258850b)
* [面试总结 -- ImportNew](http://www.importnew.com/21445.html#comment-509248)
