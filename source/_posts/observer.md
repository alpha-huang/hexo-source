title: 观察者模式（发布订阅模式）
author: alpha
tags:
  - js
  - 设计模式
categories:
  - 面试题
date: 2019-11-07 19:12:00
---

观察者模式又称为发布订阅模式，是将事件的触发和接收解耦的一种设计模式。
<!--more-->

观察者模式中有两个对象和三个动作，两个对象分别是观察者和订阅者，三个动作是消息订阅、取消订阅和消息投送。当订阅者进行消息订阅之后，如果观察者监控到有事件被触发将会把消息投放发到订阅者。

下面来写一个最简单的观察者模式的模型

```javascript
let subscribers = []//订阅者列表
function subscribe(func){//消息订阅
	subscribers.push(func)
}
function publish(param){//消息投送
	subscribers.forEach(func=>{
		func(param)
	})
}
function say(str){//订阅者
	console.log(str)
}
subscribe(say)
publish('hello')//hello
```
上面的代码非常简单，用一个队列`subscribers`来储存消息订阅者，订阅消息的过程就是通过`subscribe`方法把订阅者的回调函数加到`subscribers`里，使用`publish`方法来投送消息，具体方法就是依次调用`subscribers`里的所有函数。

这就是最简单的观察者模式了，实际上不同的订阅者可能订阅不同的消息类型，另外也还需要有取消订阅的方法。

```javascript
var Event = {
	on: function (eventName, callback){
		if(!this.observer)this.observer = {}
		if(!this.observer[eventName])this.observer[eventName] = []
		this.observer[eventName].push(callback)
	},
	emit: function (eventName){
		if(this.observer[eventName]){
			let parame = [...arguments].slice(1)
			this.observer[eventName].forEach(observer=>{
				observer(parame)
			})
		}
	},
	off: function (eventName, callback){
		if(this.observer[eventName]){
			this.observer[eventName].forEach((func,i,arr)=>{
				if(func === callback){
					this.observer[eventName].splice(i,1)
				}
			})
		}
		return this
	}
}
Event.on('test',function(result){
	console.log(result)
})
Event.emit('test','hello world')
```
最后用ES6的class来实现一遍

```javascript
class PubSub{
	constructor (){
		this.observer = {}
	}
	on(event, callback){
		if(!this.observer[event])this.observer[event] = []
		if(typeof callback !== 'function'){
			throw new Error('回调函数错误')
		}else{
			this.observer[event].push(callback)
        }
		return this
	}
	emit(event){
		if(this.observer[event]){
			this.observer[event].forEach(func=>{
				func([...arguments].slice(1))
			})
		}
		return this
	}
	off(event, callback){
		if(this.observer[event]){
			this.observer[event].forEach((func,i,arr)=>{
				if(func === callback){
					this.observer[event].splice(i,1)
				}
			})
		}
		return this
	}
}
let callback = function(test){
	console.log(test)
}
let pubsub = new PubSub()
pubsub.on('test',callback)
pubsub.emit('test',1111)
pubsub.off('test',callback)
```
观察者模式可以降低内存消耗，提高互动性能，减少系统开销，提高可维护性，但是创建可观察的对象需要花时间，所以还是需要根据实际情况判断是否适用。