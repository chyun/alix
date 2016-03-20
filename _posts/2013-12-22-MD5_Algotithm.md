---
title: MD5算法分析
description: >
  MD5算法分析
layout: post
categories: 密码Hash
---

MD5算法分析及JAVA实现代码
=====================

MD5（单向散列算法）的全称是Message-Digest Algorithm 5（信息-摘要算法），经MD2、MD3和MD4发展而来。MD5算法的使用不需要支付任何版权费用。

MD5功能：
--------
    输入任意长度的信息，经过处理，输出为128位的信息（数字指纹）；
    不同的输入得到的不同的结果（唯一性）；
    根据128位的输出结果不可能反推出输入的信息（不可逆）；

MD5属不属于加密算法：
--------
    认为不属于的人是因为他们觉得不能从密文（散列值）反过来得到原文，即没有解密算法，所以这部分人认为MD5只能属于算法，不能称为加密算法；
    认为属于的人是因为他们觉得经过MD5处理后看不到原文，即已经将原文加密，所以认为MD5属于加密算法；
    我个人支持前者。

MD5用途：
--------
    1、防止被篡改：
    1）比如发送一个电子文档，发送前，我先得到MD5的输出结果a。然后在对方收到电子文档后，对方也得到一个MD5的输出结果b。如果a与b一样就代表中途未被篡改。2）比如我提供文件下载，为了防止不法分子在安装程序中添加木马，我可以在网站上公布由安装文件得到的MD5输出结果。3）SVN在检测文件是否在CheckOut后被修改过，也是用到了MD5.
    2、防止直接看到明文：
    现在很多网站在数据库存储用户的密码的时候都是存储用户密码的MD5值。这样就算不法分子得到数据库的用户密码的MD5值，也无法知道用户的密码(其实这样是不安全的，后面我会提到)。（比如在UNIX系统中用户的密码就是以MD5（或其它类似的算法）经加密后存储在文件系统中。当用户登录的时候，系统把用户输入的密码计算成MD5值，然后再去和保存在文件系统中的MD5值进行比较，进而确定输入的密码是否正确。通过这样的步骤，系统在并不知道用户密码的明码的情况下就可以确定用户登录系统的合法性。这不但可以避免用户的密码被具有系统管理员权限的用户知道，而且还在一定程度上增加了密码被破解的难度。）
    3、防止抵赖（数字签名）：
    这需要一个第三方认证机构。例如A写了一个文件，认证机构对此文件用MD5算法产生摘要信息并做好记录。若以后A说这文件不是他写的，权威机构只需对此文件重新产生摘要信息，然后跟记录在册的摘要信息进行比对，相同的话，就证明是A写的了。这就是所谓的“数字签名”。
MD5算法过程：
--------
对MD5算法简要的叙述可以为：MD5以512位分组来处理输入的信息，且每一分组又被划分为16个32位子分组，经过了一系列的处理后，算法的输出由四个32位分组组成，将这四个32位分组级联后将生成一个128位散列值。


    第一步、填充：如果输入信息的长度(bit)对512求余的结果不等于448，就需要填充使得对512求余的结果等于448。填充的方法是填充一个1和n个0。填充完后，信息的长度就为N*512+448(bit)；
     
    第二步、记录信息长度：用64位来存储填充前信息长度。这64位加在第一步结果的后面，这样信息长度就变为N*512+448+64=(N+1)*512位，如下图所示：
    
