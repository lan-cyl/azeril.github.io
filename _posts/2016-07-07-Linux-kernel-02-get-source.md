---
layout: post
title: Linux内核分析（二）源码介绍
categories: [linux kernel ]
tags: [linux kernel, ]
description: linux 内核源码介绍
---

## 获取源码

可以在Linux Kernel的官网下载 [https://www.kernel.org/](https://www.kernel.org/).

也可以从Github fork一个 `git clone https://github.com/torvalds/linux.git`

也可以从包管理器里下载，比如Ubuntu下 `sudo apt install linux-source-4.4.0`, 然后解压缩 '/usr/src/linux-source-4.4.0.tar.bz2'

## 内核源码的结构

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

## Documentataion 文件夹

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

## 参考：

* Understanding the linux kernel 3E
* Linux kernel development 3E
* Advanced programing in the UNIX environment 3E
