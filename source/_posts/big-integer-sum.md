title: 大整数求和
author: alpha
tags:
  - js
  - 算法
categories:
  - 面试题
date: 2019-11-07 16:18:00
---

众所周知，计算机计算的数字是有上限的，js里可以精确表示的最大值是 Number.MAX_SAFE_INTEGER，那超过这个值的两个整数求和，要怎么计算呢？
<!--more-->

整数的加法无非就是从低位到高位按位相加，满10进1，按正常思维，只需要把大整数的每一位分别计算即可。
```javascript
function plus(a,b){
	let m = a.split(''), n = b.split('')
	let result = '', flag = 0
	while(m.length || n.length || flag){
		let temp = Number(m.pop() || 0) + Number(n.pop() || 0) + Number(flag)
		result = '' + temp%10 + result
		flag = temp > 9//如果满10需要进1，就把flag置为true，转化为Number后就是1
	}
	return result
}
plus('1232321412312312312313124124','987987979798797879879797')//"1233309400292111110193003921"
```
由于Number.MAX_SAFE_INTEGER的值有16位，其实至少可以每10位或者15位相加也是可以的。

```javascript
function plus(a,b){
	let m = a.split(''), n = b.split('')
	let result = '', flag = 0
	while(m.length || n.length || flag){
		let temp = Number(m.splice(-10,10).join('') || 0) + Number(n.splice(-10,10).join('') || 0) + Number(flag)
		result = '' + temp%Math.pow(10,10) + result
		flag = temp > 9
	}
	return result
}
```
实际测试后发现，使用这种方法虽然循环的次数变少了，但是并不会更高效，反而会耗时更多,所以我们还是采用第一种方法进行大整数的计算即可。