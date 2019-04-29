title: js变量提升
tags:
  - js组件
categories:
  - 前端
author: alpha
date: 2016-12-15 17:18:00
---

在ES6之前，JavaScript没有块级作用域(一对花括号{}即为一个块级作用域)，只有全局作用域和函数作用域。变量提升即将变量声明提升到它所在作用域的最开始的部分。

### 例一
下面这段代码，结果是不是你预想的样子呢？试着运行一下吧！
<!--more-->

```javascript
var a =1;
function test(){
    console.log(a);
    a=4        
    console.log(a);
    var a;
    console.log(a);
}
test();
console.log(a);
```  


运行结果：


```javascript
undefined
4
4
1
```


为什么第3行的a会是undefined呢，因为这个a并不是全局变量。在function test里已经声明了（函数体倒数第2行）一个重名的局部变量，所以全局变量a被覆盖了，这说明了Javascript在执行前会对整个脚本文件的定义部分做完整分析,所以在函数test()执行前,函数体中的变量a就被指向内部的局部变量.而不是指向外部的全局变量。但这时a只有声明，还没赋值，所以输出undefined。



用另一种简单的方式来理解就是
```javascript
//var a = xxx; 可以理解为两个操作
//var a;
//a = xxx;
//凡是var xx的语句，都会提高到函数作用域的最顶端
//所以，那段代码翻译一下，执行的顺序是：
var a;
a = 1;
function test(){
  var a
  console.log(a);
  a=4
  console.log(a);
  console.log(a);
}
test();
console.log(a);
```



现在是不是很清晰了呢？


### 例二
再来看另一个例子
```javascript
var a = 1;
function getFood(m) {
  if (m) {
      var a = 2;
      return a;
  }
  return a;
}
getFood(m);
```

运行后发现结果是

```javascript
undefined
```

`if`语句虽然没执行，但由于var的变量提升，内部的var a会被提升到函数的最前面，相当于
```javascript
var a = 1;
function getFood(m) {
  var a
  if (m) {
      a = 2;
      return a;
  }
  return a;
}
getFood(m);
```
使用ES6的`let`和`const`来进行变量声明就不会有变量提升。