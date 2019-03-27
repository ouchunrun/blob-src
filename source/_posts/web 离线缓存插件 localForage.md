---
title: web 离线缓存插件 localForage
date: 2018-3-19
tags: [JS, localForage] 
---

localStorage 能够让你实现基本的数据存储，但它的速度慢，而且不能处理二进制数据。IndexedDB 和 WebSQL 是异步的，速度快，支持大数据集，但他们的API 使用起来有点复杂。不仅如此，IndexedDB 和 WebSQL 没有被所有的主流的浏览器厂商支持，这种情况最近也不太可能改变。

Mozilla 开发了一个叫 localForage 的库 ，使得离线数据存储在任何浏览器都是一项容易的任务。

localForage 是一个使用非常简单的  库的，提供了 get，set，remove，clear 和 length 等等 API，还具有以下特点：

<!--more-->

- 支持回调的异步 API；
- 支持 IndexedDB，WebSQL 和 localStorage 三种存储模式（自动为你加载最佳的驱动程序）；
- 支持 BLOB 和任意类型的数据，让您可以存储图片，文件等等。
- 支持 ES6 Promises；
- 对 IndexedDB 和 WebSQL 的支持使您可以为您的  应用程序存储更多的数据，要比 localStorage 允许存储的多很多。其 API 的无阻塞性质使得您的应用程序更快，不会因为 Get/Set 调用而挂起主线程。






## 参考

[localForage——轻松实现 Web 离线存储](http://www.cnblogs.com/lhb25/p/localforage-offline-storage-improved.html)


