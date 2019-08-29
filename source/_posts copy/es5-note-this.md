title: js中的this
author: alpha
tags:
 - 基础知识
 - 前端
 - js
 - es5
categories:
 - 学习笔记
date: 2018-4-25 10:37:30
---
<!--以下是正文-->
## 一、什么是this
js中的this实际上保存着代码的执行环境，关于js指向问题，有下面几种情况：
1.函数中的this总是指向调用它的对象，当没有明确的调用对象的时候this指向window（非严格模式下）（其实是js引擎调用的，也可以理解为是window调用的）；
2.构造函数中的this指向构造函数的实例；
3.js事件回调函数里this指向dom节点对象；
4.使用apply、call、bind方法可以改变this的指向。

<!--more-->
## 二、普通对象里的this
先看下面的例子

``` javascript
function f(){
	console.dir(this);
	function i(){
		console.dir(this);
	}
	i();//window
}
f();//window
```
上面的例子中f、i都没有明确的调用对象（其实是引擎调用的，也可以理解为是window调用的），所以this指向window。

``` javascript
function f(){
	console.log(this);
}
var o = {
	n:'o',
	f:f
}
o.f();//{n: "o", f: ƒ}
```
上面的例子，“o.f()”表示是由对象o调用的f，所以f中的this指向o。

## 三、构造函数中的this

``` javascript
var that;
function F(name){
	that = this;
}
var o = new F('alpha');
console.log(o === that);//true
```
上面的例子表明构造函数里的this指向构造函数产生的实例。

有在其它资料里看到一个理论：如果构造函数中有return 对象，那么this指向该对象，我觉得这个说法是错误的。（[彻底理解js中this的指向，不必硬背。](https://www.cnblogs.com/pssp/p/5216085.html)）在这篇文章里作者举例

``` javascript
function fn()  
{  
    this.user = '追梦子';  
    return {};  
}
var a = new fn;  
console.log(a.user); //undefined
```
如果没有return返回的实例应该是有user属性的，于是得出结论说this是指向返回的对象的，这个是站不住脚的。我们可以看下面的代码

``` javascript
function F(name){
	return {n:'alpha',that:this}
}
var o = new F('alpha');
console.log(o);//{n: "alpha", that: F}
```
从结果可以看出，this还是指向F产生的实例的，只是构造函数返回的不是其生成的实例，而是返回了return语句的返回值。

## 四、js事件回调函数里的this

``` javascript
var btn = document.getElementsByTagName('body')[0];
btn.onclick = function () {
	console.log(this === btn);//true
}
```
上面的事件绑定，给btn对象新增了一个方法，当事件被触发的时候自然会执行btn.onclick，因此this指向btn是顺理成章的。下面的事件监听方法的回调函数里的this又是什么情况呢？
``` javascript
var btn = document.getElementsByTagName('body')[0];
btn.addEventListener('click',function(){
	console.log(this === btn);//true
},false);
```
上面的代码，给btn元素绑定了一个click事件，当事件触发时打印出的结果表明this指向btn对象。其实我们可以将其归到第一种情况：“函数中的this总是指向调用它的对象”。上面的事件监听方法给btn添加了一个“click”方法，当监听到click事件时，btn对象会调用其“click”方法（直接执行btn.click()方法可以模拟点击事件），于是this自然指向调用它的btn对象。

## 五、apply、call、bind方法里的this
``` javascript
function f(){
	console.log(this);
}
f();//window
f.apply({n:'alpha'});//{n: "alpha"}
f.call({n:'alpha'});//{n: "alpha"}
f.bind({n:'alpha'})();//{n: "alpha"}
```
上面的例子通过apply改变了函数中this的指向


## 六、几个实际例子
### 1.jquery的ajax，this的指向问题
``` javascript
var obj = {
    self: this,
    type:"get",
    url: url,
    async:true,
    success: function (res) {
        console.log(this) // this指向传入$.ajxa()中的对象
        console.log(self) // window
    }
};
$.ajax(obj);
```
上面的例子是jquery里ajax方法的调用，第二行的this来自全局环境window，自然是指向window，但success为什么会指向obj呢？由于传入的obj中包含的success方法，会在请求成功时被调用，可以想象，调用的时候执行的应该是obj.success(res)，因此，根据第1种情况所说success里的this指向的是调用它的obj对象，也就是这里传入$.ajxa()中的对象。

### 2.请说明要输出正确的myName的值要如何修改程序?并解释原因
``` javascript
foo = function(){
	this.myName = "Foo function.";
}
foo.prototype.sayHello = function(){
	alert(this.myName);
}
foo.prototype.bar = function(){
	setTimeout(this.sayHello, 1000);
}
var f = new foo;
f.bar()
```
这个例子里倒数第二行产生了一个实例，构造函数foo为该实例赋予myName属性。最后一行调用f.bar()时，直接调用对象是f，因此setTimeout里的this指向的是f。sayHello函数会在1000毫秒后由js引擎调用，因此执行的时候函数里的this会指向window，这里显然是想弹出f里的myName属性，我们只需要让sayHello执行的时候this指向f就好了。可以将bar那句改写成如下形式

``` javascript
setTimeout(this.sayHello.bind(this), 1000);
setTimeout(this.sayHello.bind(f), 1000);
```

参考链接：
[彻底弄懂js中的this指向](https://blog.csdn.net/qq_33988065/article/details/68957806)
[JS中的this 在不同的地方指向不一样，在哪些地方需要注意？](https://www.zhihu.com/question/25842198)