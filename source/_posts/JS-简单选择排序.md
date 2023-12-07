---
title: JS 简单选择排序
comments: true
toc: false
excerpt: ' '
date: 2018-07-27 16:51:32
categories: Javascript
tags: 排序
sticky: 0
recommend: 0
---
``` javascript
let arr=[49,38,65,97,76,13,27,49];
console.log(selectionSort(arr));
 
function selectionSort(arr) {
    let len = arr.length;
    let minIndex, temp;
    for (let i = 0; i < len - 1; i++) {    //只要len-1趟就可以完成排序，注意i从0开始
        minIndex = i;
        for (let j = i + 1; j < len; j++) {   //要比到最后一个数，其索引为len-1
            if (arr[j] < arr[minIndex]) {     //寻找最小的数
                minIndex = j;                 //将最小数的索引保存
            }
        }
        if(minIndex != i) {
            temp = arr[i];
            arr[i] = arr[minIndex];
            arr[minIndex] = temp;
        }
    }
    return arr;
}
```

![](running_result-0.png)

# 参考

- [常见几种排序之javascript实现 - 赵乘风_i](https://blog.csdn.net/zr15829039341/article/details/70598652)