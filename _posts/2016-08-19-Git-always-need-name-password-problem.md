---
layout: post
title: push到github时每次都要输入用户名和密码的问题
categories: [Git ]
tags: [Git, ]
description: push到github时每次都要输入用户名和密码的问题
---

## 问题

在github上建立一个小项目，可是在每次push的时候，都要输入用户名和密码，很是麻烦

## 原因

> 关键是你用了https，它不走ssh通道，所以key 都没用了，https 保存密码[可以参考][1]，如果要ssh+key 无需密码提交，远程分支需要是ssh协议的 git@github.com:xx/xx.git

## 解决

1.查看远端的连接方式

```shell
git remote -v
;; 查看当前的远端连接
;; 输出如下：
;; origin	git@github.com:lan-cyl/lan-cyl.github.io.git (fetch)
;; origin	git@github.com:lan-cyl/lan-cyl.github.io.git (push)
```

2.移除旧的origin

```shell
git remote rm origin
```

3.添加新的ssh方式的origin

```shell
git remote add origin git@github.com:lan-cyl/lan-cyl.github.io.git
```

4.提交下试试

```shell
git push origin master
```

参考：

* [Https方式使用Git@OSC设置密码的方式][1]

* [push到github时，每次都要输入用户名和密码的问题][2]

* [GIT每次都要输入密码][3]

[1]: http://git.oschina.net/oschina/git-osc/issues/2586 "Https方式使用Git@OSC设置密码的方式"
[2]: http://blog.csdn.net/yuquan0821/article/details/8210944 "push到github时，每次都要输入用户名和密码的问题"
[3]: https://segmentfault.com/q/1010000000607964 "GIT每次都要输入密码"
