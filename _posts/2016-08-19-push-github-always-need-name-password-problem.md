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

> 原因是使用了https方式push

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
