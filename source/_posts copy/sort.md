title: js常见排序算法
tags:
  - 算法
categories:
  - 前端
author: alpha
date: 2019-04-19 18:15:00
---

排序算法是js最基础的算法，只要弄清楚各种排序算法的排序逻辑，要实现起来就不难了。
1.冒泡排序
2.快速排序
3.选择排序
4.插入排序

<!--more-->
### 冒泡排序
冒泡排序的思想就是从左到右依次比较数组a相邻的两个数，将较大的数放到后面，一轮比较之后，最大的数会排到最后面；第二轮比较后次大的数会在倒数第二的位置，直到进行a.length次比较后，数组就会升序排列。

``` javascript
function bubbleSort(arr){
    arr.map((v,i)=>{
        arr.slice(0,arr.length-i).map((v,j)=>{
            if(v<arr[j-1]){
                arr[j] = arr[j-1]
                arr[j-1] = v
            }
        })
    })
    return arr
}

bubbleSort([2,4,1,7,3,5])//[1, 2, 3, 4, 5, 7]
```

### 快速排序
快速排序的思想是选定一个数为基准，小于这个基准的数放到左边，大于或等于基准的数放右边；然后对左边和右边的数分别进行快速排序，最后合并左边、基准和右边即可。

``` javascript
function quickSort(arr){
    if(arr.length <= 1){
        return arr
    }else{
        let left = [],right = []
        arr.slice(1).map((v,i)=>{
            if(v <= arr[0]){
                left.push(v)
            }else{
                right.push(v)
            }
        })
        return quickSort(left).concat(arr[0],quickSort(right))
    }
}

quickSort([2,4,1,7,3,5])//[1, 2, 3, 4, 5, 7]
```


### 选择排序
选择排序的思想是每次从数组中选出一个最小数放到数组最前面，第二次选择剩余数里最大的放到数组第二位，直到所有的数都被选择完

``` javascript
//选择排序——每次选择出最小的数，排到最前面
function selectSort(arr){
    arr.map((v,i)=>{
        let minIndex = i
        arr.slice(i).map((m,j)=>{
            minIndex = m < arr[minIndex] ? j+i : minIndex
        })
        if(minIndex !== i){
            arr[i] = arr[minIndex]
            arr[minIndex] = v
        }
    })
    return arr
}
selectSort([2,4,1,7,3,5])//[1, 2, 3, 4, 5, 7]

//选择排序——每次选择出最大的数，排到最后面
function selectSort(arr){
    arr.map((v,i)=>{
        let maxIndex = 0,lastIndex = arr.length - i - 1
        arr.slice(0,arr.length - i).map((m,j)=>{
            maxIndex = m > arr[maxIndex] ? j : maxIndex
        })
        if(maxIndex !== lastIndex){
            let lastNum = arr[lastIndex]
            arr[lastIndex] = arr[maxIndex]
            arr[maxIndex] = lastNum
        }
    })
    return arr
}
selectSort([2,4,1,7,3,5])//[1, 2, 3, 4, 5, 7]
```

### 插入排序
插入排序的思想是依次取出原数组中的一个数插入到一个有序数组中正确的位置，直到原数组里的数都被取完放入新数组，这里需要用到一个新的数组。

``` javascript
function insertSort(arr){
    let newArr = []
    arr.map((v,i)=>{
        if(i === 0){
            newArr.push(v)
        }else{
            let flag = newArr.some((m,j)=>{
                if(m > v){//将v插入m前
                    newArr.splice(j,0,v)
                    return true
                }
            })
            if(!flag)newArr.push(v)
        }
    })
    return newArr
}
insertSort([2,4,1,7,3,5])//[1, 2, 3, 4, 5, 7]
```
在斗地主的时候我们通常会把扑克牌按照大小顺序进行排列，但常常会见到这样两种场景，有的人喜欢摸一张牌然后将其插入到手里所有牌当中正确的位置，整个摸牌过程中他手里的牌总是按序排列的，这其实就是插入排序；而有的人喜欢先将所有的牌摸完然后从中依次选出最大的牌放到后面，这其实就是选择排序；还有的人把所有的牌摸完后随意选取一张作为基准，比它大的牌放一边，小的放另一边，这其实就是快速排序；不过几乎没有人会用冒泡排序来摸牌，因为需要不断交换牌的位置，比较耗时。当然，实际上大家可能是根据实际情况混用各种排序方式以达到最短时间内将牌排好序，各种排序算法其实已经被广大人民群众熟练运用了，可见也不是那么难！