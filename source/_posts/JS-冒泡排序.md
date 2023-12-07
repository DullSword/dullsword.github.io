---
title: JS 冒泡排序
comments: true
toc: true
excerpt: ' '
date: 2018-07-19 14:19:46
categories: Javascript
tags: 排序
sticky: 0
recommend: 0
---
``` javascript
let arr = [49, 38, 65, 97, 76, 13, 27, 49];
console.log('arr:' + arr);    //打印排序前的数组
bubbleSort(arr);
console.log('sortArr:' + arr);    //打印排序后的数组
 
function  (arr) {
  for (i = 0; i < arr.length - 1; i++) {    //排序趟数 注意是小于
    for (j = 0; j < arr.length - i - 1; j++) {
      //一趟确认一个数，数组长度减当前趟数就是剩下未确认的数需要比较的次数
      //因为j从0开始，所以还要再减1，或者理解为arr.length-(i+1)
      if (arr[j] > arr[j + 1]) {
        let temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
    }
    console.log('newArr:' + arr);
  }
}
```

![](running_result-0.png)

加入`flag`，没有进行过交换，则不再进行多余地排序。 

``` javascript
let arr = [49, 38, 65, 97, 76, 13, 27, 49];
console.log('arr:' + arr);
bubbleSort(arr);
console.log('sortArr:' + arr);
 
function bubbleSort(arr) {
  let flag = 1;    //flag用来标记某一趟排序是否发生交换
  for (i = 0; i < arr.length - 1; i++) {
    flag = 0;    //flag置为0，如果本趟排序没有发生交换，则不会执行下一趟排序
    for (j = 0; j < arr.length - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        let temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
        flag = 1;    //flag置为1，表示本趟排序发生了交换
      }
    }
    console.log('newArr:' + arr);
    if (flag == 0) break;
  }
}
```

![](running_result-1.png)