![enter image description here][1]

     第三步、装入标准的幻数（四个整数）：标准的幻数（物理顺序）是（A=(01234567)16，B=(89ABCDEF)16，C=(FEDCBA98)16，D=(76543210)16）,这是小端字节序，有的编程语言默认采用大端字节序，如JAVA，在程序中应该是（A=0X67452301L，B=0XEFCDAB89L，C=0X98BADCFEL，D=0X10325476L）。
     
     第四步、对每一个512位的数据块，作四轮循环运算（每一轮循环又有16步）： 
     
     1）将每一512字节细分成16个小组，每个小组32位（4个字节）
    
     2）先认识四个非线性函数(&是与,|是或,~是非,^是异或)
        F(X,Y,Z)=(X&Y)|((~X)&Z)
        G(X,Y,Z)=(X&Z)|(Y&(~Z))
        H(X,Y,Z)=X^Y^Z
        I(X,Y,Z)=Y^(X|(~Z))
		F是一个逐位运算的函数，作用是如果X，那么Y，否则Z。
		G的作用与F类似。
		H是一个逐位奇偶操作函数。
		I与H类似。
   
     3）设Mj表示消息的第j个子分组（从0到15），<<<s表示循环左移s位，则四种操作为：
        FF(a,b,c,d,Mj,s,ti)表示a=b+((a+F(b,c,d)+Mj+ti)<<<s)
        GG(a,b,c,d,Mj,s,ti)表示a=b+((a+G(b,c,d)+Mj+ti)<<<s)
        HH(a,b,c,d,Mj,s,ti)表示a=b+((a+H(b,c,d)+Mj+ti)<<<s)
        II(a,b,c,d,Mj,s,ti)表示a=b+((a+I(b,c,d)+Mj+ti)<<<s)

     4）四轮运算
        第一轮
        a=FF(a,b,c,d,M0,7,0xd76aa478)
        b=FF(d,a,b,c,M1,12,0xe8c7b756)
        c=FF(c,d,a,b,M2,17,0x242070db)
        d=FF(b,c,d,a,M3,22,0xc1bdceee)
        a=FF(a,b,c,d,M4,7,0xf57c0faf)
        b=FF(d,a,b,c,M5,12,0x4787c62a)
        c=FF(c,d,a,b,M6,17,0xa8304613)
        d=FF(b,c,d,a,M7,22,0xfd469501)
        a=FF(a,b,c,d,M8,7,0x698098d8)
        b=FF(d,a,b,c,M9,12,0x8b44f7af)
        c=FF(c,d,a,b,M10,17,0xffff5bb1)
        d=FF(b,c,d,a,M11,22,0x895cd7be)
        a=FF(a,b,c,d,M12,7,0x6b901122)
        b=FF(d,a,b,c,M13,12,0xfd987193)
        c=FF(c,d,a,b,M14,17,0xa679438e)
        d=FF(b,c,d,a,M15,22,0x49b40821)

        第二轮
        a=GG(a,b,c,d,M1,5,0xf61e2562)
        b=GG(d,a,b,c,M6,9,0xc040b340)
        c=GG(c,d,a,b,M11,14,0x265e5a51)
        d=GG(b,c,d,a,M0,20,0xe9b6c7aa)
        a=GG(a,b,c,d,M5,5,0xd62f105d)
        b=GG(d,a,b,c,M10,9,0x02441453)
        c=GG(c,d,a,b,M15,14,0xd8a1e681)
        d=GG(b,c,d,a,M4,20,0xe7d3fbc8)
        a=GG(a,b,c,d,M9,5,0x21e1cde6)
        b=GG(d,a,b,c,M14,9,0xc33707d6)
        c=GG(c,d,a,b,M3,14,0xf4d50d87)
        d=GG(b,c,d,a,M8,20,0x455a14ed)
        a=GG(a,b,c,d,M13,5,0xa9e3e905)
        b=GG(d,a,b,c,M2,9,0xfcefa3f8)
        c=GG(c,d,a,b,M7,14,0x676f02d9)
        d=GG(b,c,d,a,M12,20,0x8d2a4c8a)

        第三轮
        a=HH(a,b,c,d,M5,4,0xfffa3942)
        b=HH(d,a,b,c,M8,11,0x8771f681)
        c=HH(c,d,a,b,M11,16,0x6d9d6122)
        d=HH(b,c,d,a,M14,23,0xfde5380c)
        a=HH(a,b,c,d,M1,4,0xa4beea44)
        b=HH(d,a,b,c,M4,11,0x4bdecfa9)
        c=HH(c,d,a,b,M7,16,0xf6bb4b60)
        d=HH(b,c,d,a,M10,23,0xbebfbc70)
        a=HH(a,b,c,d,M13,4,0x289b7ec6)
        b=HH(d,a,b,c,M0,11,0xeaa127fa)
        c=HH(c,d,a,b,M3,16,0xd4ef3085)
        d=HH(b,c,d,a,M6,23,0x04881d05)
        a=HH(a,b,c,d,M9,4,0xd9d4d039)
        b=HH(d,a,b,c,M12,11,0xe6db99e5)
        c=HH(c,d,a,b,M15,16,0x1fa27cf8)
        d=HH(b,c,d,a,M2,23,0xc4ac5665)

        第四轮
        a=II(a,b,c,d,M0,6,0xf4292244)
        b=II(d,a,b,c,M7,10,0x432aff97)
        c=II(c,d,a,b,M14,15,0xab9423a7)
        d=II(b,c,d,a,M5,21,0xfc93a039)
        a=II(a,b,c,d,M12,6,0x655b59c3)
        b=II(d,a,b,c,M3,10,0x8f0ccc92)
        c=II(c,d,a,b,M10,15,0xffeff47d)
        d=II(b,c,d,a,M1,21,0x85845dd1)
        a=II(a,b,c,d,M8,6,0x6fa87e4f)
        b=II(d,a,b,c,M15,10,0xfe2ce6e0)
        c=II(c,d,a,b,M6,15,0xa3014314)
        d=II(b,c,d,a,M13,21,0x4e0811a1)
        a=II(a,b,c,d,M4,6,0xf7537e82)
        b=II(d,a,b,c,M11,10,0xbd3af235)
        c=II(c,d,a,b,M2,15,0x2ad7d2bb)
        d=II(b,c,d,a,M9,21,0xeb86d391)

    5）每轮循环后，将A，B，C，D分别加上a，b，c，d，然后进入下一循环。
    
