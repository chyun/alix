---
title: JAVA静态方法的调用
description: >
  JAVA对静态方法的调用
layout: post
categories: JAVA
---

#JAVA练习题
首先，先看一道题目（题目来源是[Dev.tucao()][1]）：
下面的程序对巴辛吉小鬣狗和其它狗之间的行为差异进行了建模。如果你不知道什么是巴辛吉小鬣狗，那么我告诉你，这是一种产自非洲的小型卷尾狗，它们从来都不叫唤。那么，这个程序将打印出什么呢？

```
class Dog {
    public static void bark() {
        System.out.print("woof ");
    }
}

class Basenji extends Dog {
    public static void bark() { }
}

public class Bark {
    public static void main(String args[]) {
        Dog woofer = new Dog();
        Dog nipper = new Basenji();
        woofer.bark();
        nipper.bark();
    }
}

A.woof woof
B.woof
C.无打印
D.异常
```

这道题的答案是A。

随意地看一看，好像该程序应该只打印一个woof。毕竟，Basenji 扩展自Dog，并且它的bark 方法定义为什么也不做。main 方法调用了bark 方法，第一次是在Dog 类型的woofer 上调用，第二次是在Basenji 类型的nipper 上调用。巴辛吉小鬣狗并不会叫唤，但是很显然，这一只会。如果你运行该程序，就会发现它打印的是woof woof。这只可怜的小家伙到底出什么问题了？

问题在于bark 是一个静态方法，而对静态方法的调用不存在任何动态的分派机制[JLS 15.12.4.4]。当一个程序调用了一个静态方法时，要被调用的方法都是在编译时刻被选定的，而这种选定是基于修饰符的编译期类型而做出的，修饰符的编译期类型就是我们给出的方法调用表达式中圆点左边部分的名字。在本案中，两个方法调用的修饰符分别是变量woofer 和nipper，它们都被声明为Dog类型。因为它们具有相同的编译期类型，所以编译器使得它们调用的是相同的方法：Dog.bark。这也就解释了为什么程序打印出woof woof。尽管nipper 的运 行期类型是Basenji，但是编译器只会考虑其编译器类型。

要订正这个程序，直接从两个bark 方法定义中移除掉static 修饰符即可。这样，Basenji 中的bark 方法将覆写而不是隐藏Dog 中的bark 方法，而该程序也将会打印出woof，而不是woof woof。通过覆写，你可以获得动态的分派；而通过隐藏，你却得不到这种特性。

当你调用了一个静态方法时，通常都是用一个类而不是表达式来标识它：例如，Dog.bark 或Basenji.bark。当你在阅读一个Java 程序时，你会期望类被用作为静态方法的修饰符，这些静态方法都是被静态分派的，而表达式被用作为实例方法的修饰符，这些实例方法都是被动态分派的。通过耦合类和变量的不同的命名规范，我们可以提供一个很强的可视化线索，用来表明一个给定的方法调用是动态的还是静态的。本谜题的程序使用了一个表达式作为静态方法调用的修饰符，这就误导了我们。千万不要用一个表达式来标识一个静态方法调用。


  [1]: http://www.devtucao.com/dailyTest/516becfa0cf2095afff7726b?utm_source=douban&utm_medium=sns&utm_campaign=516becfa0cf2095afff7726b