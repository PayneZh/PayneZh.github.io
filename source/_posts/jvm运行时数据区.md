---
title: jvm运行时数据区
date: 2020-09-22 05:12:24
tags: java基础
categories: 编程
---
# 概述
内存是非常重要的系统资源，是硬盘和CPU的中间仓库及桥梁，承载着操作系统和应用程序的实时运行。JVM内存布局规定了Java在运行过程中内存申请、分配、管理的策略，保证了JVM的高效稳定运行。
不同的JVM对于内存的划分方式和管理机制存在着部分差异。结合JVM虚拟机规范，来探讨一下经典的JVM内存布局。
如下所示：![图1](https://github.com/PayneZh/MarkDownPhotos/raw/master/res/%E8%BF%90%E8%A1%8C%E6%97%B6%E5%AD%90%E7%B3%BB%E7%BB%9F.jpg)
![图2](https://github.com/PayneZh/MarkDownPhotos/raw/master/res/%E8%BF%90%E8%A1%8C%E6%97%B6%E5%AD%90%E7%B3%BB%E7%BB%9F1.jpg)
Java虚拟机定义了若干种程序运行期间会使用的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则是与线程一一对应的，这些与线程对应的数据区域会随着线程开始和结束而创建和销毁
即每个线程：独立包括程序计数器，栈，本地栈。线程间共享：堆，堆外内存（永久代或元空间，代码缓存）
注：每个JVM只有一个Runtime实例，即为运行时环境，相当于内存结构的中间的那个框框：运行时环境
![图3](https://github.com/PayneZh/MarkDownPhotos/raw/master/res/Class%20Runtime.jpg)

