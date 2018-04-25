title: 构造函数的原型重写后实例的构造函数指向错误
author: alpha
tags:
  - 基础知识
  - 前端
  - js
  - es5
categories:
  - 学习笔记
date: 2018-4-24 10:55:52
---
<!--以下是正文-->
## 一、问题描述
《JavaScript高级程序设计（第三版）》第164页，讲述了原型链继承的方法，将一个构造函数的原型指向另一个父构造函数的实例，这时子构造函数产生的实例，其构造函数不再是子构造函数，而是父构造函数了。
<!--more-->
``` javascript
function Sup(){

}

function Sub(){

}

Sub.prototype = new Sup();

var instance = new Sub();

console.dir(instance.constructor);//f Sup()
```

instance分明是子构造函数Sub的实例，但其constructor属性却指向Sup，这是为什么呢？

## 二、问题分析

我们知道构造函数和实例之间本来没有直接联系，只是实例里的__proto__属性指向构造函数的原型，因此会有

``` javascript
function Sub(){

}
var instance = new Sub();
console.log(instance.__proto__ === Sub.prototype);//true
```
而在构造函数的属性prototype上保存着一个指向原构造函数的constructor属性，这样我们获取实例的constructor属性时，实例本身没有这个属性，于是去原型上取，取到的constructor自然就是构造函数Sub。

在最上面的例子里，子构造函数Sub的原型已经被改写了，不再有constructor属性了，因此实例和构造函数Sub之间已经没有联系了。这时再取instance.constructor的过程是：

 1. instance本身不存在constructor属性，于是去原型prototype上取；
 2. 原型由于被重写为Sup的实例，也没有了constructor属性，于是继续去Sup的原型prototype上取;
 3. Sup的原型prototype上保存着指向构造函数Sup的指针constructor，因此返回的自然就是Sup。

所以这时候的instance.constructor已经取不到正确的构造函数了。
## 三、解决方案
我们可以通过修改Sub的原型让实例和构造函数重新建立联系

``` javascript
function Sup(){

}

function Sub(){

}

Sub.prototype = new Sup();
Sub.prototype.constructor = Sub;

var instance = new Sub();

console.dir(instance.constructor);//f Sub()
```
上面的代码在修改子构造函数的原型后重新为其添加指向子构造函数的constructor属性，就可以了。

相关问题：[csdn论坛](https://bbs.csdn.net/topics/390850551)