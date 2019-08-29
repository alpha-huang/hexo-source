title: 三数之和
author: alpha
tags:
  - 前端
  - js
  - 算法
categories:
  - 面试题
date: 2019-4-30 14:39:56
---
<!--以下是正文-->

> 写给定一个包含n个整数的数组nums，判断nums中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？找出所有满足条件且不重复的三元组。
> 注意：答案中不可以包含重复的三元组。
> 例如, 给定数组 nums = [-1, 0, 1, 2, -1, -4]，
> 满足要求的三元组集合为：
> [
>  [-1, 0, 1],
>  [-1, -1, 2]
> ]

<!--more-->
　　
## 一、问题分析

在集合中分别不重复地取3个数进行组合，要求他们的和为0，最直接的方法就是三层循环取3个数，然后判断它们的和是否为0，如果满足条件并且不重复，就将其取出放入结果集合中。但是这里也有一些可以优化的地方
1.第一次取完a，取b的时候可以直接从a后面开始取，c可以直接从b后面开始取，可以减少重复取值；
2.两层循环之后可以将`a+b+c=0`转化为`-(a+b)=c`，只需要两层循环取两个值，然后用`indexOf()`查看剩下的数里有没有`-(a+b)`
3.在对比三元组是否重复时，如果每个数两两比较，比较复杂，可以对其进行排序并转化为字符串比较
4.在最开始就对原数组进行升序排序后可以发现，`a<=b<=c`，如果a大于0，则三数之和一定大于0，就不用往后判断了；并且在对比是否重复时也可以不用重新排序了

## 二、算法实现
根据以上分析，代码如下
``` javascript
let data = [-1, 0, 1, 2, -1, -4];
let nums = [...data].sort((m,n)=>(m-n))
let result = [];
for(var i = 0; i < nums.length; i++){
	if(nums[i]>0) break;//如果最小的数大于0，和不可能为0
	for(var j = i+1; j<nums.length; j++){
		let index = nums.indexOf(-nums[i]-nums[j],j+1);
		index !== -1 
			&& noRepeat([nums[i],nums[j],nums[index]],result)
			&& result.push([nums[i],nums[j],nums[index]]);
	}
}
function noRepeat(a,b){
	if(b.length === 0)return true
	for(item of b){
		if(item.join(',') === a.join(',')){
			return false
		}
	}
	return true
}
console.log(result)//[[-1, -1, 2],[-1, 0, 1]]
```

## 四、总结

这题的逻辑并不复杂，但是对于重复数组的处理需要一些技巧，直接对两个数组中的各个元素两两进行比较是比较复杂的，但是排序之后再对比就简单很多了。另外，转换成字符串之后再比较也可以简化算法，但是值得注意的是，如果是先把所有满足条件的数组放入结果集合，然后对其进行去重，处理起来会麻烦一些，还需要再将字符串转回数组，但是在将三元组放入结果集合前就先判断是否结果集中已经存在相同的三元组就会简单一些了。