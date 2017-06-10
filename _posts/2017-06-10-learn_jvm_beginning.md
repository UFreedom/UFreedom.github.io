---
layout: post
title: JVM 学习笔记 - 初识 Java 虚拟机
comments: true
category: JVM
tags: [JVM]
---

何为虚拟机，说白了就是工作在 PC 或者移动手机操作系统之上的一款软件，有的虚拟机能完整的虚拟某个操作系统的环境比如 VMWare,Parallels Desktop，让你能在 Mac 系统上使用 Windows，Windows 系统里面使用 Linux。而有的虚拟机呢，则用来解释执行某个计算机程序，让你无关它的底层实现，你只需要关注上层如何使用它提供的编程套件就好，正所谓：一次编译，到处运行，比如 Java 虚拟机。

Java 之所以能做到跨平台执行，主要的功劳归于 Java 虚拟机:

<div align="center">
<img src="/attachments/images/learn_jvm/learn_jvm_chapter_1.png" />
 </div>


Java 以虚拟机为中介，横行在各大操作系统平台，比如 Windows, Linux, Android等等，Java 虚拟机可以看做一台抽象的计算机，如同真实的计算机一样，它有自己的指令集，以及各种内存区域。

如果单纯的说 Java 虚拟机这个概念，它只是一种抽象的规范，怎么实现我不管，反正你要能虚拟运行环境，比如说 JRockit, Hotpot等，目前 来说 Hotpot 占绝对的市场地位。

除了 Hotpot 之外，作为 Android 开发者，我们感觉最亲切的当然是 Davlik 和 ART 虚拟机，它们也是 Java 虚拟机，但是因为它们要服务于移动设备平台，移动设备的内存和处理速度有限，所以 Android 专门做了优化。

作为 Android 开发者来说，了解虚拟机非常有必要，特别是类加载机制和垃圾回收机制。现在市面上已经有很多分享虚拟机的书籍和资料，我将跟随前辈的脚步，一窥虚拟机的世界。

参考书籍：
《深入理解 Java 虚拟机：JVM 高级特性与最佳实践》，周志明
《Java 虚拟机规范,Java SE 8 版》，Tim Lindholm, Frank Yellin, Gilad Bracha, Alex Buckley
《实战 JVM 虚拟机: JVM 故障诊断与性能优化》，葛一鸣
《深入 Java 虚拟机》，Bill Venners
