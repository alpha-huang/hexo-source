---
title: php导出excel 
author: alpha
tags:
  - 前端
  - php
  - codeigniter
  - excel
categories:
  - 学习笔记
date: 2018-5-29 20:23:14
---
在日常开发尤其是管理后台的开发中我们常常会遇到导入excel文件进行批量数据操作以及将数据导出到excel的场景，本篇主要说一下本人在开发过程中总结的php导出excel的步骤和需要注意的问题。
<!--more-->
## 一、常规方式
CI框架里导出excel可以调用excel_helper库里的**writeExcelToBrowser**方法，该方法可以将数组导出为excel文件输出到浏览器。
在业务中通常是根据用户的筛选条件调接口查询数据，或者是根据用户导入的excel进行相关操作获取操作结果，然后组装成数组，导出excel。
但是我们可能遇到页面超时，服务器内存溢出，超长数值型数据被显示成科学计数法，或者是用户有需要对某些列设置背景色等问题。

## 二、注意事项
### 1、科学计数法
excel编辑软件通常会将较大的数字例如订单号、身份证号显示成科学计数法，有些数字串太长被excel编辑软件转化为科学计数法后还会丢失一定的精度。例如原身份证号“421022199111111116”在Microsoft Excel里会被转化成“4.21022E+17”，重新转换成数值会变成“421022000000000000”，因此在导出文件时我们应该将单元格格式设置为文本型。

### 2、超时和内存溢出
进行批量操作的时候数据量大或操作接口耗时较长时可能遇到页面超时报错或者服务器内存溢出的情况。一方面我们可以对超时时间和内存限制进行设置，另一方面需要根据实际测试设置一个数据量限制。

``` php
set_time_limit(120);
ini_set('memory_limit','512M');
```
这里设置的超时时间只是php的超时时间，页面超时时间还受限于nginx配置的超时时间，以二者中较小的设置为准。

### 3、文件分发
如果生成的文件是暂存在服务器，需要将文件发布到所有的机器，否则用户下载文件的时候可能下载不到，虽然通常情况下用户ip不变，请求的服务器也不会变，但是我们没办法保证用户一定会请求到这台服务器。

### 4、删除文件
如果生成的文件是暂存在服务器，用户下载完之后应该尽量删除服务器上的文件，以节约磁盘空间。

## 三、优化
如果处理的数据量实在是太大了，接口不能在规定的时间内处理完数据，一定会造成超时，这时我们可以对系统进行优化，这里有三种优化方案。

### 1、采用异步任务
用一个异步任务调接口查询数据并且将数据写成文件暂存到服务器，前端页面每隔一段时间自动查询一下导出状态，这样就将一个请求分成了多次，规避了页面超时的问题，而且用户不用在页面等待，甚至可以关闭页面。这种方法需要写异步任务，还需要扭转任务状态，提供查询任务状态的接口，相对而言实现上会稍微复杂一些。

### 2、分多次请求
首次查询时只获取数据总量，然后前端页面分多次请求数据，每次将一定数量的数据追加写入文件暂存到服务器，请求结束后再自动下载文件，这样可以有效避免页面超时的问题。

### 3、使用csv格式
由于excel会带有一些额外的格式，例如单元格宽度，背景色，边框，字体，导致解析excel文件时非常占用服务器内存（尽管导出时可能并不需要设置），所以导出csv文件，会让系统可导出的数据量大大增加。
使用csv格式导出数据时，需要注意以下几点

 1. 对逗号进行转义，否则会被识别成csv间隔符。
 2. 保存csv文件时应该在文件头部加入BOM标识，将文件保存为utf-8编码格式以保证中文不会出现乱码

## 四、相关php代码
### 1、设置csv文件BOM
在所有数据写入前先写入**chr(0xEF).chr(0xBB).chr(0xBF)**
``` php
 fwrite($fp, chr(0xEF).chr(0xBB).chr(0xBF))
```

### 2、csv文件追加数据
下面是我封装的向csv文件追加数据的方法
``` php
private function _putcsv($_excelFile,$data,$lMainId = 1401,$documentType){
    foreach($data as $k =>$val){
        $data[$k] = str_replace(',','，',$val);
    }
    $data = $this ->_unsetSomeRow($data,$lMainId,$documentType);
    $fp = fopen($_excelFile, 'a+');
    fputcsv($fp,$data);
}
```

