---
title: JS 直接插入排序
comments: true
toc: true
excerpt: ' '
date: 2018-07-19 16:51:19
categories: Javascript
tags: 排序
sticky: 0
recommend: 0
---
``` javascript
    let arr = [49, 38, 65, 97, 76, 13, 27, 49];
    console.log('arr:' + arr);
    insertionSort(arr);
    console.log('sortArr:' + arr);
 
    function insertionSort(arr) {
        let preIndex, current;
        for (let i = 1; i < arr.length; i++) {    //第一个数不需要排，其他剩余几个值就要排几次
            preIndex = i - 1;
            current = arr[i];
            while(preIndex >= 0 && arr[preIndex] > current) {
                arr[preIndex+1] = arr[preIndex];    //把比当前要插入的数字大的值往后移
                preIndex--;
            }
            arr[preIndex+1] = current;
            //跳出循环说明前一个值比要插入的值小，插入的值的位置就应该是在它后面
        }
    }
```

![](running_result-0.png)