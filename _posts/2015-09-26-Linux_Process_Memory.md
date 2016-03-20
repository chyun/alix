---
title: LINUX 对进程内存分配的简单介绍
description: >
  LINUX 对进程内存分配的简单介绍
layout: post
categories: Cassandra
---

#进程内存结构

本文简单介绍一下, 进程在运行时, 在内存中的结构, 关于Linux对内存的分页, 虚拟内存, 地址转换等内容属于操作系统的基础, 本文不作介绍.

毫无疑问, 进程都必须占用一定数量的内存, 它或是用来存放从磁盘载入的程序代码, 或是存放取自用户输入的数据等等. 不过进程对这些内存的管理方式因内存用途不一而不尽相同, 有些内存是事先静态分配和统一回收的,而有些却是按需要动态分配和回收的.

对任何一个普通进程来讲，它都会涉及到5种不同的内存段, 即“程序代码段”, "程序数据段", "BSS", "堆", "栈"。现在我们来简单归纳一下进程对应的内存空间中所包含的5种不同的数据区。

                +------------+
                |   stack    |
                +------------+
                |    heap    |
                +------------+
                |    BSS     |
                +------------+
                |    DATA    |
                +------------+
                |    CODE    |
                +------------+
                
上述几种内存区域中数据段、BSS和堆通常是被连续存储的——内存位置上是连续的(事实上在栈和堆之间还存在一个特殊的区域叫Mapping Area: 这是与mmap系统调用相关的区域, 本文暂不介绍), 而代码段和栈往往会被独立存放. 有趣的是, 堆和栈两个区域关系很"暧昧", 他们一个向下"长"(i386体系结构中栈向下, 堆向上), 一个向上"长", 相对而生. 但你不必担心他们会碰头，因为他们之间间隔很大(到底大到多少, 你可以从下面的例子程序计算一下), 绝少有机会能碰到一起.

Note: 事实上在stack和heap之间还存在一种内存空间叫Mapping Area, 由系统调用mmap来分配, 一般用于分配较大的内存块, 如文件.

首先用一个小例子, 来更具体的说明一下这几个区域的差别.

```
#include<stdio.h>
#include<unistd.h>
int bss_var;
int data_var0=1;
int main(int argc,char **argv)
{
  printf("below are addresses of types of process's mem\n");
  printf("Text location:\n");
  printf("\tAddress of main(Code Segment):%p\n",main);
  printf("____________________________\n");
  int stack_var0=2;
  printf("Stack Location:\n");
  printf("\tInitial end of stack:%p\n",&stack_var0);
  int stack_var1=3;
  printf("\tnew end of stack:%p\n",&stack_var1);
  printf("____________________________\n");
  printf("Data Location:\n");
  printf("\tAddress of data_var(Data Segment):%p\n",&data_var0);
  static int data_var1=4;
  printf("\tNew end of data_var(Data Segment):%p\n",&data_var1);
  printf("____________________________\n");
  printf("BSS Location:\n");
  printf("\tAddress of bss_var:%p\n",&bss_var);
  printf("____________________________\n");
  char *b = sbrk(0);
  printf("Heap Location:\n");
  printf("\tInitial end of heap:%p\n",b);
  brk(b+4);
  b=sbrk(0);
  printf("\tNew end of heap:%p\n",b);
  return 0;
 }
 ```
 
运行结果如下所示:
 
```
below are addresses of types of process's mem
Text location:
	Address of main(Code Segment):0x10e65ec10
____________________________
Stack Location:
	Initial end of stack:0x7fff515a197c
	new end of stack:0x7fff515a1978
____________________________
Data Location:
	Address of data_var(Data Segment):0x10e65f028
	New end of data_var(Data Segment):0x10e65f02c
____________________________
BSS Location:
	Address of bss_var:0x10e65f030
____________________________
Heap Location:
	Initial end of heap:0x10e693000
	New end of heap:0x10e693000
```

可以看到代码段, 数据段, bss段, 堆, 栈的空间依次升高, 栈的空间从高向下扩展, 堆空间从低向上扩展. 

##堆
每个进程都有自己的虚拟地址空间(通过MMU转换成物理地址), 如上所述, 该地址空间被分成若干个部分, 当我们用标准C函数库中的函数malloc分配空间时, 就是分配这一段的内存.

堆是内存中一段连续(逻辑上)的空间, Linux对堆的管理示意如下:

        +--------+------------------------------+---------------------------+------------+
        |        |        Mapped Region         |       Unmapped   Region   | Unavailable|
        +--------+------------------------------+---------------------------+------------+
                 ^                              ^                           ^                 
                 |                              |                           |  
             Heap Start                       break                       rlimit
             
 Linux维护一个break指针，这个指针指向堆空间的某个地址。从堆起始地址到break之间的地址空间为映射好的，可以供进程访问；而从break往上，是未映射的地址空间，如果访问这段空间则程序会报错。
 
要增加一个进程实际的可用堆大小，就需要将break指针向高地址移动。Linux通过brk和sbrk系统调用操作break指针(malloc也是通过这两个系统调用来分配空间的)。两个系统调用的原型如下：
 
 ```
	int brk(void *addr);
	void *sbrk(intptr_t increment);
 ```
 
brk将break指针直接设置为某个地址, 而sbrk将break从当前位置移动increment所指定的增量. brk在执行成功时返回0, 否则返回-1并设置error为ENOMEM; sbrk成功时返回break移动之前所指向的地址, 否则返回(void *)-1.

一个小技巧是, 如果将increment设置为0, 则可以获得当前break的地址.

另外需要注意的是, 由于Linux是按页进行内存映射的, 所以如果break被设置为没有按页大小对齐, 则系统实际上会在最后映射一个完整的页, 从而实际已映射的内存空间比break指向的地方要大一些. 但是使用break之后的地址是很危险的(尽管也许break之后确实有一小块可用内存地址).

系统对每一个进程所分配的资源不是无限的，包括可映射的内存空间，因此每个进程有一个rlimit表示当前进程可用的资源上限. 这个限制可以通过getrlimit系统调用得到, 下面代码获取当前进程虚拟内存空间的rlimit:

```
int main() {
    struct rlimit *limit = (struct rlimit *)malloc(sizeof(struct rlimit));
    getrlimit(RLIMIT_AS, limit);
    printf("soft limit: %ld, hard limit: %ld\n", limit->rlim_cur, limit->rlim_max);
}
```

每种资源有软限制和硬限制, 并且可以通过setrlimit对rlimit进行有条件设置. 其中硬限制作为软限制的上限, 非特权进程只能设置软限制, 且不能超过硬限制.

本文主要参考了[Malloc_Tutorial](http://www.inf.udec.cl/~leo/Malloc_tutorial.pdf), 这片文章中有非常详细的malloc简单实现版本.