### 3、设定单元格格式
下面是我封装的一个将数组导出为excel文件的方法，包含的功能有

 - 设置指定列格式为文本型
 - 设置单元格背景色

``` php
/**
 * 输入生成excel文件到浏览器
 * writeExcelToBrowserV3($filename, $head, $data,array(),array('A','D'),array('D'=>'FFFFFF00','H'=>'FFFF0000'));
 * @param $filename 文件名
 * @param $headLine 表头
 * @param $data 数据
 * @param array $lostData
 * @param array $textCol 需要设置成文本类型的列
 * @param array $highLight 设置的单元格颜色array('D1'=>'FFFFFF00','H1'=>'FFFF0000')
 */
function writeExcelToBrowserV3($filename,$headLine,$data,$lostData=array(),$textCol=array(),$highLight=array()) {
	require_once  BASEPATH.'libraries/phpexcel/PHPExcel.php';
	$objPHPExcel = new PHPExcel();
	$objPHPExcel->setActiveSheetIndex(0);
	//第一行
	$i=0;
	foreach ($headLine as $v){
		$objPHPExcel->getActiveSheet()->setCellValueByColumnAndRow($i++,1,$v);
	}
	//写入数据
	$writeLineNumb	=	2;
	foreach ($data as $v) {
		$colNumb	=	0;
		foreach (array_keys($headLine) as $objAttr) {
			if (is_object($v) && isset($v->$objAttr)) {
				$cellValue	=	$v->$objAttr;
			}elseif(is_array($v) && isset($v[$objAttr])) {
				$cellValue	=	$v[$objAttr];
			}else {
				$cellValue	=	$lostData[$objAttr];
			}
			if(in_array(IntToChr($colNumb),$textCol)){
				$objPHPExcel->getActiveSheet()->setCellValueExplicit(IntToChr($colNumb++).$writeLineNumb, $cellValue,PHPExcel_Cell_DataType::TYPE_STRING);
			}else{
				$objPHPExcel->getActiveSheet()->setCellValueByColumnAndRow($colNumb++,$writeLineNumb,$cellValue);
			}
		}
		$writeLineNumb++;
	}
	if(!empty($highLight)){
		foreach($highLight as $row=>$color){
			$objPHPExcel->getActiveSheet()->getStyle($row)->getFill()->setFillType(PHPExcel_Style_Fill::FILL_SOLID);//单元格背景色
			$objPHPExcel->getActiveSheet()->getStyle($row)->getFill()->getStartColor()->setARGB($color);
		}
	}
	ob_end_clean();//清除缓冲区,避免乱码
	header('Content-Type: application/vnd.ms-excel');
	header(sprintf('Content-Disposition: attachment;filename="%s.xls"',$filename));
	header('Cache-Control: max-age=0');
	$objWriter = PHPExcel_IOFactory::createWriter($objPHPExcel, 'Excel5');
	$objWriter->save('php://output');
}

/**
 * 数字转字母 （类似于Excel列标）
 * @param Int $index 索引值
 * @param Int $start 字母起始值
 * @return String 返回字母
 */
function IntToChr($index, $start = 65) {
	$str = '';
	if (floor($index / 26) > 0) {
		$str .= IntToChr(floor($index / 26)-1);
	}
	return $str . chr($index % 26 + $start);
}
```
调用方法如下

``` php
$this ->load ->helper('excel_helper');
writeExcelToBrowserV3($filename,$headLine, $data,$lostData,array('A','D'),array('D1'=>'33FFFF00','E2'=>'FFFFFF00'));
```
示例

``` php
function export_user_info(){
        $head = array(
            '姓名',
            '手机号',
            '身份证号',
        );
        $data = array(
            array(
                '张三',
                13035144567,
                '421022199810134567',
            ),
            array(
                '李四',
                15945785412,
                '421022199711132548',
            ),
        );
        $this ->load ->helper('excel_helper');
        writeExcelToBrowserV3('用户信息', $head,$data,array(),array('B','C'),array('B1'=>'FFFFFF00','C1'=>'FFFF0000'));
    }
```
导出数据截图如下
 ![php导出excel文件][1]

[1]: /images/20180529212801.png "php导出excel文件"