JAVA核心代码
-----

MD5初始的128位值为初始变量，为4组8位16进制数。分别为：

```
A=0x67452301

B=0xefcdab89

C=0x98badcfe

D=0x98badcfe
```
  
在java中用数组封装：

```
    private long[] state = new long[4]; // state (ABCD)  
    state[0] = 0x67452301L;  
    state[1] = 0xefcdab89L;  
    state[2] = 0x98badcfeL;  
    state[3] = 0x10325476L;  
```

基本处理思想：初始为128个bit 信息，将128bit 初始信息与分割后的每一块512bit 数据经过4组16次hash循环算法，得到128bit 的新信息，依次循环，直到与最后一组512 bit的数据hash算法后，就得到的整个文件的MD5值。每轮循环后，将A, B, C, D 分别加上a, b, c, d, 然后进入下一循环。

MD5主循环有四轮，每轮循环都很相似。第一轮进行16次操作。每次操作对a、b、c和d中的其中三个作一次非线性函数运算，然后将所得结果加上第四个变量，文本的一个子分组和一个常数。再将所得结果向左环移一个不定的数，并加上a、b、c或d中之一。最后用该结果取代a、b、c或d中之一。

![enter image description here][2]

java代码：

```
    /** 
     * F, G, H ,I 是4个基本的MD5函数 把它们封装成private方法，名字保持一致。 
     *  
     * @param x 
     * @param y 
     * @param z 
     * @return 
     */  
    private long F(long x, long y, long z) {  
        return (x & y) | ((~x) & z);  
      
    }  
      
    private long G(long x, long y, long z) {  
        return (x & z) | (y & (~z));  
      
    }  
      
    private long H(long x, long y, long z) {  
        return x ^ y ^ z;  
    }  
      
    private long I(long x, long y, long z) {  
        return y ^ (x | (~z));  
    } 
```

进一步位运算： FF, GG, HH和II

 

