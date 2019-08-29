title: 浮点数丢失精度
author: alpha
tags:
  - 问题
categories:
  - 前端
date: 2016-12-01 09:37:00
---
# 问题
浮点数在运算过程中常常会丢失精度，这是由于二进制数的存储特点造成的，在php或者js中进行浮点数运算或者类型转换的时候常常会丢失精度。而在电商公司，对金额比较敏感，是万万不能接受丝毫的误差的。
看下面这段代码，它的运行结果分别是什么呢？
<!--more-->
```javascript
$var1 = 298.90;
$var2 = $var1 * 100;
$var3 = (int)$var2;
$var4 = (string)$var2;

echo $var2;
echo $var3;
echo $var4;
```

你的答案可能是

```javascript
29890
29890
29890
```
如果真是这样，也就没必要特意提出来说了，其实运行结果是这样的

```javascript
29890
29889
29890
```

为什么第二个值变成了29889呢？这和预期不符

让我们来分析一下这个问题，下面这段js代码，可以copy到控制台运行一下

```javascript
var a = 289.90;
console.log(a * 100);
console.log(parseInt(a * 100));
console.log(a * 100 + '');
```
运行结果如下
```javascript
28989.999999999996
28989
28989.999999999996
```
运行上面这段js代码，结果是浮点数经过乘法运算之后得出的值已经是略小于真实值了，原因是计算机是以二进制数处理数字的，进行运算之后由于长度限制会丢失精度。而经过强制类型转换，变成整型会截取非数字前的部分，就比如运行下面的代码结果会是数值289和数值-289。
```javascript
var a = '289abc';
var b = '-289abc';
console.log(parseInt(a));
console.log(parseInt(b));
```
运行结果
```javascript
289
-289
```
这样就可以解释为啥开头的例子里第二个数是29889了。

再来看另一个例子

```javascript
$var1 = 298.90;
$var2 = $var1 * 100;

$var3 = (int)($var2.'');

$var4 = (string)$var2;

echo $var2;
echo $var3;
echo $var4;
```
运行结果是
```javascript
29890
29890
29890
```
就因为转化成了字符串，就一切如常了。

为什么会这样呢？这里我也不太清楚原理，查阅资料也没有弄清楚，希望有知道的同学留言解答一下！


那精度丢失的问题到底有多严重，什么时候我们需要注意呢？我们可以写几个demo来大概了解一下。最常遇到的运算就是“元”和“分”的相互转换。

在js里将0.01元 到100元之间的10000个数值，分别转化成分
```javascript
var right = 0,error = 0,j = 100;
for(var i = 0;i < 100;i = (parseFloat(i) + 0.01).toFixed(2)){
	var res = i * j;
	if(res != res.toFixed(2)){
		error ++;
		console.log(i + ' * ' + j + ' = ' + res);
	}else{
		right ++;
	}
}
console.log('right: ' + right);
console.log('error: ' + error);
console.log('over');
```
结果如下

```javascript
...
81.46 * 100 = 8145.999999999999
81.49 * 100 = 8148.999999999999
81.51 * 100 = 8151.000000000001
81.54 * 100 = 8154.000000000001
81.57 * 100 = 8156.999999999999
81.60 * 100 = 8159.999999999999
81.65 * 100 = 8165.000000000001
81.68 * 100 = 8168.000000000001
81.71 * 100 = 8170.999999999999
81.74 * 100 = 8173.999999999999
81.76 * 100 = 8176.000000000001
81.79 * 100 = 8179.000000000001
81.82 * 100 = 8181.999999999999
81.85 * 100 = 8184.999999999999
81.90 * 100 = 8190.000000000001
right: 8854
error: 1146
over
```
在js里将1分到10000分之间的10000个数值，分别转化成元

```javascript
var right = 0,error = 0,j = 100;
for(var i = 0;i < 10000;i = Math.round(parseInt(i) + 1)){
	var res = i / j;
	if(res != res.toFixed(2)){
		error ++;
		console.log(i + ' / ' + j + ' = ' + res);
	}else{
		right ++;
	}
}
console.log('right: ' + right);
console.log('error: ' + error);
console.log('over');
```
结果如下

```javascript
right: 10000
error: 0
over
```
在js里使两个0到1之间的两位小数相减

```javascript
var right = 0,error = 0;
for(var i = 0;i < 1;i = (parseFloat(i) + 0.01).toFixed(2)){
	for(var j = 0;j < 1;j = (parseFloat(j) + 0.01).toFixed(2)){
		var res = parseFloat(i) - parseFloat(j);
		if(res != res.toFixed(2)){
			error ++;
			console.log(i + ' - ' + j + ' = ' + res);
		}else{
			right ++;
		}
	}
}
console.log('right: ' + right);
console.log('error: ' + error);
console.log('over');
```
结果如下

