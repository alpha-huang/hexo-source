title: jQuery中的attr、prop、data方法对比
author: alpha
tags:
  - 前端
  - js
  - 基础知识
  - jquery
categories:
  - 笔记
date: 2018-04-29 21:12:34
---
<!--以下是正文-->
jQuery中的attr、prop、data方法区别如下：
1.attr用来读写HTML节点上的属性
2.prop用来读写DOM对象的属性以及状态相关的属性
3.data用来读写自定义的jquery属性

<!--more-->

``` javascript
<input type="checkbox" id="demo" class="demo_class" style="width:20px;" myattr="html_attr" data-test="demo_data">
```
上面的HTML标签中，带有id、class、style、myattr、data-test等属性，用三种方法读出来的结果分别是：

``` javascript
console.log($('#demo').attr('id'))//"demo"
console.log($('#demo').attr('type'))//"checkbox"
console.log($('#demo').attr('class'))//"demo_class"
console.log($('#demo').attr('className'))//undefined
console.log($('#demo').attr('style'))//"width:20px;"
console.log($('#demo').attr('myattr'))//"html_attr"
console.log($('#demo').attr('data-test'))//"demo_data"
console.log($('#demo').attr('checked'))//undefined

console.log($('#demo').prop('id'))//"demo"
console.log($('#demo').prop('type'))//"checkbox"
console.log($('#demo').prop('class'))//"demo_class"
console.log($('#demo').prop('className'))//"demo_class"
console.log($('#demo').prop('style'))//CSSStyleDeclaration
console.log($('#demo').prop('myattr'))//undefined
console.log($('#demo').prop('data-test'))//undefined
console.log($('#demo').prop('checked'))//false

console.log($('#demo').data('test'))//"demo_data"
```
可以看到，attr方法可以读出HTML标签上的所有属性，prop方法可以读出HTML标签的部分属性（id、class、name、type、checked、selected、disabled、selectedIndex、tagName、nodeName、nodeType、ownerDocument、defaultChecked 和 defaultSelected等），data方法可以读出HTML标签上以“data-”开头的属性。

此外，prop可以正确读出checked、selected等属性，并且在更新状态之后也能正确识别，而attr读不到或者读到的不正确。

接着我们用三种方法对这个DOM对象的属性进行设置，并分别用三种方式读取设置的属性。

``` javascript
$('#demo').attr('data-attr',1);
$('#demo').prop('data-prop',2);
$('#demo').data('data',3);

console.log($('#demo').attr('data-attr'))//1
console.log($('#demo').attr('data-prop'))//undefined
console.log($('#demo').attr('data-data'))//undefined

console.log($('#demo').prop('data-attr'))//undefined
console.log($('#demo').prop('data-prop'))//2
console.log($('#demo').prop('data-data'))//undefined

console.log($('#demo').data('attr'))//1
console.log($('#demo').data('prop'))//undefined
console.log($('#demo').data('data'))//3
```
可以发现用attr设置的属性将会直接反映到HTML标签上，其中prop原本就能读取的那些固有属性会同步更新，而自定义属性不会更新，data读到的自定义属性也不会更新。

prop无法修改固有属性，但是能修改checked、selected等，prop设置的属性将会被保存在DOM对象中，但是prop设置的属性不能被attr、data方法读取，data方法设置的属性也不能被attr、prop方法读取。

因此，如果是HTML标签上带有的属性，最好用attr方法来读写；自定义属性可以用data方法读写；checked、selected、disabled等状态属性只能用prop来读写。