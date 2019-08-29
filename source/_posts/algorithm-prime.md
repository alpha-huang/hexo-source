title: 求第n个质数
author: alpha
tags:
  - 前端
  - js
  - 算法
categories:
  - 面试题
date: 2019-4-29 18:25:56
---
<!--以下是正文-->

> 写一个求第n个质数的算法

<!--more-->

## 一、问题分析

可以分为两步，第一步判断一个数是否是质数，第二步返回第n个质数。

## 二、算法实现
质数是不能被1和其自身之外的整数整除的正整数，因此只需要用一个数去对所有比它小的整数取余，如果余数有0，表示能被整除，则不是质数
``` javascript
function isPrime(n){
	for(var i = 2; i < n; i++){
		if(n % i === 0){
			return false
		}
	}
	return true
}
```
然后从2开始依次判断每个整数是否为质数，直到统计到第n个质数后结束。
``` javascript
function prime(n){
	let m = 0
	let i = 2
	while(m < n){
		if(isPrime(i)){
			m++
		}
		i++
	}
	return i-1;
}
prime(1)//2
prime(2)//3
prime(3)//5
prime(4)//7
```

## 三、算法优化

### 1、平方根

由于如果一个正整数在2到n的平方根之间都不能被整除就可以确定这个数是质数，所以判断质数时只用对2到n的平方根之间的整数取余。

``` javascript
function isPrime(n){
	for(var i = 2; i * i <= n; i++){
		if(n % i === 0){
			return false
		}
	}
	return true
}
```

### 2、2的倍数

另外，由于除2以外所有的偶数都不是质数，所以在循环时可以直接排除，这样取余的数将会少一半

``` javascript
function isPrime(n){
	if(n === 2)return true
	if(n % 2 === 0)return false
	for(var i = 3; i * i <= n; i += 2){
		if(n % i === 0){
			return false
		}
	}
	return true
}
```

### 3、3的倍数

由于除3以外，所有的3的倍数都不是质数，所以在循环时可以直接排除，这样取余的数将会又少1/3。要在循环时跳过2的倍数和3的倍数，可以把每6个数分为一组，例如第一组：[1,2,3,4,5,6]，第二组：[7,8,9,10,11,12]，第m组：[6m+1,6m+2,6m+3,6m+4,6m+5,6m+6]，除去2的倍数和3的倍数，剩下的就是6m+1和6m+5。

写一个循环，打印出0到n之间的6m+1和6m+5

``` javascript
let n = 50
let result = []
for(var i = 0; i <= n; i += 6){
	result.push(i+1,i+5)
}
console.log(result)//[1, 5, 7, 11, 13, 17, 19, 23, 25, 29, 31, 35, 37, 41, 43, 47, 49, 53]
```
如此，判断是否是质数的函数就可以改造为

``` javascript
function isPrime(n){
	if(n === 2 || n === 3)return true
	if(n % 2 === 0 || n % 3 === 0)return false
	for(var i = 5; i * i <= n; i += 6){
		if(n % (i + 2) === 0 || n % i === 0){
			return false
		}
	}
	return true
}
```

### 4、5的倍数
由于除5以外，所有的5的倍数都不是质数，所以在循环时可以直接排除，这样取余的数将会又减少1/5。要在循环时跳过2的倍数、3的倍数和5的倍数，可以把每30个数分为一组，第m组：[30m+1,30m+2,30m+3,...,30m+30]，除去2的倍数、3的倍数和5的倍数，剩下的就是30m+1、30m+7、30m+11、30m+13、30m+17、30m+19、30m+23、20m+29。

写一个循环，打印出上面这些数

``` javascript
let n = 100
let result = []
for(var i = 5; i <= n; i += 30){
	result.push(i - 4, i + 2, i + 6, i + 8, i + 12, i + 14, i + 18, i + 24)
}
console.log(result)//[1, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 49, 53, 59, 61, 67, 71, 73, 77, 79, 83, 89, 91, 97, 101, 103, 107, 109, 113, 119]
```

如此，判断是否是质数的函数就可以改造为

``` javascript
function isPrime(n){
	if(n === 2 || n === 3 || n === 5)return true
	if(n % 2 === 0 || n % 3 === 0 || n % 5 === 0)return false
	for(var i = 5; i * i <= n; i += 30){
		if(n % (i + 26) === 0
		|| n % (i + 2) === 0
		|| n % (i + 6) === 0
		|| n % (i + 8) === 0
		|| n % (i + 12) === 0
		|| n % (i + 14) === 0
		|| n % (i + 18) === 0
		|| n % (i + 24) === 0
		){
			return false
		}
	}
	return true
}
```

### 5、最终代码

以此类推，还可以确定所有7的倍数、11的倍数、13的倍数等都不是质数，但是越往后，可以排除的数越少，而要在循环里判断条件中列出来的数越多，可以想象，极端情况下就是手动列举出所有的质数。

综合考虑代码的简洁性和效率，只需要排除2的倍数和3的倍数即可。

有了判断一个数是不是质数的方法后，就需要从2开始往后循环，找出第n个质数。同样，这里也可以直接跳过2的倍数和3的倍数。

最终代码为

``` javascript
function isPrime(n){
	if(n === 2 || n === 3)return true
	if(n % 2 === 0 || n % 3 === 0)return false
	for(var i = 5; i * i <= n; i += 6){
		if(n % (i + 2) === 0 || n % i === 0){
			return false
		}
	}
	return true
}
function prime(n){
	if(n <= 2)return [2,3][n-1]
	let m = 2
	let i = 5
	let r
	while(m < n){
		if(isPrime(i)){
			m++
			if(m === n){
                r = i
                break;
            }
		}
		if(isPrime(i + 2)){
			m++
			if(m === n){
                r = i + 2
                break;
            }
		}
		i += 6
	}
	return r;
}
```

## 四、总结

这题的核心在于对质数的判断，从定义上来说要判断一个数是否是质数，只需要判断这个数能否被每一个小于该数的整数整数即可。但是实际上我们不用挨个判断每一个数，可以根据质数的特性优化算法。