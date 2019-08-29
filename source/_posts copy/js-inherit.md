title: 组合继承和寄生组合继承
author: alpha
tags:
  - 基础知识
  - 前端
  - js
  - es5
categories:
  - 学习笔记
date: 2019-4-24 19:53:52
---
<!--以下是正文-->
js继承中的组合继承和寄生组合继承有什么区别呢？js中的继承分为两个部分，对实例的继承和对原型的继承。可以用原型来继承公用的属性和方法，当原型上的属性和方法改变时，继承关系链上的所有对象都将受到影响；用实例来继承特有的属性和方法，当实例上的属性和方法发生改变时，继承关系链上的所有对象互不影响。

<!--more-->
## 组合继承
``` javascript
function Sup(grade){
    this.grade = grade
}
Sup.prototype.sayGrade = function (){
    console.log(this.grade)
}

function Sub(name){
    Sup.call(this,'一3班')
    this.name = name
}

Sub.prototype = new Sup('一3班');
Sub.prototype.constructor = Sub;

var instance = new Sub('小强');

console.dir(instance);
console.dir(instance.sayGrade());
```
运行结果如下

 ![组合继承][1]

可以看到实例上确实已经有了继承自`Sup`的`grade`属性和由`Sub`产生的`name`属性，在实例上执行`sayGrade`方法也能正确打印出结果。但可以看到由于`Sup`构造函数实例化了两次，在`Sub`的原型上也出现了`grade`属性。由于在实例上获取属性时会按原型链逐级往上找，而在实例上就已经获取到了`grade`属性，所以通常情况下原型上的这个属性不会生效，拥有这个“多余的”属性也无伤大雅。但在某些情况下会造成错误，例如删除实例上的`grade`属性，实际上还能访问到，此时获取到的是原型上的属性。

问题就出在第一次的`new Sup('一3班')`上，只是为了继承原型上的实例和方法其实不必要执行该方法，寄生组合继承就可以解决这个问题。

## 寄生组合继承
``` javascript
function Sup(grade){
    this.grade = grade
}
Sup.prototype.sayGrade = function (){
    console.log(this.grade)
}

function Sub(name){
    Sup.call(this,'一3班')
    this.name = name
}

Sub.prototype = Object.create(Sup.prototype);
Sub.prototype.constructor = Sub;

var instance = new Sub('小强');

console.dir(instance);
console.dir(instance.sayGrade);
```
运行结果如下

 ![寄生组合继承][2]

 可以看到，`Sub`的原型上已经没有`grade`属性，这是因为`Object.create(Sup.prototype)`方法返回一个以`Sup.prototype`为原型的对象，而不用执行`Sup`方法。

 显然寄生组合继承要比组合继承更加合理。


[1]: /images/20190424200901.png "组合继承"
[2]: /images/20190424202901.png "寄生组合继承"