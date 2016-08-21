---
layout:
title: Linux shell命令笔记
categories: [linux ]
tags: [linux, ]
description: Linux常见问题记录
---

1.linux下查看文件第几行内容的方法?

```
.输出一个文件的第4行
sed -n '4p' ufile
awk 'NR==4' ufile
head -4 file|tail -1
```

2.一个端口的TCP进程结束了，一段时间之内，该端口还会不会被分配？

除非指定端口，否则不会。端口是+1指定的
