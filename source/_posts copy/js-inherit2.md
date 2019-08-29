title: js继承
author: alpha
tags:
  - 基础知识
  - 前端
  - js
  - es5
categories:
  - 学习笔记
date: 2019-4-30 19:53:52
---
<!--以下是正文-->
js中的继承，是指让一个对象拥有另一个对象的属性和方法。而实现继承主要有以下六种方式
- 原型链继承
- 借用构造函数继承
- 组合继承
- 原型式继承
- 寄生式继承
- 寄生组合式继承


<!--more-->
## 原型链继承

其原理是通过将子构造函数的原型指向父构造函数的实例从而继承父构造函数的属性

``` javascript
function SuperType(){
    this.colors = ["red", "blue", "green"];
}

function SubType(){            
}

//inherit from SuperType
SubType.prototype = new SuperType();

var instance1 = new SubType();
instance1.colors.push("black");

var instance2 = new SubType();
console.log(instance2.colors);    //"red,blue,green,black"
```
原型链继承有个缺点：
- 对于引用类型的属性，所有子构造函数和父构造函数的实例都共享同样的属性

## 借用构造函数继承

其原理是在子构造函数内部调用父构造函数（使用call或apply），子构造函数的实例将会有各自独立的属性

``` javascript
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    console.log(this.name);
};

function SubType(name, age){  
    SuperType.call(this, name);
    this.age = age;
}

var instance1 = new SubType("Nicholas", 29);
```

借用构造函数继承有两个缺点：
- 缺点1：子构造函数的实例都将拥有各自独立的属性，方法没法复用；
- 缺点2：无法继承父构造函数的原型属性

## 组合继承

其原理是用原型链来继承原型属性，用借用构造函数来继承实例属性

``` javascript
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    console.log(this.name);
};

function SubType(name, age){  
    SuperType.call(this, name);
    
    this.age = age;
}

SubType.prototype = new SuperType();

SubType.prototype.sayAge = function(){
    console.log(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
console.log(instance1.colors);  //"red,blue,green,black"
instance1.sayName();      //"Nicholas";
instance1.sayAge();       //29


var instance2 = new SubType("Greg", 27);
console.log(instance2.colors);  //"red,blue,green"
instance2.sayName();      //"Greg";
instance2.sayAge();       //27
```

组合继承融合了两种方法的优点，弥补了对方的缺陷，但引进了新的问题：
- 调用了两次父构造函数，导致子构造函数的原型上有冗余属性，其实原型链继承的时候本不必调用父构造函数

## 原型式继承

其原理是将子对象的原型指向父对象，没有严格意义上的构造函数（在把父对象赋值给子对象原型的过程中实际上用到了一个临时的构造函数）

``` javascript
function object(o){
    function F(){}
    F.prototype = o;
    return new F();
}

var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

console.log(person.friends);   //"Shelby,Court,Van,Rob,Barbie"
```
在ES6中，有Object.create()方法可以产生一个以传入的参数作为__proto__属性的对象，可以用来替代上面方法里的object()
``` javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = Object.create(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

console.log(person.friends);   //"Shelby,Court,Van,Rob,Barbie"
```

原型式继承的缺点是：
- 引用类型的属性会共享

## 寄生式继承

其原理是在原型式继承的基础上附加属性来增强对象

``` javascript
function SubType(obj){
    var clone = Object.create(obj);   //create object
    clone.sayHi = function(){  //augment object
        console.log('Hi')
    }
    return clone
}

var instance1 = new SubType({name: 'Nicholas'});
console.log(instance1.name);      //"Nicholas";
instance1.sayHi();       //"Hi"
```

寄生式继承的缺点是：
- 每个对象都创建了方法，函数无法复用

## 寄生组合式继承

其原理是使用寄生式继承来复制原型对象并进行增强，使用组合继承来继承增强过的原型对象和实例属性

``` javascript
function inheritPrototype(subType, superType){
    var prototype = Object.create(superType.prototype);   //create object
    prototype.constructor = subType;               //augment object
    subType.prototype = prototype;                 //assign object
}
                        
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    console.log(this.name);
};

function SubType(name, age){  
    SuperType.call(this, name);
    
    this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function(){
    console.log(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
console.log(instance1.colors);  //"red,blue,green,black"
instance1.sayName();      //"Nicholas";
instance1.sayAge();       //29


var instance2 = new SubType("Greg", 27);
console.log(instance2.colors);  //"red,blue,green"
instance2.sayName();      //"Greg";
instance2.sayAge();       //27
```

寄生组合式继承结合了上面所有继承方式的优点，规避了其缺点，是最为合理的一种继承方式。

## 总结

以上六种继承方式，原理和特点如下

|继承方式| 原理 | 特点 |
| ------ | ------ | ------ |
|原型链继承	|通过将子构造函数的原型指向父构造函数的实例从而继承父构造函数的属性	|（缺点：对于引用类型的属性，所有子构造函数和父构造函数的实例都共享同样的属性）|
|借用构造函数继承	|在子构造函数内部调用父构造函数（使用call或apply），子构造函数的实例将会有各自独立的属性	|（解决了上面的缺点，但引进了新的问题；缺点1：子构造函数的实例都将拥有各自独立的属性，方法没法复用；缺点2：无法继承父构造函数的原型属性）|
|组合继承	|用原型链来继承原型属性，用借用构造函数来继承实例属性	|（融合了两种方法的优点，弥补了对方的缺陷，但引进了新的问题；缺点：调用了两次父构造函数，导致子构造函数的原型上有冗余属性，其实原型链继承的时候本不必调用父构造函数）|
|原型式继承	|将子对象的原型指向父对象，没有严格意义上的构造函数（在把父对象赋值给子对象原型的过程中实际上用到了一个临时的构造函数）	|（缺点：引用类型的属性会共享）|
|寄生式继承	|在原型式继承的基础上附加属性来增强对象	|（缺点：每个对象都创建了方法，函数无法复用）|
|寄生组合式继承	|使用寄生式继承来复制原型对象并进行增强，使用组合继承来继承增强过的原型对象和实例属性	||

他们的关系如下图所示

 ![继承][1]

[1]: /images/2019043016580001.png "继承"