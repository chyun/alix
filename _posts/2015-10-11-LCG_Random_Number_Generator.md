---
title: 随机数生成原理初探-LCG
description: >
  现代计算机中主流的伪随机数生成器就是LCG
layout: post
---

categories: Cryptography
#随机数生成原理初探

在计算机上可以用物理方法来产生随机数，但价格昂贵，不能重复，使用不便。另一种方法是用数学递推公式产生，这样产生的序列与真正的随机数序列不同，所以称为伪随机数或伪随机序列. 虽然带了一个伪字, 它的生成却一点也不容易. 看似几个简简单单的公式却是前辈们努力了几代的成果, 相关的研究可以写好几本书了.

#线性同余生成器(Linear Congruential Method, LCG)

产生随机数的方法是先用一定的方法产生[0,1)均匀分布的随机数，然后通过一个适当的变换就可以得到符合某一概率模型的随机数。

常用的产生[0,1)均匀分布的随机数的方法就是线性同余法. 线性同余法的基本理念就是:

```
对于一个一般正整数正整数x和一个给定的正整数m, x除m的余数是不可预测的.
```

现在的计算机中几乎所有的运行库提供的rand都是采用的LCG, 形如:

X<sub>n+1</sub> = (aX<sub>n</sub> + c) mod m,     n >= 0

其中:

m, 模数, m > 0;

a, 乘数, m > a >= 0;

c, 增量, m > c >= 0;

X<sub>0</sub>, 开始值, m > X<sub>0</sub> >= 0;

在线性同余法中, 随机数生成的质量与参数的选择是息息相关的, 例如: m = 10, X<sub>0</sub> = a = c =7, 则生成的随机数序列如下所示:

7, 6, 9, 0, 7, 6, 9, 0,...

可以看到线性同余序列和快会进入一个循环过程中. 事实上, 任何一个线性同余序列, 最终都会进入一个无休止的循环中, 这种循环称为周期, 一次循环中数字的个数称为周期的长度, 如上述的例子周期长度就是4. 而一个良好的随机数生成器当然是周期越长越好. 

当c = 0时, 称为乘同余生成器(Multiplicative Congruential Generator, MCG).

#模数的选择
显然, 要生成较好的随机数, 周期的长度当然是越长越好. 而周期的最大长度是m, 因此m的取值应当尽可能的大一些. 

m取值的另一个考虑要素就是计算速度, 取模运算如果按照除法来算是非常缓慢的, 但是如果将m取成2^e, 计算机使用移位来计算则是非常迅速的(事实上将m取成2^e - 1计算也是非常迅速的, 这里涉及到计算机里的一些trick, 详细可翻阅Donald E. Knuth的"计算机程序设计艺术"第二卷第三章). 

#乘数的选择

如上所述, a的取值, 最好状态下, 应该是使得随机数生成器的周期达到最大m. 要令LCG达到最大周期，应符合以下条件: 

    c, m互素
    m的所有质因数都能整除a−1
    若m是4的倍数, a−1也是
    a, c, x0都比m小
    a, c是正整数

不过这些约束怎么来的本文就暂不讨论了(同上, 参阅"计算机程序设计艺术"第二卷第三章).

#主流函数的参数取值

参数的选择除了满足速度快, 周期大意外, 还有很重要的一点就是概率分布需要服从平均分布.以下是来自维基百科的一些主流随机函数的参数取值.

![image](https://raw.githubusercontent.com/chyun/Blog/gh-pages/images/2015-09-30-LCG_parameters.png)

评估随机数生成器随机性效果的方法有许多, 其中最著名的应该是Knuth's Spectral Test. 这种方法是在实践中衡量生成器效果的, 本文不做讨论. 

关于随机数生成的参数选择, Knuth在"计算机程序设计艺术"第二卷第三章中进行了详细的论述, 有兴趣的可以自行翻阅(有难度).





