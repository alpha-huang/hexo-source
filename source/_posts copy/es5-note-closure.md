title: 闭包的特征和应用场景
author: alpha
tags:
  - 基础知识
  - 前端
  - js
  - es5
categories:
  - 学习笔记
date: 2018-4-24 16:57:56
---
<!--以下是正文-->
## 一、定义
闭包就是能够读取其他函数内部变量的函数。我们知道js的作用域特点是外层函数无法读取内层函数的变量，但是内层函数可以读取外层函数的变量，所以这要求闭包一定要在其他函数内部。所以闭包可以定义为“在一个函数内部的函数”。

``` javascript
function outer(){
	var n = 1;
	function inner(){
	
	}
}
```
上面代码里的inner函数就是一个闭包。
<!--more-->
## 二、特征
闭包有两个很重要的特性

 1. 可以读取父函数内部的变量
 2. 可以在父函数执行完之后还保留其内部变量
 3. 在定义时只取得变量的引用，在执行时才会实际取值

第1点很容易理解，因为js的作用域的特点赋予闭包能访问父函数的内部变量。关于第2点，通常我们定义闭包之后不会立即执行，而是将其返回，等待合适的时机再执行。

``` javascript
function outer(){
	var n = 1;
	return function inner(){
		return n;
	}
}
var get_n = outer();
var n = get_n();
```
上面的例子，我们在父函数outer内部定义了一个闭包inner，在执行outer函数后，将get_n指向inner函数，当在外层执行get_n函数也就是inner函数时，才返回outer函数内部定义的n。

## 三、应用场景

根据闭包的特征，我们可以做到在函数外部取到内部的变量值，因此我们可以将某些不想要定义到全局的变量，用闭包来实现，类似于“私有变量”。

``` javascript
function outer(){
	var n = 1;
	return {
		get_n:function (){
			return n;
		},
		set_n:function (s){
			n = s;
		}
	}
}
var o = outer();
console.log(o.get_n());//1
o.set_n(2);
console.log(o.get_n());//1
```
上面的代码，封装了一个“私有变量”n，在外部无法直接读写，但是可以通过get_n和set_n两个函数来读写。

## 四、特别关注
闭包很常见的一个问题是

``` javascript
function outer(){
	var result = [];
	for(var i = 0;i < 3;i++){
		result[i] = function (){
			console.log(i);
		}
	}
	return result;
}
var o = outer();
o[0]();
o[1]();
o[2]();
```
上面的代码将会在控制台打印出3行3。这个其实很好理解，当执行“var o = outer();”这行代码时，得到的o这个数组里有三个元素，都是一个用来打印出i的函数。从上面闭包的第3个特征知道，定义这3个函数的时候，并没有取得i的值，只是保存了对i的引用，在实际执行前，i的值已经经过循环后变成了3，所以再执行返回出来的方法，都将打印出3。

所以尽量不要在闭包里使用循环。不过还是有办法解决这个问题的。

``` javascript
function outer(){
	var result = [];
	for(var i = 0;i < 3;i++){
		result[i] = (function (i){
			 return function (){
				console.log(i);
			}
		})(i);
	}
	return result;
}
var o = outer();
o[0]();
o[1]();
o[2]();
```
上面的代码将会分别打印0,1,2。和上一个例子相比，只是在闭包外多嵌套了一层立即执行的函数，在该函数执行的时候取到的值是当时的值，而内部的闭包取到的是立即执行的函数内的变量的引用，也就是当时的i（相当于复制了一个当时的i，已经不是outer内部定义的那个i的引用了），我们可以看看的代码的执行过程。

``` javascript
var result = [];
result[0] = (function (i){return function (){console.log(i);}})(0);//这里返回的闭包引用的变量i来自于立即执行函数的入参0，所以后面即使outer里的i变化了，也不会影响到闭包引用的变量i
result[1] = (function (i){return function (){console.log(i);}})(1);
result[2] = (function (i){return function (){console.log(i);}})(2);
result[0]();//等同于console.log(0);
result[1]();//等同于console.log(1);
result[2]();//等同于console.log(2);
```

参考资料：[带你一分钟理解闭包--js面向对象编程](https://www.cnblogs.com/qieguo/p/5457040.html)