title: 闭包的应用
author: alpha
tags:
  - 基础知识
  - 前端
  - js
  - es5
  - 算法
categories:
  - 学习笔记
date: 2019-4-29 15:33:56
---
<!--以下是正文-->

关于闭包的定义，已经在另一篇文章 [闭包的特征和应用场景](/2018/04/24/es5-note-closure/) 中介绍了，这里想来分析一个例子。

> 写一个go方法，要求：
> 执行`go('l')`返回`gol`,
> 执行`go()('l')`返回`gool`,
> 执行`go()()('l')`返回`goool`。

## 一、问题分析

按照题干的描述，`go`方法可以选择是否传参数，传参数时返回字符串，不传参数返回另一个函数，而且这个函数还可以再次调用生成另一个函数，但每调用一次，最后返回的字符串中间会多一个`o`。那就说明有一个变量在函数执行结束后仍然被保留，很容易联想到闭包。

<!--more-->

## 二、简化模型

在学习闭包时有个很常见的例子

> 写一个`plus`方法，要求每次调用时，返回的结果在上次的基础上加一。

``` javascript
function outer(){
	var n = 0;
	return function inner(){
		return n++
	}
}
let plus = outer()
plus()//0
plus()//1
plus()//2
```
上面例子中`inner`函数里保留有`outer`函数里的变量`n`，执行`outer`函数后，将`plus`指向返回的`inner`函数，每次执行`plus`都会对同一个变量n进行自增操作。

在上一个例子的基础上，我们再来解决另一个问题

> 写一个`plus`方法，要求：
> 执行`plus()`打印出`0`，
> 执行`plus()()`打印出`1`，
> 执行`plus()()()`打印出`2`。

和上面的例子一样，只不过返回值从数字，变成了函数。
``` javascript
function plus(){
	let n = 0;
	console.log(n)
	return function inner(){
		n++
		console.log(n)
		return inner;
	}
}
plus()//0
plus()()//0 1
plus()()()//0 1 2
```
调用`plus`会返回一个新的函数，而函数里的n却是一直保留，执行新的函数时对n自增并返回另一个函数。回到开头的那个问题，也是这个原理，只不过根据是否传参而执行不同的操作罢了。

## 三、解决问题

`go`方法判断是否有传入参数，如果没传参数，修改内部变量并返回新的方法，否则返回字符串。
``` javascript
function go(){
	let n = 'go';
	if(arguments.length){
		return n + arguments[0]
	}
	return function inner(){
		n += 'o'
		if(arguments.length){
			return n + arguments[0]
		}
		return inner;
	}
}
go('l')//"gol"
go()('l')//"gool"
go()()('l')//"goool"

```

其实核心在于怎么在多次调用一个方法时保留和传递变量，闭包就是一种很好的方法。除了可以利用闭包来保留变量，还可以用作用域链来传递变量。我们可以用`call`、`apply`、`bind`来修改作用域，从而将变量从一个函数传递到另一个函数。

``` javascript
function go(){
	if(!this.str){
		this.str = 'go'
	}
	if(arguments.length){
		return this.str + arguments[0]
	}else{
		return go.bind({str: this.str + 'o'})
	}
}

go('l')//"gol"
go()('l')//"gool"
go()()('l')//"goool"
```
在上面的方法中，用`bind`方法改变`go`的作用域，使其每次调用的作用域中保持同一个`str`变量。但是上面的代码有个问题，那就是第一次调用`go`函数的时候，this是指向window的，给this添加`str`属性会污染window，如果恰好window中正好存在`str`属性，甚至还会出错，所以我们可以在初次调用`go`函数时就为其指定作用域。

``` javascript
function go(){
 return this === window ? go.call('go', arguments[0]) : (arguments[0] ? this + arguments[0] : go.bind(this + 'o'))
}

go('l')//"gol"
go()('l')//"gool"
go()()('l')//"goool"
```
在上面的函数中，首先根据`this === window`判断是否是第一次执行`go`方法，第一次执行时用`call`方法为其制定作用域，在这里直接将“go”字符串作为其作用域，以后每次执行都将“go”字符串拼接“o”后作为新的作用域。

## 四、总结

这题的核心在于在多次执行函数时保留变量，而保留变量的方法除了闭包还有作用域、全局变量、原型链等。作为一个函数为了保持健壮性最好是不依赖和污染外部环境，所以利用闭包和作用域是比较合适的了。