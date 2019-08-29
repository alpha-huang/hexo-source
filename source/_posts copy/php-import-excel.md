---
title: php读取excel 
author: alpha
tags:
  - 前端
  - php
  - codeigniter
  - excel
categories:
  - 学习笔记
date: 2018-5-29 10:36:06
---
在日常开发尤其是管理后台的开发中我们常常会遇到导入excel文件进行批量数据操作以及将数据导出到excel的场景，本篇主要说一下本人在开发过程中总结的php读取excel的步骤和需要注意的问题。
<!--more-->

## 一、常规方式

读取excel的过程简单说来就是接收文件、解析文件、数据处理、返回结果，但是由于导入文件的自由度比在页面上填写一个表单大多了，所以服务器可能接收到各种形式的文件，因此数据的有效性校验尤其重要。另外，由于excel导入文件通常会进行大量数据的批量操作，因此处理好页面超时、服务器内存溢出、容错也非常重要。

导入excel的一般处理步骤如下

 1. 接收和校验文件（类型，大小，是否存在：直接报错） 
 2. 校验空表（无数据，整行全是空字段：跳过这行） 
 3. 校验数据行数（不能超过允许处理的数据量）
 4. 校验空字段（只读需要的列，并且校验空字段：记录下来）
 5. 校验字段（长度、类型：记录下来）
 6. 校验重复（去重）
 7. 类型转换，xss过滤，数据处理，重新组装数据
 8. 字段有效性校验（例如商品是否存在）
 9. 调接口进行业务操作
 10. 结果处理，异常处理
 11. 反馈操作结果（生成日志和错误提示或错误表格）

 ![php读取excel文件][1]

上图左边是php校验和解析excel文件，右边是校验和操作数据。

看起来很繁琐，其实前面7步都是在进行数据校验和预处理。在开发过程中我们应该尽量对用户的误操作进行友好的提示，尤其是excel操作的过程中，极高的自由度会导致极高的出错概率。用户不是开发人员，通常不知道怎么生产出让系统理解的数据，在开发和维护过程中我们遇到了各种令人啼笑皆非的“误操作”，因此系统应该尽量引导用户进行检查和修改。

### 1、接收和校验文件
通过校验接收的文件的MIME以及文件大小来判断用户上传的文件是否有效，避免用户上传错误格式的文件或者是超出服务器处理能力的超大文件，上传超大文件很容易造成服务器内存溢出。

### 2、校验空表
用户常常会在编辑表格时删除整行的每个单元格而不是直接删除整行，导致留下一些空行，这些空行是没有意义的，系统应该对其进行过滤，只保留有数据的行。

### 3、校验数据行
限于服务器的处理能力以及业务的复杂程度，我们需要限定一个允许的数据行数，例如10万行，如果超过很可能造成服务器内存溢出或者页面超时报错。同时我们也可以将服务器处理excel的超时和内存限制适当设置得大一些。

### 4、校验空字段
对系统需要的字段进行非空检查。有时候系统不需要读取表格的所有列，有些列是为了便于用户查看才加上的，对系统没有意义的列不需要校验（但可能需要读取和暂存，例如出现错误时需要提示用户具体是哪一行出错了，那一行的内容是什么）。

### 5、校验数据格式
按业务要求校验字段类型、长度等。

### 6、去重
按业务要求进行数据行去重或报错。

### 7、类型转换，xss过滤，数据处理，重新组装数据
经过前面6步校验，基本上可以确定数据的有效性了，接下来对数据进行预处理，使数据符合操作接口的要求。例如xss过滤、用户填的文本需要转化成接口需要的数字、将数据结构调整为接口需要的形式。

### 8、数据有效性校验
对数据进行业务相关的校验，例如需要对商品进行修改应该先查询商品是否存在，对订单进行发货操作应该先查询订单是否存在（这里只是举两个例子，实际上操作接口一般会对这种情况做兼容，即使商品或订单不存在，直接调接口也不会有问题）。

