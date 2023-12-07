---
title: JS 快速排序
comments: true
toc: false
excerpt: ' '
date: 2018-07-18 23:30:45
categories: Javascript
tags: 排序
sticky: 0
recommend: 0
---
基于[JS实现快速排序（2种方法）](https://blog.csdn.net/xin9910/article/details/73927936)的代码修改了下，跳出左右侧搜索时不需要判断等于基准值的情况，其次”当排序完有一侧只有0或者1个数字时则该侧不再进行排序”，不判断也可以，因为此时left等于right，进行排序时，会直接跳出循环，但是仍会打印排序后的数组，影响判断排序次数。数组数值用的是[数据结构（C语言版）（第二版）（严蔚敏 李冬梅 吴伟民编著）](https://e.jd.com/30611239.html)的，所以可以参照该书对比打印出来的情况。

至于第二种简捷的方式，非常简单易懂，但是由于那种方式每次都需要分配数组，会有额外内存开销。

``` javascript
    let arr = [49, 38, 65, 97, 76, 13, 27, 49];
    console.log('arr:' + arr);    //打印排序前的数组
    quickSort(arr, 0, arr.length-1);
    console.log('sortArr:' + arr);    //打印排序后的数组
 
    function quickSort(arr,low,high){    //数组  排序部分的初始索引  排序部分的结尾索引
        let key = arr[low];    //设置起始索引值为基准值
        let left = low;
        let right = high;
        while(right > left) {
            while (right > left && arr[right] >= key) right--;    //从右侧开始搜索
            if (arr[right] < key) {        //需要判断，因为可能右侧没有比基准值小的
                let temp = arr[right];
                arr[right] = arr[left];
                arr[left] = temp;
            }
            while (right > left && arr[left] <= key) left++;
            if (arr[left] > key) {    //可能左侧没有比基准值大的
                let temp = arr[left];
                arr[left] = arr[right];
                arr[right] = temp;
            }
        }
        console.log('newArr:' + arr);    //每次排完打印一次，显示几条就说明排了几次
        //排完后left等于right，即基准值在此次排序中的最终位置
        //如果left小等于low+1，说明左侧只有0或者1个数字，不需要再进行排序
        if (left > low+1) this.quickSort(arr, low, left - 1);
        //同理可得
        if (right < high-1) this.quickSort(arr, right + 1, high);
    }
```

![](running_result-0.png)

排序了4次，排序结果与最终结果均与书上一致。

# 参考

- [JS实现快速排序（2种方法） - 鱼儿跳](https://blog.csdn.net/xin9910/article/details/73927936)