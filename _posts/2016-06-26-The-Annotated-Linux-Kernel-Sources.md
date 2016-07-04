---
layout: post
title: The annotated linux kernel sources(1) -- get the kernel
categories: [blog ]
tags: [Jekyll, Blog, ]
description: linux 内核源码剖析第一篇，获取内核源码
---


## obtain a copy

Linux kernel sources tree can be find in [github](https://github.com/torvalds/linux).

I fork this repository (forking a long time, really a big program), and clone it to my locale pc. `git clone https://github.com/torvalds/linux.git`

## the kernel source tree

*Table 1 Directories in the root of the kernel source tree*

| Directory      | Description                  |
|:---------------|:-----------------------------|
| Documentataion | 内核源码的相关文档 |
| arch           | 跟硬件架构相关的代码  |
| block          | 块I/O层代码 |
| certs          | ..., i don't know |
| crypto         | 加密相关的API |
| drivers        | 设备驱动代码 |
| fireware       | 设备驱动之上的防火墙代码 |
| fs             | 虚拟文件系统(VFS)和各个独立的文件系统 |
| include        | 内核代码的头文件 |
| init           | 内核启动和初始化代码 |
| ipc            | 进程间通信代码 |
| kernel         | 核心子系统代码，例如scheduler |
| lib            |  |
| mm             | 内存管理子系统和虚拟内存代码 |
| net            | 网络子系统 |
| samples        | 示例，示范代码 |
| scripts        | 编译内核所用的脚本 |
| security       | Linux的安全模块 |
| sound          | 声音子系统 |
| tools          | Linux开发中的实用工具 |
| usr            | 早期用户空间使用的文件系统 |
| virt           | 内核虚拟机驱动，该模块使Intel VT-x直接运行虚拟机，无需模拟器或者二进制转换 |

*Table 2 Files in the root of the kernel source tree*

| File           | Description                  |
|:---------------|:-----------------------------|
| COPYING        | 许可证(GNU GPL v2) |
| CREDITS        | 开发了很多内核代码的开发人员列表  |
| Kbuild         |  |
| Kconfig        |  |
| MAINTAINERS    | 维护内核子系统和驱动的人员列表，提交内核变更的方法 |
| Makefile       | 编译配置文件 |
| README         | 项目说明文档，配置编译内核的方法 |
| REPORTING-BUGS | 提交BUG的注意事项 |

## Documentataion folder about

| File           | Description                  |
|:---------------|:-----------------------------|
| HOWTO          | 许可证(GNU GPL v2) |
| CREDITS        | 开发了很多内核代码的开发人员列表  |
| Kbuild         |  |
| Kconfig        |  |
| MAINTAINERS    | 维护内核子系统和驱动的人员列表，提交内核变更的方法 |
| Makefile       | 编译配置文件 |
| README         | 项目说明文档，配置编译内核的方法 |
| REPORTING-BUGS | 提交BUG的注意事项 |

{% highlight java %}
public class Main {
  public static void main(String[] args) {
    System.out.println("Hello World!");
  }
}
{% endhighlight %}

## 参考
* [打造你的 GitHub Pages 专属博客](http://azeril.me/blog/Build-Your-First-GitHub-Pages-Blog.html)
* [Jekyll 博客主题精选](http://azeril.me/blog/Selected-Collection-of-Jekyll-Themes.html)
* [Git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)  
* [Jekyll 语法简单笔记](http://github.tiankonguse.com/blog/2014/11/10/jekyll-study.html)
* [Jekyll/Liquid API 语法文档](http://alfred-sun.github.io/blog/2015/01/10/jekyll-liquid-syntax-documentation/)