### 9、调接口
调接口进行业务操作，如果接口不能一次性处理所有数据，就需要分批循环调接口，如果接口还有频率限制，在循环时需要注意保持适当的时间间隔。

### 10、结果处理、异常处理
根据接口返回的信息组装成合适的数据格式，用来提示用户哪些数据操作成功，哪些数据操作失败，失败的原因等。

### 11、反馈操作结果
在批量操作时经常会遇到部分成功部分失败的情况，因此导出一份错误的表格是比较直观的方式，可以在表格里加一列注明操作失败的原因，用户可以根据提示对该表格进行再次编辑和导入系统。如果出现系统错误还应该记录操作日志，以便于及时定位和处理。

## 二、注意事项

### 1、科学计数法
excel编辑软件通常会将较大的数字例如订单号、身份证号显示成科学计数法，这对于系统来说是不合法的数据格式，如果能进行转换，系统应该尽量允许这种格式，并将其转换为正确的数据。但当转换结果不准确的时候应该提示用户将数据格式设置为文本型，例如有些数字串太长被excel编辑软件转化为科学计数法后还会丢失一定的精度，即使再转化成字符串，数据也不对了。例如原身份证号“421022199111111116”在Microsoft Excel里会被转化成“4.21022E+17”，重新转换成数值会变成“421022000000000000”。

### 2、超时和内存溢出
进行批量操作的时候数据量大或操作接口耗时较长时可能遇到页面超时报错或者服务器内存溢出的情况。一方面我们可以对超时时间和内存限制进行设置，另一方面需要根据实际测试设置一个数据量限制。

``` php
set_time_limit(120);
ini_set('memory_limit','512M');
```
这里设置的超时时间只是php的超时时间，页面超时时间还受限于nginx配置的超时时间，以二者中较小的设置为准。

### 3、空白单元格
在系统使用过程中，可能会遇到用户上传的excel文件分明只有几百行，但还是内存溢出。可能是因为用户在编辑excel的过程中对单元格格式进行了修改，导致很多空行或者列有了无用信息，从而在服务器解析excel文件时造成内存溢出。这时可以将原excel文件里的内容粘贴到一份新的excel文件里以丢弃掉那些无用信息。

## 三、优化
如果处理的数据量实在是太大了，接口不能在规定的时间内处理完数据，一定会造成超时，这时我们可以对系统进行优化，这里有三种优化方案。

### 1、采用异步任务
将表格解析出来之后用一个异步任务调接口，服务端直接返回结果到前端页面，页面每隔一段时间自动查询一下操作状态，这样就将一个请求分成了多次，规避了页面超时的问题，而且用户不用在页面等待，甚至可以关闭页面。这种方法需要写异步任务，还需要扭转任务状态，提供查询任务状态的接口，相对而言实现上会稍微复杂一些。

### 2、分多次请求
上传完文件之后服务端不进行数据操作，只解析excel文件里的数据，解析完之后将数据返回页面或暂存到redis里。前端页面循环发送请求，将数据拆分成多份进行处理。需要注意的是如果将数据返回到页面，暂存到浏览器内存中，请求过程需要注意http请求的参数限制，post请求默认接收1000个参数，所以页面在传参到服务器时应该将参数转化为字符串（如果直接传多维数组，数组的每一个子元素都会被视为一个参数，很容易超出限制）。

### 3、使用csv格式
由于excel会带有一些额外的格式，例如单元格宽度，背景色，边框，字体，导致解析excel文件时非常占用服务器内存（尽管用户可能并没有设置），而通常情况下系统并不关心这些信息，所以将excel转换成csv后再导入系统，会让系统可处理的数据量大大增加。

## 四、相关php代码
下面是我封装的一个解析excel文件为数组的方法，该方法接收一个excel文件，返回一个数组，包含的功能有

 - 校验文件MIME类型
 - 校验文件大小
 - 读取数据
 - 过滤空行

