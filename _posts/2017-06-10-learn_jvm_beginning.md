---
layout: post
title: JVM 学习笔记 - 初识 Java 虚拟机
comments: true
category: JVM
tags: [JVM]
---
## 初识 Java 虚拟机

### 何为虚拟机

何为虚拟机，说白了就是工作在 PC 或者移动手机操作系统之上的一款软件，有的虚拟机能完整的虚拟某个操作系统的环境比如 VMWare,Parallels Desktop，让你能在 Mac 系统上使用 Windows，Windows 系统里面使用 Linux。而有的虚拟机呢，则用来解释执行某个计算机程序，让你无关它的底层实现，你只需要关注上层如何使用它提供的编程套件就好，正所谓：一次编译，到处运行，比如 Java 虚拟机。

<div align="center">
<img src="/attachments/images/learn_jvm/learn_jvm_chapter_1.png"  width="350"  height="282"/>
 </div>
<br>

### 平台无关性的 class 文件

Java 之所以能做到跨平台执行，主要的功劳归于 Java 虚拟机，当我们编写 Java 程序时，通过调用 JDK 提供的API来实现我们的需求，当编译程序时，Java 编译器将 Java 文件翻译为 class 字节码文件，class 文件是一个平台无关性文件，它不拘束于某个特定平台，也就是说，只要有虚拟机，你在 Windows 系统编译后的 class，拿到 Linux 环境照样可以执行。

<div align="center">
<img src="/attachments/images/learn_jvm/learn_jvm_chapter_1-2.png" width="350"  height="275" />
 </div>
<br>

Java 虚拟机的主要任务就是执行 class 文件的字节码，让 class 文件变成一个平台无关性的文件。但是问题是各个平台的底层API还是不同的，比如你用 Java IO API 编写文件读写API，在 Windows 和 Linux 底层实现文件的读写机制是不同的。这就要求 Java 虚拟机能针对不同的平台去做适配。

### 类装载器与执行引擎

Java 有两种方法：Java 方法和本地方法，Java 方法由 Java 编写，会被编译成字节码存储在 class 文件中，当 Java 虚拟机运行时，先使用类装载器（Class Loader）将 class 文件装载到虚拟机内。而本地方法和平台有关，使用其他语言开发，比如C，C++，编写，被编译成和处理器相关的机器代码。本地方法保存在动态链接库中，当然根据不同的平台会有不同的格式。当程序运行时，Java 虚拟机的执行引擎会扮演调度者的角色，调用和平台相关的本地方法。

<div align="center">
<img src="/attachments/images/learn_jvm/learn_jvm_chapter_1-3.png" width="220"  height="397"/>
 </div>
<br>

### Java 体系结构
Java 虚拟机在运行 Java 程序时，使用类装载器装载 class 文件，然后执行引擎使用本地方法连接底层操作系统，这个过程需要存储结构的支持，这部分被称为运行时数据区，运行时数据区又被细划分为：方法区，堆，虚拟机栈，本地方法栈，程序计数器。

<div align="center">
<img src="/attachments/images/learn_jvm/learn_jvm_chapter_1-4.png" width="350"  height="221"/>
 </div>
<br>

### The Beginning

Java 以虚拟机为中介，横行在各大操作系统平台，比如 Windows, Linux, Android 等等，如果单纯的说 Java 虚拟机这个概念，它只是一种抽象的规范，它规范了每个实现者必须实现的特性，但是又会给实现者留下多种选择，比如说 JRockit, Hotpot等，目前来说 Hotpot 占绝对的市场地位。

除了 Hotpot 之外，作为 Android 开发者，我们感觉最亲切的当然是 Davlik 和 ART 虚拟机，它们也是 Java 虚拟机，但是因为它们要服务于移动设备平台，移动设备的内存和处理速度有限，所以 Android 专门做了优化。

作为 Android 开发者来说，了解虚拟机非常有必要，特别是类加载机制和垃圾回收机制。现在市面上已经有很多分享虚拟机的书籍和资料，我将跟随前辈的脚步，一窥虚拟机的世界。


参考书籍：

《深入理解 Java 虚拟机：JVM 高级特性与最佳实践》，周志明

《Java 虚拟机规范,Java SE 8 版》，Tim Lindholm, Frank Yellin, Gilad Bracha, Alex Buckley

《实战 JVM 虚拟机: JVM 故障诊断与性能优化》，葛一鸣

《深入 Java 虚拟机》，Bill Venners
