title: 跨浏览器的事件监听方法
author: alpha
tags:
  - 基础知识
  - 前端
  - js
  - es5
categories:
  - 学习笔记
date: 2018-4-26 14:36:37
---
<!--以下是正文-->
要写一个兼容所有浏览器的事件监听方法，首先要清楚不同浏览器里的事件监听方法的区别。
注册和销毁事件监听的方法有三种：
1.addEventListener、removeEventListener：DOM2级方法，兼容IE9及以上和非IE浏览器
2.attachEvent、detachEvent：IE方法，兼容IE8及以下浏览器
3.on：DOM0级方法，兼容更早期的浏览器

另外DOM2级方法允许设置在事件捕获阶段触发（第三个参数设置为true），而IE方法只允许在冒泡阶段触发，因此为了保持统一，我们都在冒泡阶段触发。
<!--more-->
所以可以用如下代码实现

``` javascript
var EventUtil = {
	addHeadler: function(btn,type,handler){
		if(btn.addEventListener){
			btn.addEventListener(type,handler,false);
		}else if(btn.attachEvent){
			btn.attachEvent('on' + type,handler);
		}else{
			btn['on' + type] = handler;
		}
	},
	removeHeadler: function(btn,type,handler){
		if(btn.removeEventListener){
			btn.removeEventListener(type,handler,false);
		}else if(btn.detachEvent){
			btn.detachEvent('on' + type,handler);
		}else{
			btn['on' + type] = null;
		}
	}
};
```
关于事件对象，IE浏览器的事件对象在window上，而且只有在事件发生时才可访问，时间结束即被销毁，而非IE浏览器的事件对象会被作为参数传入回调函数。

关于事件目标，IE浏览器保存在事件对象的srcElement属性上，非IE浏览器保存在事件对象的target属性上。

关于阻止默认事件，IE浏览器需要设置事件对象上的returnValue属性为false，非IE浏览器可以调用事件对象上的preventDefault方法。

关于阻止事件冒泡，IE浏览器需要设置事件对象上的cancelBubble属性为true，非IE浏览器可以调用事件对象上的stopPropagation方法。

因此，可以对上面的事件监听方法进行完善，加入一些常用的方法。

``` javascript
var EventUtil = {
	addHeadler: function(btn,type,handler){
		if(btn.addEventListener){
			btn.addEventListener(type,handler,false);
		}else if(btn.attachEvent){
			btn.attachEvent('on' + type,handler);
		}else{
			btn['on' + type] = handler;
		}
	},
	removeHeadler: function(btn,type,handler){
		if(btn.removeEventListener){
			btn.removeEventListener(type,handler,false);
		}else if(btn.detachEvent){
			btn.detachEvent('on' + type,handler);
		}else{
			btn['on' + type] = null;
		}
	},
	getEvent: function(e){
		return e || window.event;
	},
	getTarget: function(e){
		return e.target || e.srcElement;
	},
	preventDefault: function(e){
		if(e.preventDefault){
			e.preventDefault();
		}else{
			e.returnValue = false;
		}
	},
	stopPropagation: function(e){
		if(e.stopPropagation){
			e.stopPropagation();
		}else{
			e.cancelBubble = true;
		}
	}
};
```