``` php
/**
 * 解析excel表格为数组
 * @param array $params
 * int max_size: 限制文件大小（M）
 * int count_col:需要读取表格前多少列，防止读到后面的空白列导致内存溢出
 * bool need_head: 是否需要读取表头
 * 将会跳过空行
 * @return array
 */
function parse_excel($params = array()){
	$fileType = array("application/vnd.ms-excel","application/vnd.openxmlformats-officedocument.spreadsheetml.sheet","application/kset");
	$max_size = isset($params['max_size']) ? $params['max_size'] : 2;
	if (!(isset($_FILES["file"]) && 0 == $_FILES["file"]["error"])) {
		cilog('debug', '上传失败：isset($_FILES["file"])：'.isset($_FILES["file"]).'上传文件$_FILES["file"]["error"]：'.$_FILES["file"]["error"]);
		$result = array(
			'errcode' =>1001,
			'errmsg' =>"文件上传失败",
		);
	} elseif ($_FILES["file"]["size"] > $max_size * 1024 * 1024) {
		cilog('debug', '上传文件大小：'.$_FILES["file"]["size"]);
		$result = array(
			'errcode' =>1002,
			'errmsg' =>"上传文件不得超过{$max_size}Mb",
		);
	} elseif (!in_array($_FILES["file"]["type"],$fileType)) {
		cilog('debug', '上传文件MIME：'.$_FILES["file"]["type"]);
		$result = array(
			'errcode' =>1003,
			'errmsg' =>"文件类型错误，请上传.xls或.xlsx文件",
		);
	} else{
		$file = $_FILES['file']['tmp_name'];
		require_once BASEPATH . '/libraries/phpexcel/PHPExcel.php';
		$excelReader = new PHPExcel_Reader_Excel2007();
		if (!$excelReader->canRead($file)) {
			$excelReader = new PHPExcel_Reader_Excel5();
		}
		$sheet = $excelReader->load($file)->getSheet(0); //sheet1操作
		$excelCont = array(
			'highestCol' => $sheet->getHighestColumn(), //列
			'highestRow' => $sheet->getHighestRow(), //行
			'highestColumnIndex' => PHPExcel_Cell::columnIndexFromString($sheet->getHighestColumn()) // 几列
		);
		$countCol = isset($params['count_col']) ? $params['count_col'] : $excelCont['highestRow'];//有效列的数目，读取时只取前$countCol列
		$startRow = isset($params['need_head']) && $params['need_head'] ? 1 : 2;//从第1行开始读，读取表头
//              $excelCont['highestRow'] 有几行
//              表示第一行第一列 $sheet->getCellByColumnAndRow(0, 1)->getValue()的值
		$rightArr = array();
//              总共导入多少行  $row - $emptyLine -1
		$row = $excelCont['highestRow'];
		$emptyLine = 0;
		for ($j = $startRow; $j < intval($row) + 1; $j++) {
			$retArr = array();//该行的各个单元格的数据
			$emptyCol = 0;
			for($i = 0; $i < $countCol;$i++){//循环该行的单元格
				$retArr[$i] = $sheet->getCellByColumnAndRow($i, $j)->getValue();
				$retArr[$i] = isset($retArr[$i]) ? trim($retArr[$i]) : $retArr[$i];
				if($retArr[$i] === null){
					$emptyCol++;
				}
			}
			if($emptyCol == $countCol){//这行为空行，不放入任何数组
				$emptyLine++;
			}else {
				$rightArr[] = $retArr;
			}
		}
		if(empty($rightArr)){
			$result = array(
				'errcode' =>1004,
				'errmsg' =>"上传文件内没有有效数据",
			);
		}else{
			$result = array(
				'errcode' =>0,
				'errmsg' =>"",
				'data' =>$rightArr,
			);
		}
	}
	return $result;
}
```

[1]: /images/20180529200101.png "php读取excel文件"