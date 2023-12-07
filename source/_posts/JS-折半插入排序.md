---
title: JS 折半插入排序
comments: true
toc: false
excerpt: ' '
date: 2018-07-19 23:46:47
categories: Javascript
tags: 排序
sticky: 0
recommend: 0
---
![](JS折半插入排序.png)

<a class="tag is-dark is-medium" style="margin-bottom: 1rem" href="https://blog.csdn.net/guoweimelon/article/details/50904206" target="_blank">
<span class="icon"><i class="fas fa-camera"></i></span>&nbsp;&nbsp;
来源自 经典排序算法（4）——折半插入排序算法详解 - 郭威gowill
</a>

>折半插入排序的对象移动次数与直接插入排序相同，依赖于对象的初始排列。\
\
在平均情况下，折半插入排序仅减少了关键字间的比较次数，而记录的移动次数不变。因此，折半插入排序的时间复杂度仍为O(n^2)。\
\
<b>数据结构（C语言版）（第二版）（严蔚敏 李冬梅 吴伟民编著）</b>

``` javascript
    let arr = [49, 38, 65, 97, 76, 13, 27, 49];
    console.log('arr:' + arr);
    binaryInsertionSort(arr);
    console.log('sortArr:' + arr);
 
    function binaryInsertionSort(arr){
	    for(let i=1;i<arr.length;i++){    //第一个数不需要排，其他剩余几个值就要排几次
		    let current=arr[i],left=0,right=i-1;    //i的值等于当前排序值的索引
		    while(left<=right){
			    let middle=Math.floor((left+right)/2);
			    if(current<arr[middle]){
				    right=middle-1;
			    }else{    //等于的情况下，继续在后面进行折半插入排序，最终新插入的值会排在后面
				    left=middle+1;
			    }
		    }
                    //找到插入的位置
		    for(let j=i-1;j>=left;j--){    //把插入值前一个的值到插入位置的值全部往后移动一位
			    arr[j+1]=arr[j];
		    }
		    arr[left]=current;    //把插入值放置在空出来的位置
	    }
    }
```

![](running_result-0.png)

# 参考

- [常见几种排序之javascript实现 - 赵乘风_i](https://blog.csdn.net/zr15829039341/article/details/70598652)