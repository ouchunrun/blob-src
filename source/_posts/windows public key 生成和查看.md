---
title: windows public key 生成和查看
date: 2021-6-20
tags: [SSH key] [windows]
---

## 生成key

用git bash的终端，输入ssh-keygen，一路回车：

> ssh-keygen

<!--more-->

----

## 查看

打开git bash 终端

输入：

> cd ～


输入：
> cd .ssh

输入ls，你就看到很多文件，id_rsa.pub：
>ls

输入：

>vi id_rsa.pub

就能够看到key了！

----

## FAQ

1、解决：`no matching key exchange method found. Their offer: diffie-hellman-group1-sha1`

> vi  ~/.ssh/config

添加：
```
Host *
KexAlgorithms +diffie-hellman-group1-sha1
```
此方法只对当前用户生效，使用其他用户是又会报错。

- 参考：https://blog.csdn.net/u011983700/article/details/81738657