```javascript
...
0.99 - 0.83 = 0.16000000000000003
0.99 - 0.84 = 0.15000000000000002
0.99 - 0.88 = 0.10999999999999999
0.99 - 0.89 = 0.09999999999999998
0.99 - 0.90 = 0.08999999999999997
0.99 - 0.91 = 0.07999999999999996
0.99 - 0.92 = 0.06999999999999995
0.99 - 0.93 = 0.05999999999999994
0.99 - 0.94 = 0.050000000000000044
0.99 - 0.95 = 0.040000000000000036
0.99 - 0.96 = 0.030000000000000027
0.99 - 0.97 = 0.020000000000000018
0.99 - 0.98 = 0.010000000000000009
right: 4844
error: 5156
over
```
在js里使两个0到1之间的两位小数相加
```javascript
var right = 0,error = 0;
for(var i = 0;i < 1;i = (parseFloat(i) + 0.01).toFixed(2)){
	for(var j = 0;j < 1;j = (parseFloat(j) + 0.01).toFixed(2)){
		var res = parseFloat(i) + parseFloat(j);
		if(res != res.toFixed(2)){
			error ++;
			console.log(i + ' + ' + j + ' = ' + res);
		}else{
			right ++;
		}
	}
}
console.log('right: ' + right);
console.log('error: ' + error);
console.log('over');
```
结果如下
```javascript
...
0.99 + 0.12 = 1.1099999999999999
0.99 + 0.35 = 1.3399999999999999
0.99 + 0.37 = 1.3599999999999999
0.99 + 0.40 = 1.3900000000000001
0.99 + 0.58 = 1.5699999999999998
0.99 + 0.60 = 1.5899999999999999
0.99 + 0.62 = 1.6099999999999999
0.99 + 0.65 = 1.6400000000000001
0.99 + 0.67 = 1.6600000000000001
0.99 + 0.83 = 1.8199999999999998
0.99 + 0.85 = 1.8399999999999999
0.99 + 0.87 = 1.8599999999999999
0.99 + 0.90 = 1.8900000000000001
0.99 + 0.92 = 1.9100000000000001
right: 7894
error: 2106
over
```
在这些例子里，出错的值占到了很高的比例，但错误值和真实值之间的误差非常小，四舍五入就可以避免。我们在处理数值运算时一定要注意进行处理。


# 总结

1 如果遇到精度丢失，最简单的办法就是四舍五入
```javascript
//php方法
$lDefSupPrice = round(79.60 * 100);//取整
$lDefSupPrice = sprintf("%.2f", (0.99 + 0.92));//保留两位小数
//js方法
var fPrice = Math.round(79.60 * 100);//取整
var fPrice = (0.99 + 0.92).toFixed(2);//保留两位小数
```


2 将整数部分与小数部分分开分别运算，例如


```javascript
define("float.operation", function(require, exports, module) {
    //加法
    Number.prototype.add = function(arg){
        var r1,r2,m;
        try{r1=this.toString().split(".")[1].length}catch(e){r1=0}
        try{r2=arg.toString().split(".")[1].length}catch(e){r2=0}
        m=Math.pow(10,Math.max(r1,r2))
        return (Number((this*m).toFixed())+Number((arg*m).toFixed()))/m
    }

    //减法
    Number.prototype.sub = function (arg){
        return this.add(-arg);
    }

    //乘法
    Number.prototype.mul = function (arg)
    {
        var m=0,s1=this.toString(),s2=arg.toString();
        try{m+=s1.split(".")[1].length}catch(e){}
        try{m+=s2.split(".")[1].length}catch(e){}
        return Number(s1.replace(".",""))*Number(s2.replace(".",""))/Math.pow(10,m)
    }

	//除法
    Number.prototype.div = function (arg){
        var t1=0,t2=0,r1,r2;
        try{t1=this.toString().split(".")[1].length}catch(e){}
        try{t2=arg.toString().split(".")[1].length}catch(e){}
        with(Math){
            r1=Number(this.toString().replace(".",""))
            r2=Number(arg.toString().replace(".",""))
            return (r1/r2)*pow(10,t2-t1);
        }
    }
});
```

3 如果遇到在调接口的时候php传到接口的时候是准确的，后台读取的时候出错了，可以以字符串的形式来传这个字段，因为PHP是弱类型语言，现在我们的接口大多数情况允许类型不准确

4 在用php处理excel、csv等表格的时候，也可能遇到数据类型的问题，例如生成表格的时候如果以字符串形式存大数字（例如手机号、订单号、身份证号），默认会以科学计数法来显示，甚至身份证号精确度不够直接将后几位置为0了，在前面拼接上空格或英文单引号’以字符串形式输出，在表格里就能正确显示了

5 对于订单号和手机验证码之类可能以0开头的数字，千万不能转整型，另外也不能用 getValueI()方法
```javascript
$this ->getValueI();
```
6 对于较大数字的运算，例如解析和拼装后台的属性标，不建议在js中运算，容易溢出，在php中运算会有所改善，附上php解析属性标的方法
```javascript
/***
 * 解析订单属性lTradeProperty1标签
 * 入参是属性标
 * 回参是一个包含属性标字符串的数组
 */
function getTradeProperty1($val=NULL){
    $skuPropertyNew=array(
        0x000000001 => 'change', //参加以旧换新活动
        0x000000002 => 'coupon', //使用优惠券
        0x000000004 => 'presell', //预售商品（只推迟发货）
        0x000000008 => 'limit', //限时优惠
        0x000000010 => 'score', //积分抵扣
        0x000000020 => 'gift', //礼品券
        0x000000040 => 'newtest', //新品试用
        0x000000080 => 'presellmoney', //预售商品(要交定金)
);
    $arrProperty=array();
    foreach($skuPropertyNew as $k=>$v) {
        if($val & $k) {
            $arrProperty[]=$v;
        }
    }
    return $arrProperty;
}
```

```javascript
欢迎补充！
```


参考：

[PHP浮点数的一个常见问题的解答](http://www.laruence.com/2013/03/26/2884.html)

[关于PHP浮点数你应该知道的](http://www.laruence.com/2011/12/19/2399.html)
