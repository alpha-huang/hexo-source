title: js常用方法
author: alpha
tags:
 - 基础知识
 - 前端
 - js
 - 方法库
categories:
 - 学习笔记
date: 2018-4-27 15:59:06
---
<!--以下是正文-->
## 目录

	* [1.es6 keys()](#1es6-keys)
	* [2.判断空对象](#2判断空对象)
	* [3.判断数组是否存在某元素](#3判断数组是否存在某元素)
	* [4.深拷贝](#4深拷贝)
	* [5.复制对象的指定字段](#5复制对象的指定字段)
	* [6.时间戳转时间](#6时间戳转时间)
	* [7.时间转时间戳](#7时间转时间戳)
	* [8.获取url参数](#8获取url参数)
	* [9.jquery ajax](#9jquery-ajax)

<!--more-->
## 方法

#### 1.es6 keys()

``` javascript
var keys = function(obj) {
    var arr = []
    for (var index in obj) {
        if (obj.hasOwnProperty(index)) {
            arr.push(index)
        }
    }
    return arr
}
```
#### 2.判断空对象
``` javascript
var isEmptyObject = function(obj){
    for(var n in obj){return false}
    return true;
}
```

#### 3.判断数组是否存在某元素
``` javascript
var inArray = function(value, array) {
    return array.indexOf(value) != -1
}
```
#### 4.深拷贝
``` javascript
var deepCopy = function(source) {
    var result = {};
    for (var key in source) {
        result[key] = typeof source[key] === 'object' ? deepCopy(source[key]) : source[key];
    }
    return result;
}
```

#### 5.复制对象的指定字段
``` javascript
var copyObj = function(obj, keys) {
    var newObj = {}
    for (var key of keys) {
        newObj[key] = obj[key]
    }
    return newObj
}
```
#### 6.时间戳转时间
``` javascript
var formatDate = function(timestamp,strEmpty) {
    if (timestamp) {
        timestamp = (timestamp + '').length <= 10 ? parseInt(timestamp + '000') : parseInt(timestamp);
        var time = new Date(timestamp);
        var year = time.getFullYear();
        var month = time.getMonth() + 1;
        var date = time.getDate();
        var hour = time.getHours();
        var minute = time.getMinutes();
        var second = time.getSeconds();
        month.toString().length < 2 && (month = '0' + month.toString());
        date.toString().length < 2 && (date = '0' + date.toString());
        hour.toString().length < 2 && (hour = '0' + hour.toString());
        minute.toString().length < 2 && (minute = '0' + minute.toString());
        second.toString().length < 2 && (second = '0' + second.toString());

        return year + "-" + month + "-" + date + " " + hour + ':' + minute + ':' + second;
    } else {
        return strEmpty ? strEmpty : '';
    }
}
```
#### 7.时间转时间戳

``` javascript
var timeStamp = function(stringTime) {
    var timestamp = stringTime ? Date.parse(new Date(stringTime)) : 0;
    return timestamp / 1000; //2014-07-10 10:21:12的时间戳为：1404958872
}
```

#### 8.获取url参数
``` javascript
var getUrlParam = function() {
    var strUrl = window.location.search.slice(1);
    var arrUrl = strUrl.split("&");
    var param = {}
    if (arrUrl.length) {
        for (var index in arrUrl) {
            param[arrUrl[index].split("=")[0]] = arrUrl[index].split("=")[1]
        }
    }
    return param;
}
```

#### 9.jquery ajax

``` javascript
$.ajax({
    url : URL + urlKey,
    type : 'post',
    datatype : 'JSON',
    data : data,
    success : function(data){
        if(data.errcode == 0){

        }else{
            showError('操作失败，请重试');
        }
    },
    error : function(){
        showError();
    }
});
```

