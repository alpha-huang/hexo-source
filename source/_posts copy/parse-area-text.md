title: 解析地区文本の方法
author: alpha
tags:
  - php组件
categories:
  - 前端
date: 2016-12-15 16:13:00
---
# 场景

最近需求有涉及到将地区解析成编码的功能，例如将“北京市北京市辖县密云县”，解析出地址编码。没找到既有组件，于是自己写了一个。

<!--more-->
# 功能

将地区文本解析为地区编码或省市区，地区文本格式

```javascript
北京市北京市辖县密云县
```  
返回结果
```javascript
{
	"iResult": 0,
	"strErrmsg": "",
	"addressCode": 110228,
	"arrAddressCode": {
		"strProvinceCode": 110000,
		"strCityCode": 110200,
		"strAreaCode": 110228
	},
	"addressText": {
		"strProvince": "北京市",
		"strCity": "北京市辖县",
		"strArea": "密云县"
	}
}
```  
# PHP代码
```php
/**
     * 解析地区文本为编码
     * @param  $strAddress              地区文本，可接受的格式有两种  北京市北京市辖县密云县  北京市，北京市辖县，密云县 或者空格分隔
     * @param  int $accuracy            可接受的精确位数 1-精确到省，2-精确到市，3-精确到区，如果能解析到要求的位数则错误码为0
     * @return object iResult           1-只取到了市，但要求取到区 2-只取到了省，但要求取到市或区 3-取不到省
     * @return object strErrmsg
     * @return object addressCode       区编码
     * @return object arrAddressCode    省市区编码
     * @return object addressText       省市区文本
     */
    function get_code_by_address_name($strAddress,$accuracy=3){
        $oDistrictResp = $this->get_districtData();
        if($oDistrictResp ->iResult == 0){
            $arrDistrictData = $oDistrictResp->arrDistrictData;
            $strProvinceCode = '';
            $strCityCode = '';
            $strAreaCode = '';
            $strAddressCode = '';
            $strAddress = str_replace(' ', '', $strAddress);
            $arrAddress = preg_split( "/,|，/", $strAddress);
            $strProvince = isset($arrAddress[0]) ? $arrAddress[0] : '';
            $strCity     = isset($arrAddress[1]) ? $arrAddress[1] : '';
            $strArea     = isset($arrAddress[2]) ? $arrAddress[2] : '';
            if($strCity !== ''){//如果能解析到市，说明有分隔符
                foreach($arrDistrictData as $key=>$value){
                    if($strProvince === $value[0]){
                        $strProvinceCode = $key;
                        break;
                    }
                }
                if($strProvinceCode !== ''){
                    foreach($value[1] as $k=>$v){
                        if($strCity === $v[0]){
                            $strCityCode = $k;
                            break;
                        }
                    }
                    if($strCityCode !== ''){
                        foreach($v[1] as $code =>$area){
                            if($strArea === $area){
                                $strAreaCode = $code;
                                break;
                            }
                        }
                    }
                }
            }else{
                foreach($arrDistrictData as $key=>$value){
                    if(strpos($strAddress,$value[0],0) === 0){
                        $strProvinceCode = $key;
                        $strProvince = $value[0];
                        $strAddress = mb_substr($strAddress,mb_strlen($strProvince),50,'utf-8');
                        break;
                    }
                }

                if($strProvinceCode !== ''){
                    $specialDistrict = array(110000,120000,310000,500000);//北京 天津 上海 重庆
                    if(in_array($strProvinceCode,$specialDistrict) && strpos($strAddress,$strProvince.'辖县',0) === 0){
                        //如果是辖县
                        switch ($strProvinceCode){
                            case 110000:
                                $strCityCode = 110200;
                                break;
                            case 120000:
                                $strCityCode = 120200;
                                break;
                            case 310000:
                                $strCityCode = 310200;
                                break;
                            case 500000:
                                $strCityCode = 500200;
                                break;
                            default:
                                //TODO
                        }
                        $strCity = $strProvince.'辖县';
                        $strAddress = mb_substr($strAddress,mb_strlen($strProvince.'辖县'),50,'utf-8');
                    }else{
                        foreach($arrDistrictData[$strProvinceCode][1] as $k =>$v){
                            if(strpos($strAddress,$v[0],0) === 0){
                                $strCityCode = $k;
                                $strCity = $v[0];
                                $strAddress = mb_substr($strAddress,mb_strlen($strCity),50,'utf-8');
                                break;
                            }
                        }
                    }

                    if($strCityCode !== ''){
                        foreach($arrDistrictData[$strProvinceCode][1][$strCityCode][1] as $code =>$area){
                            if($strAddress === $area){
                                $strAreaCode = $code;
                                $strArea = $area;
                                break;
                            }
                        }
                    }
                }
            }

            $iResult = 0;
            $strErrmsg = '';
            if($strAreaCode !== ''){
                $strAddressCode = $strAreaCode;//取到了区，一定正确
            }elseif($strCityCode !== ''){
                $strAddressCode = $strCityCode;
                if($accuracy == 3){//只取到了市，但要求取到区，则错误
                    $iResult = 1;
                    $strErrmsg = '只取到了市，但要求取到区';
                }
            }elseif($strProvinceCode != ''){
                $strAddressCode = $strProvinceCode;
                if($accuracy == 2 || $accuracy == 3 ){//只取到了省，但要求取到市或区，则错误
                    $iResult = 2;
                    $strErrmsg = '只取到了省，但要求取到市或区';
                }
            }else{
                $iResult = 3;//取不到省
                $strErrmsg = '取不到省';
            }

            $oResult = (object)array(
                'iResult' => $iResult,
                'strErrmsg' => $strErrmsg,
                'addressCode' => $strAddressCode,
                'arrAddressCode' => array(
                    'strProvinceCode' => $strProvinceCode,
                    'strCityCode' => $strCityCode,
                    'strAreaCode' => $strAreaCode,
                ),
                'addressText' => array(
                    'strProvince' => $strProvince,
                    'strCity' => $strCity,
                    'strArea' => $strArea,
                )
            );

        }else{
            $oResult = $oDistrictResp;
        }
        return $oResult;
    }
```  

这里依赖了aiden哥的get_districtData()方法获取地区数据。
```php
/**
     * 获取地址数据
     *
     * @access private
     * @return object 调用结果（iResult为0表示通过，非0表示异常；arrDistrictData为地址数据）
     */
    private function get_districtData()
    {
        if (!$this->arrDistrictData) {
            $this->load->library('common/http');
            $content = file_get_contents('/usr/local/project/htdocs/static/sinclude/data/districtData.js');
            if (!$content) {
                return (object)array(
                    'iResult' => 536850208
                );
            }
            $this->arrDistrictData = json_decode($content, true);
        }

        return (object)array(
            'iResult' => 0,
            'arrDistrictData' => $this->arrDistrictData
        );
    }
```  


# 使用方法
app_pc项目下可以直接用，其他项目下需要把代码复制过去。
```php
$this->load->model('my/addressmodel');
$oRespCode = $this->addressmodel->get_code_by_address_name($addressText,2);
if($oRespCode ->iResult == 0){
    $arrcode = $oRespCode ->addressCode;
}
```
