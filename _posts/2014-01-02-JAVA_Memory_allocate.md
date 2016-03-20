---
title: JAVA内存分配
description: >
  JAVA内存分配
layout: post
categories: JAVA
---

JAVA内存分配
=====================

JAVA虚拟机自动管理内存的功能使程序员从繁琐细致的内存管理工作中解放出来，但是理解JAVA虚拟机的内存模型对我们提高整个应用程序的性能还是很有帮助的。本文按照[JAVA虚拟机规范][1]介绍一下JAVA虚拟机的内存分配。

java虚拟机内部结构
---

JAVA虚拟机的内部结构如下图所示。JAVA虚拟机定义了许多运行时数据区域，其中一部分由JAVA虚拟机负责创建与销毁，还有一部分由线程负责创建与销毁。

![enter image description here][2]

如图所示，每一个JAVA虚拟机都有类加载子系统，执行引擎和运行时数据区域。本文主要讲解运行时数据区域这一部分，以后会介绍类加载子系统。


1.程序计数器(The pc Register)
---

JAVA虚拟机能支持多个线程的同时运行。每个线程都有自己的程序计数器。每个线程在同一时刻只能执行一个方法。程序计数器中存放的是当前执行的指令的地址，若当前执行的方法是java方法，存放该指令的地址，若是本地方法，则为空。

2.JAVA栈
---

每个线程都有自己的栈空间。JAVA栈由栈帧组成，每一个栈帧表示一个java方法调用。当一个线程调用一个java方法时，java栈中压入一个栈帧，帧中包含了该方法所需的局部变量。当方法结束时，弹出栈帧。
这里介绍两个异常：

    1.当一个线程试图申请的内存空间超过java虚拟机的允许时，会抛出StackOverflowError。
    2.当没有足够的空间分配为一个新的线程分配栈空间时，会抛出OutOfMemoryError：unable to create new native thread。

3.本地方法栈
---

功能与JAVA栈类似，只是这里存放的不是java方法，而是本地方法。

程序计数器，java栈和本地方法栈的示意图如下所示。

![enter image description here][3]

4.heap
---

heap被所有线程共享，存放java类的实例和数组。heap又可分为年轻区和老年区，这部分涉及到java的垃圾回收机制，以后再作介绍。

5.方法区
---

即perm区，存放类信息，如方法，运行时常量池（存放编译器就能确定的变量，如静态变量），域(field)等信息。JAVA虚拟机规范中提到，逻辑上，方法区时堆区的一部分。


  [1]: http://docs.oracle.com/javase/specs/jvms/se7/html/index.html
  [2]: https://github.com/chyun/Blog/blob/gh-pages/images/jvm_internal_structure.gif?raw=true
  [3]: https://github.com/chyun/Blog/blob/gh-pages/images/jvm_pc_statck.gif?raw=true
