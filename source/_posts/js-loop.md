title: js里常见的循环
author: alpha
tags:
  - 基础知识
  - 前端
  - js
categories:
  - 学习笔记
date: 2018-4-28 15:29:23
---
<!--以下是正文-->
## 一、目录

* [1.基本循环for，while，do...while]
	* [for](#for)
	* [while](#while)
	* [do...while](#dowhile)
* [2.适用于数组的循环forEach，map，for...of]
	* [forEach](#foreach)
	* [map](#map)
	* [for...of](#forof)
* [3.适用于对象的for...in]
	* [for...in](#forin)
* [4.其他组件库里的循环]
	* [$.each](#each)
	* [v-for...in](#v-forin)
	* [wx:for](#wxfor)

<!--more-->
## 二、示例
js里常见的循环有一下几种：
### 1.基本循环for，while，do...while
#### for

``` javascript
for(var i = 0; i < 10; i++){
	console.log(i);
}
```

#### while

``` javascript
var i = 0;
while(i < 10){
	console.log(i++);
}
```

#### do...while

``` javascript
var i = 0;
do{
	console.log(i++);
}while(i < 10);
```
### 2.适用于数组的循环forEach，map，for...of
#### forEach

``` javascript
var arr = [1,2,3];
arr.forEach(function(val,index,array){
	console.log(val,index,array);
})
```

#### map

``` javascript
var arr = [1,2,3];
var obj = {a:1};
arr.map(function(val,index,array){
	console.log(val,index,this);//这里的this指向map的第二个参数
},obj);
```

#### for...of

``` javascript
var arr = [1,2,3];
for(var val of arr){
	console.log(val);
}
```
for...of除了可以用于遍历数组，也可以遍历Set,Map等任何有iterator接口的对象

``` javascript
var map = new Map();
map.set('a',1);
map.set('b',2);
for(var [key,val] of map){
	console.log(key,val);//a 1 / b 2
}
```

### 3.适用于对象的for...in
#### for...in

``` javascript
var obj = {a:1,b:2,c:3};
for(var key in obj){
	console.log(key,obj[key]);
}
```
### 4.其他组件库里的循环

#### $.each

``` javascript
var obj = {a:1,b:2,c:3};
$.each(obj,function(key,val){
	console.log(key,val,this);//这里this指向val
});
```


#### v-for...in

``` javascript
v-for="(item,index) in arr"
```

#### wx:for

``` javascript
wx:for="{{arr}}"//index,item
```
## 三、总结
### 1.关于参数
可以看到大多数循环都有参数，而且参数的顺序还不太一样，但是对数组来说，遍历的时候index不太必要，因此作为可选参数，都是在val的后面，例如forEach，map；而对于对象来说key比较重要，因此对象的遍历一般key在前面，例如for...in，$.each。


### 2.关于this
可以看到jquery的$.each里this指向循环对象的每一个子对象，而map方法允许传入第二个参数用来指定this的指向。

### 3.关于continue，break，return
三种基本循环以及for...of、for...in都能正常使用continue，break，return；但forEach、map、$.each都是遍历数组或对象对其执行回调函数，所以没办法正常使用continue，break。各种遍历方法对continue，break，return的处理方式如下：

| 方法       | continue | break    | return | 备注                                      |
| ---------- | -------- | -------- | ------ | ----------------------------------------- |
| for        | 正常     | 正常     | 正常   |                                           |
| while      | 正常     | 正常     | 正常   |                                           |
| do...while | 正常     | 正常     | 正常   |                                           |
| forEach    | 语法错误 | 语法错误 | 正常   | return后的语句不执行                      |
| map        | 语法错误 | 语法错误 | 正常   | return后的语句不执行                      |
| for...of   | 正常     | 正常     | 正常   |                                           |
| for...in   | 正常     | 正常     | 正常   |                                           |
| $.each     | 语法错误 | 语法错误 | 正常   | return后的语句不执行                      |

注意不要在while、do...while的i++前使用continue，否则会死循环。