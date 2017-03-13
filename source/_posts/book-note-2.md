title: 《学习CSS布局》知识要点
author: alpha
tags:
  - 前端
  - css
  - 学习笔记
categories:
  - 笔记
date: 2015-9-6 14:58
---
《学习CSS布局》知识要点
<!--more-->
#### 知识点
- box-sizing
- 媒体查询
- column
- flexbox

##### box-sizing
当你设置一个元素为 box-sizing: border-box; 时，此元素的内边距和边框不再会增加它的宽度。

来自 <http://zh.learnlayout.com/box-sizing.html>


既然没有比这更好的方法，一些CSS开发者想要页面上所有的元素都有如此表现。所以开发者们把以下CSS代码放在他们页面上：
```javascript
* {
  -webkit-box-sizing: border-box;
     -moz-box-sizing: border-box;
          box-sizing: border-box;
}
```
这样可以确保所有的元素都会用这种更直观的方式排版。
既然 box-sizing 是个很新的属性，目前你还应该像我之前在例子中那样使用 -webkit-和 -moz- 前缀。这可以启用特定浏览器实验中的特性。同时记住它是支持IE8+。

来自 <http://zh.learnlayout.com/box-sizing.html>


##### 媒体查询
“响应式设计（Responsive Design）”是一种让网站针对不同的浏览器和设备“响应”不同显示效果的策略，这样可以让网站在任何情况下显示的很棒！
媒体查询是做此事所需的最强大的工具。让我们使用百分比宽度来布局，然后在浏览器变窄到无法容纳侧边栏中的菜单时，把布局显示成一列：
```javascript
@media screen and (min-width:600px) {
  nav {
    float: left;
    width: 25%;
  }
  section {
    margin-left: 25%;
  }
}
@media screen and (max-width:599px) {
  nav li {
    display: inline;
  }
}
```
来自 <http://zh.learnlayout.com/media-queries.html>


##### column
这里有一系列新的CSS属性，可以帮助你很轻松的实现文字的多列布局。让我们瞧瞧：
```javascript
.three-column {
  padding: 1em;
  -moz-column-count: 3;
  -moz-column-gap: 1em;
  -webkit-column-count: 3;
  -webkit-column-gap: 1em;
  column-count: 3;
  column-gap: 1em;
}
```
来自 <http://zh.learnlayout.com/column.html>


##### flexbox
定义两个可伸缩的 p 元素。如果父元素的总宽度是 300 像素，则 #p1 的宽度是 100 像素，而 #p2 的宽度是 200 像素：
```javascript
#p1
{
-moz-box-flex:1.0; /* Firefox */
-webkit-box-flex:1.0; /* Safari 和 Chrome */
box-flex:1.0;
border:1px solid red;
}
#p2
{
-moz-box-flex:2.0; /* Firefox */
-webkit-box-flex:2.0; /* Safari 和 Chrome */
box-flex:2.0;
border:1px solid blue;
}
```
来自 <http://www.w3school.com.cn/cssref/pr_box-flex.asp>
（个人感觉和bootstrap的栅格式布局很相似）

参考：
[学习CSS布局]([异步非阻塞](http://blog.csdn.net/feitianxuxue))
