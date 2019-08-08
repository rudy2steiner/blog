---
title: JDK 7、8及9中虚拟机的变化
tags:
  - JDK
  - 翻译
date: 2019-08-09 00:34:52
---


{% asset_image jdk_changes.png %}
感谢Kunal Saxena

## Java 7的变化

PermGen 区中移除了如下部分:

* 符号(Symbols)被移到本地内存
* Interned字符串被移到堆内存
* 类的静态部分被移到堆内存  

## Java 8的变化

* 永久代(PermGen)被元空间取代(Metaspace)
* 类及元数据被移到元空间
* 元空间从本地内存分配，也即是说它可以使用系统的所有剩余内存，并且它与堆内存不连续
* 虚拟机的选项可以限制元空间的最大值
* G1 开始支持并行地卸载类
* 引入代码缓存用于存储及时编译器(Just-In-Time,JIT)编译过的代码
* 引入压缩类空间

## Java 9的变化

* G1作为默认的垃圾回收器
* 并发标记清除垃圾回收器计划被移除
* 还删除了一些用于年轻代和老年代的垃圾回收器组合

原文[Java Virtual Machine Changes in Java 7,8 and 9](https://www.linkedin.com/pulse/java-virtual-machine-changes-78-9-kunal-saxena)
　