FF(a,b,c,d,Mj,s,ti）表示 a = b + ((a + F(b,c,d) + Mj + ti) << s)

GG(a,b,c,d,Mj,s,ti）表示 a = b + ((a + G(b,c,d) + Mj + ti) << s)

HH(a,b,c,d,Mj,s,ti）表示 a = b + ((a + H(b,c,d) + Mj + ti) << s)

Ⅱ（a,b,c,d,Mj,s,ti）表示 a = b + ((a + I(b,c,d) + Mj + ti) << s)

Mj表示消息的第j个子分组（从0到15），常数ti是2^32*abs(sin(i)）的整数部分，i取值从1到64

作用如下图：

![enter image description here][3]

java代码实现：

```
/**
 * FF,GG,HH和II将调用F,G,H,I进行近一步变换 FF, GG, HH和II
 * 
 * @param a
 * @param b
 * @param c
 * @param d
 * @param Mj : 第j个子分组
 * @param s : 左移位数
 * @param ac : 2^32 * abs(sin(i))的整数部分，i取值从1到64，单位是弧度。
 * @return
 */
private long FF(long a, long b, long c, long d, long Mj, long s, long ac) {
	a += F(b, c, d) + Mj + ac;
	a = ((int) a << s) | ((int) a >>> (32 - s));
	a += b;
	return a;
}

private long GG(long a, long b, long c, long d, long Mj, long s, long ac) {
	a += G(b, c, d) + Mj + ac;
	a = ((int) a << s) | ((int) a >>> (32 - s));
	a += b;
	return a;
}

private long HH(long a, long b, long c, long d, long Mj, long s, long ac) {
	a += H(b, c, d) + Mj + ac;
	a = ((int) a << s) | ((int) a >>> (32 - s));
	a += b;
	return a;
}

private long II(long a, long b, long c, long d, long Mj, long s, long ac) {
	a += I(b, c, d) + Mj + ac;
	a = ((int) a << s) | ((int) a >>> (32 - s));
	a += b;
	return a;
}
```
循环哈希算法

```
第一轮
FF(a,b,c,d,M0,7,0xd76aa478）
FF(d,a,b,c,M1,12,0xe8c7b756）
FF(c,d,a,b,M2,17,0x242070db)
FF(b,c,d,a,M3,22,0xc1bdceee)
FF(a,b,c,d,M4,7,0xf57c0faf)
FF(d,a,b,c,M5,12,0x4787c62a)
FF(c,d,a,b,M6,17,0xa8304613）
FF(b,c,d,a,M7,22,0xfd469501）
FF(a,b,c,d,M8,7,0x698098d8）
FF(d,a,b,c,M9,12,0x8b44f7af)
FF(c,d,a,b,M10,17,0xffff5bb1）
FF(b,c,d,a,M11,22,0x895cd7be)
FF(a,b,c,d,M12,7,0x6b901122）
FF(d,a,b,c,M13,12,0xfd987193）
FF(c,d,a,b,M14,17,0xa679438e)
FF(b,c,d,a,M15,22,0x49b40821）
第二轮
GG(a,b,c,d,M1,5,0xf61e2562）
GG(d,a,b,c,M6,9,0xc040b340）
GG(c,d,a,b,M11,14,0x265e5a51）
GG(b,c,d,a,M0,20,0xe9b6c7aa)
GG(a,b,c,d,M5,5,0xd62f105d)
GG(d,a,b,c,M10,9,0x02441453）
GG(c,d,a,b,M15,14,0xd8a1e681）
GG(b,c,d,a,M4,20,0xe7d3fbc8）
GG(a,b,c,d,M9,5,0x21e1cde6）
GG(d,a,b,c,M14,9,0xc33707d6）
GG(c,d,a,b,M3,14,0xf4d50d87）
GG(b,c,d,a,M8,20,0x455a14ed)
GG(a,b,c,d,M13,5,0xa9e3e905）
GG(d,a,b,c,M2,9,0xfcefa3f8）
GG(c,d,a,b,M7,14,0x676f02d9）
GG(b,c,d,a,M12,20,0x8d2a4c8a)
第三轮
HH(a,b,c,d,M5,4,0xfffa3942）
HH(d,a,b,c,M8,11,0x8771f681）
HH(c,d,a,b,M11,16,0x6d9d6122）
HH(b,c,d,a,M14,23,0xfde5380c)
HH(a,b,c,d,M1,4,0xa4beea44）
HH(d,a,b,c,M4,11,0x4bdecfa9）
HH(c,d,a,b,M7,16,0xf6bb4b60）
HH(b,c,d,a,M10,23,0xbebfbc70）
HH(a,b,c,d,M13,4,0x289b7ec6）
HH(d,a,b,c,M0,11,0xeaa127fa)
HH(c,d,a,b,M3,16,0xd4ef3085）
HH(b,c,d,a,M6,23,0x04881d05）
HH(a,b,c,d,M9,4,0xd9d4d039）
HH(d,a,b,c,M12,11,0xe6db99e5）
HH(c,d,a,b,M15,16,0x1fa27cf8）
HH(b,c,d,a,M2,23,0xc4ac5665）
第四轮
Ⅱ（a,b,c,d,M0,6,0xf4292244）
Ⅱ（d,a,b,c,M7,10,0x432aff97）
Ⅱ（c,d,a,b,M14,15,0xab9423a7）
Ⅱ（b,c,d,a,M5,21,0xfc93a039）
Ⅱ（a,b,c,d,M12,6,0x655b59c3）
Ⅱ（d,a,b,c,M3,10,0x8f0ccc92）
Ⅱ（c,d,a,b,M10,15,0xffeff47d)
Ⅱ（b,c,d,a,M1,21,0x85845dd1）
Ⅱ（a,b,c,d,M8,6,0x6fa87e4f)
Ⅱ（d,a,b,c,M15,10,0xfe2ce6e0)
Ⅱ（c,d,a,b,M6,15,0xa3014314）
Ⅱ（b,c,d,a,M13,21,0x4e0811a1）
Ⅱ（a,b,c,d,M4,6,0xf7537e82）
Ⅱ（d,a,b,c,M11,10,0xbd3af235）
Ⅱ（c,d,a,b,M2,15,0x2ad7d2bb)
Ⅱ（b,c,d,a,M9,21,0xeb86d391）
```

为了方便期间，我们定义一个所用到的s用4 * 4的矩形来存储。

```
S = { {7, 12, 17, 22},
      {5,  9, 14, 20},
      {4, 11, 16, 23},
      {6, 10, 15, 21}
    }
```
上面的部分用java代码实现：

```
/**
 * MD5核心变换程序，有md5Update调用
 * @param block 512位的数据块
 */
private void md5Transform(byte block[]) {
	long a = state[0], b = state[1], c = state[2], d = state[3];
	long[] x = new long[16];

	Decode(x, block, 64);//把long类型（64bit）按顺序拆成byte数组

	/* Round 1 */
	a = FF(a, b, c, d, x[0], S11, 0xd76aa478L); /* 1 */
	d = FF(d, a, b, c, x[1], S12, 0xe8c7b756L); /* 2 */
	c = FF(c, d, a, b, x[2], S13, 0x242070dbL); /* 3 */
	b = FF(b, c, d, a, x[3], S14, 0xc1bdceeeL); /* 4 */
	a = FF(a, b, c, d, x[4], S11, 0xf57c0fafL); /* 5 */
	d = FF(d, a, b, c, x[5], S12, 0x4787c62aL); /* 6 */
	c = FF(c, d, a, b, x[6], S13, 0xa8304613L); /* 7 */
	b = FF(b, c, d, a, x[7], S14, 0xfd469501L); /* 8 */
	a = FF(a, b, c, d, x[8], S11, 0x698098d8L); /* 9 */
	d = FF(d, a, b, c, x[9], S12, 0x8b44f7afL); /* 10 */
	c = FF(c, d, a, b, x[10], S13, 0xffff5bb1L); /* 11 */
	b = FF(b, c, d, a, x[11], S14, 0x895cd7beL); /* 12 */
	a = FF(a, b, c, d, x[12], S11, 0x6b901122L); /* 13 */
	d = FF(d, a, b, c, x[13], S12, 0xfd987193L); /* 14 */
	c = FF(c, d, a, b, x[14], S13, 0xa679438eL); /* 15 */
	b = FF(b, c, d, a, x[15], S14, 0x49b40821L); /* 16 */

	/* Round 2 */
	a = GG(a, b, c, d, x[1], S21, 0xf61e2562L); /* 17 */
	d = GG(d, a, b, c, x[6], S22, 0xc040b340L); /* 18 */
	c = GG(c, d, a, b, x[11], S23, 0x265e5a51L); /* 19 */
	b = GG(b, c, d, a, x[0], S24, 0xe9b6c7aaL); /* 20 */
	a = GG(a, b, c, d, x[5], S21, 0xd62f105dL); /* 21 */
	d = GG(d, a, b, c, x[10], S22, 0x2441453L); /* 22 */
	c = GG(c, d, a, b, x[15], S23, 0xd8a1e681L); /* 23 */
	b = GG(b, c, d, a, x[4], S24, 0xe7d3fbc8L); /* 24 */
	a = GG(a, b, c, d, x[9], S21, 0x21e1cde6L); /* 25 */
	d = GG(d, a, b, c, x[14], S22, 0xc33707d6L); /* 26 */
	c = GG(c, d, a, b, x[3], S23, 0xf4d50d87L); /* 27 */
	b = GG(b, c, d, a, x[8], S24, 0x455a14edL); /* 28 */
	a = GG(a, b, c, d, x[13], S21, 0xa9e3e905L); /* 29 */
	d = GG(d, a, b, c, x[2], S22, 0xfcefa3f8L); /* 30 */
	c = GG(c, d, a, b, x[7], S23, 0x676f02d9L); /* 31 */
	b = GG(b, c, d, a, x[12], S24, 0x8d2a4c8aL); /* 32 */

	/* Round 3 */
	a = HH(a, b, c, d, x[5], S31, 0xfffa3942L); /* 33 */
	d = HH(d, a, b, c, x[8], S32, 0x8771f681L); /* 34 */
	c = HH(c, d, a, b, x[11], S33, 0x6d9d6122L); /* 35 */
	b = HH(b, c, d, a, x[14], S34, 0xfde5380cL); /* 36 */
	a = HH(a, b, c, d, x[1], S31, 0xa4beea44L); /* 37 */
	d = HH(d, a, b, c, x[4], S32, 0x4bdecfa9L); /* 38 */
	c = HH(c, d, a, b, x[7], S33, 0xf6bb4b60L); /* 39 */
	b = HH(b, c, d, a, x[10], S34, 0xbebfbc70L); /* 40 */
	a = HH(a, b, c, d, x[13], S31, 0x289b7ec6L); /* 41 */
	d = HH(d, a, b, c, x[0], S32, 0xeaa127faL); /* 42 */
	c = HH(c, d, a, b, x[3], S33, 0xd4ef3085L); /* 43 */
	b = HH(b, c, d, a, x[6], S34, 0x4881d05L); /* 44 */
	a = HH(a, b, c, d, x[9], S31, 0xd9d4d039L); /* 45 */
	d = HH(d, a, b, c, x[12], S32, 0xe6db99e5L); /* 46 */
	c = HH(c, d, a, b, x[15], S33, 0x1fa27cf8L); /* 47 */
	b = HH(b, c, d, a, x[2], S34, 0xc4ac5665L); /* 48 */

	/* Round 4 */
	a = II(a, b, c, d, x[0], S41, 0xf4292244L); /* 49 */
	d = II(d, a, b, c, x[7], S42, 0x432aff97L); /* 50 */
	c = II(c, d, a, b, x[14], S43, 0xab9423a7L); /* 51 */
	b = II(b, c, d, a, x[5], S44, 0xfc93a039L); /* 52 */
	a = II(a, b, c, d, x[12], S41, 0x655b59c3L); /* 53 */
	d = II(d, a, b, c, x[3], S42, 0x8f0ccc92L); /* 54 */
	c = II(c, d, a, b, x[10], S43, 0xffeff47dL); /* 55 */
	b = II(b, c, d, a, x[1], S44, 0x85845dd1L); /* 56 */
	a = II(a, b, c, d, x[8], S41, 0x6fa87e4fL); /* 57 */
	d = II(d, a, b, c, x[15], S42, 0xfe2ce6e0L); /* 58 */
	c = II(c, d, a, b, x[6], S43, 0xa3014314L); /* 59 */
	b = II(b, c, d, a, x[13], S44, 0x4e0811a1L); /* 60 */
	a = II(a, b, c, d, x[4], S41, 0xf7537e82L); /* 61 */
	d = II(d, a, b, c, x[11], S42, 0xbd3af235L); /* 62 */
	c = II(c, d, a, b, x[2], S43, 0x2ad7d2bbL); /* 63 */
	b = II(b, c, d, a, x[9], S44, 0xeb86d391L); /* 64 */

	state[0] += a;
	state[1] += b;
	state[2] += c;
	state[3] += d;
}
```

经过上述步骤后，再对下一个512位的分组进行操作。所有分组操作完成后，就可以得到文件的MD5值了

  [1]: https://github.com/chyun/Blog/blob/gh-pages/images/MD5_Length.jpg?raw=true
  [2]: https://github.com/chyun/Blog/blob/gh-pages/images/MD5_Process.jpg?raw=true
  [3]: https://github.com/chyun/Blog/blob/gh-pages/images/MD5_funtion.png?raw=true
