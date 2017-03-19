title: CI联表查询
author: alpha
tags:
  - CodeIgniter
  - 联表查询
categories:
  - 前端
date: 2016-5-19 18:14:20
---
昨天小伙伴遇到一个联表查询的问题，隐约记得大学的《数据库原理》上有讲过，但是无奈当时只顾着梦游，不记得具体怎么做了，请教前辈后得出三种处理方法。
具体问题：
表A与表B中同时存在某个字段f，现在需要在表A里查找字段f后到表B查询相关数据，这是个很典型且基础的联表查询。
<!--more-->
以数据表t_mos和user为例

| sel_id | uid   | sel_name  |
| --- | ----- | ------ |
| 329 | alpha | 阿尔法 |
| 330 | beta  | 贝塔   |
| 331 | gamma | 伽马   |
| 332 | delta | 阿尔法 |

（数据表t_mos）

| usr_id | usr_email       |
| ------ | --------------- |
| alpha    | alpha@midea.com |
| beta    | beta@midea.com  |
| gamma    | gamma@midea.com |
| delta    | delta@midea.com |

（数据表user）
现在需要查询昵称（sel_name）为‘阿尔法’的用户的邮箱地址（user_email）。
### 方法一：
left join sql语句查询

``` php
$select = "select u.usr_email from t_mos t left join user u on t.uid =u.usr_id  where t.sel_name in ('阿尔法');";
$query = $this->MASTER_DB->query($select)->result();
```


运行结果

``` scheme
[0]=>
  object(stdClass)#34 (1) {
    ["usr_email"]=>
    string(14) "alpha@midea.com"
  }
[1]=>
  object(stdClass)#35 (1) {
    ["usr_email"]=>
    string(18) "delta@midea.com"
  }
```


### 方法二：
使用的CI的数据库操作类

``` php
$query = $this ->MASTER_DB ->where('sel_name','阿尔法') ->join('user', 't_mos.uid =user.usr_id', 'left') ->select('usr_email') ->get('t_mos')->result();
```
运行结果和上面一致
### 方法三：
分两次查询，在表t_mos中查到uid后再去表user里查usr_email

``` php
$resp = $this ->MASTER_DB ->where('sel_name',阿尔法') ->select('uid') ->get("t_mos") ->result_array();
$uids =array();
foreach($resp as $v){
    $uids[] = $v['uid'];
}
$query = $this ->MASTER_DB ->where_in('usr_id',$uids) ->select('usr_email') ->get("user") ->result_array();
```
运行结果和上面一致


第一种方法直接写sql语句，存在sql注入的风险
第二种方法用CI的数据库操作类join，大表不建议使用，因为使用join进行关联查询，一张表的数据量大时可能会阻塞其他线程的写入操作。
第三种方法分两次查询，需要前端手动处理数据，相比于前两种方法写法稍复杂，但可以优化性能。
把逻辑都放到代码中，数据库就当成 k-v 存储，配合索引，效率更高。 因为数据库服务器比较集中，复杂查询容易出现瓶颈，但是前端代码的服务器，一直都有富裕的，扩充也比较容易。
因此推荐第三种方法。
为了测试三种方法的效率，我往两张数据表里都添加了50W条数据，经过多次测试，结果显示，首次查询耗时比较长，约为100ms,之后都在1ms左右，下面是查询耗时（单位是毫秒ms）

``` stylus
float(147.28403091431)
float(84.766149520874)
float(85.854053497314)
float(0.73790550231934)
float(0.81992149353027)
float(1.5439987182617)
```

测试结果仅供参考。

### insert_batch()
另外，在批量插入数据时发现了CI的批量插入数据的数据库类insert_batch()的一个bug，那就是每次插入100条数据，最后返回的affected_rows()只是最后一次的影响条数。CI的insert_batch()部分的源码如下：

``` php
for ($i = 0, $total = count($this->ar_set); $i < $total; $i = $i + 100)
{
    $sql = $this->_insert_batch($this->_protect_identifiers($table, TRUE, NULL, FALSE), $this->ar_keys, array_slice($this->ar_set, $i, 100));
    //echo $sql;
    $this->query($sql);
}
```


要想获取准确的条数只能自己处理了。

参考资料：

[sql之left join、right join、inner join的区别][1]

[MYSQL加锁的测验][2]

[CodeIgniter的insert_batch()数以千计的插入有丢失的记录][3]


  [1]: http://www.cnblogs.com/wangtao_20/p/3463435.html
  [2]: http://www.4byte.cn/question/537523/codeigniter-s-insert-batch-with-thousands-of-inserts-has-missing-records.html
  [3]: http://www.4byte.cn/question/537523/codeigniter-s-insert-batch-with-thousands-of-inserts-has-missing-records.html
