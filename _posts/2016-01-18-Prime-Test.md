---
title: 利用无穷数据流进行素数筛选
description: >
  利用无穷数据流进行素数筛选
layout: post
categories: Algorithm
---

最近看了SICP中关于数据流的介绍后,感觉脑洞大开, 就尝试把其中用数据流求素数的方法,用javascript的形式来实现, 以检测自己对递归的理解.本文首先介绍几种素数检测算法, 最后构造一种无穷流, 将输入看作一种无穷流, 筛选出第n个素数。

#素数检测算法

##试除法
尝试从2到$\sqrt{n}$的整数是否整除N, 这是最简单粗暴的一种方式, 地球人都会, 在此不讨论.

##拉宾米勒素数测试
该方法利用费马小定理

```
a^(n - 1) mod n = 1,  (a, n互质, 且n为素数)
```

当一个数字满足上式时, 不一定为素数, 但是不满足上式时,一定不是素数. 进行若干次检测后, 可以较高的准确性来判断一个数是否为素数(满足上式的合数个数非常少). 这个算法因其高效以及非常低的错误率在实际工程中使用非常广泛.

##筛选法
为了找出小于n的所有素数,构造一个由2到n组成的数组.2是第一个素数, 为了找出2以外的素数,就要从其余的整数中过滤掉2所有的倍数,这样就留下了
一个从3开始的数组。由于3没有被小于它的任何素数整除，因此3就是下一个素数,依次类推,就可以求出所有小于等于n的所有素数.以下是js代码示例.

```
var prime = function(n) {
	var primes = [];
	for (var i = 2; i < n + 1; i++) {
		primes[i] = 1;
	}
	var multi = 1;
	var i = 2;
	for (i = 2; i <=n ;i++) {
		multi = 2;
		if (primes[i] != 1) {
			continue;
		}
		while (i * multi <= n) {
			primes[i * multi] = 0;
			multi++;
		}
	}
	var ret = [];
	for (i = 2; i < n + 1; i++) {
		if (primes[i] === 1) {
			ret.push(i);
		}
	}
	return ret;
}
```

这种方法可以较快的计算出前n个素数, 但是需要以数组为辅助. 空间浪费比较大. 下面介绍一种以递归思想和延迟计算构造数据流的方式,来筛选素数的方法.

##数据流
当我们采用数组为辅助工具来筛选素数时,需要预先定义好一定大小的数组. 例如, 当我们要求第3个素数时, 我们可能需要先定义一个大小为10的数组. 可是当我们要求第50个素数时,应该定义多大的数组呢? 100还是200? 我们不知道,最好的方式是当程序用到时自动去分配.

下面考虑一个整数序列的定义.当我们需要定义一个无穷长度的整数序列时,显然用数组的方式是不合适的, 因为我们无法定义一个无穷长的数组. 我们会考虑定义一种方法, 只有当真正用到这个整数时, 才通过调用一个方法去生成这个整数.
下面考虑这个数据流的构造.首先, 定义数据流.

```
var make_stream = function(start, proc) {
	this.first = start;
	this.proc = proc;
}
```

其中, first代表流的第一个数据, proc代表为了获取数据流后面的部分,需要运行proc过程.

很自然的, 关于这个数据流, 我们需要两个操作数据流的函数. 首先, 我们需要一个操作获取数据流当前的第一个整数.

```
var stream_car = function(stream) {
	return stream.first;
}
```

其次, 我们还需要一个操作获取数据流的后面部分.

```
var stream_cdr = function(stream) {
	return stream.proc();
}
```

接下来, 我们就可以利用这个数据流的定义, 来构造一个无穷的整数序列.

```
var integers_starting_from = function(n) {
	return new make_stream(n, currying(integers_starting_from, n + 1));
}
var currying = function(fn) {
    var args = [].slice.call(arguments, 1);
    return function() {
        var newArgs = args.concat([].slice.call(arguments));
        return fn.apply(null, newArgs);
    };
};
```

使用柯里化函数的目的在于使得对于integers_starting_from(n + 1)的计算可以推迟到调用stream_cdr时进行.

对integers_starting_from稍作修改

```
var integers_starting_from = function(n) {
	console.log(n);
	return new make_stream(n, currying(integers_starting_from, n + 1));
}
```

运行integers_starting_from(2),运行结果:

![image]({{site.baseurl}}/images/20160121.png)

可以看到,函数只计算了一遍, 而不会计算后续的值.

在此基础上, 我们构造素数流.首先,定义一个代表空的数据流.

```
var stream_null = function(stream) {
	if (null == stream || !stream.first){
		return true;
	}
	return false;
}
```

接下来设计一个过滤器, 过滤掉非素数.

```
var stream_filter = function(pred, stream) {
	if (stream_null(stream)) {
		return null;
	} else if (pred(stream_car(stream))) {
		return new make_stream(stream_car(stream), currying(stream_filter, pred, stream_cdr(stream)));
	} else {
		return stream_filter(pred, stream_cdr(stream));
	}
}
```

其中pred代表对数据的检测器, 在筛选素数时,它检测是否可以整数. stream代表经过这个过滤器的数据流.

最后定义一个素数的筛选器.

```
var sieve = function(stream) {
	var first = stream_car(stream);
	var dividable = function(x) {
		if ((x % first) == 0) {
			return false;
		}
		return true;
	}
	return new make_stream(first, currying(sieve, stream_filter(dividable, stream_cdr(stream))));
}
```

通过运行

```
var primes = sieve(integers_starting_from(2)); 
```

可以得到素数流. 定义函数

```
var stream_ref = function(stream, n) {
	if (n == 0) {
		return stream_car(stream);
	} else {
		return stream_ref(stream_cdr(stream), n - 1);
	}
}
```

可以取得数据流中的第n个数据. 运行

```
var ret = stream_ref(primes, 50);
```

可以得到第50个素数.

这是一种非常精妙的设计,理论上来说可以求得第任意个素数, 但在实践中受到调用栈大小的限制. 当然, 我们可以用一定的方法, 将其改成迭代的形式, 以避免溢出, 但这不是本文的讨论范围, 在此不做讨论.

参考资料: 计算机程序的构造与解释

