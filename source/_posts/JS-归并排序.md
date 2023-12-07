---
title: JS 归并排序
comments: true
toc: true
excerpt: ' '
date: 2018-07-27 16:00:30
categories: Javascript
tags: 排序
sticky: 0
recommend: 0
---
# 递归

``` javascript
let arr = [49, 38, 65, 97, 76, 13, 27, 49];
console.log(mergeSort(arr))
 
function merge(left, right) {
  let tmp = [];
 
  while (left.length && right.length) {
    if (left[0] < right[0])
      tmp.push(left.shift());
    else
      tmp.push(right.shift());
  }
 
  return tmp.concat(left, right);
}
 
function mergeSort(a) {
  if (a.length === 1) 
    return a;
 
  let mid = Math.floor(a.length / 2);
  let left = a.slice(0, mid);
  let right = a.slice(mid);
 
  return merge(mergeSort(left), mergeSort(right));
}
```

![](running_result-0.png)

>这段合并排序的代码相当简单直观，但是`mergeSort()`函数会导致很频繁的自调用。一个长度为n的数组最终会调用`mergeSort()` 2*n-1次，这意味着如果需要排序的数组长度很大会在某些栈小的浏览器上发生栈溢出错误。\
\
<b>来源自网络</b>

# 迭代

``` javascript
console.log(mergeSort([1, 3, 4, 2, 5, 0, 8, 10, 4]));
 
function merge(left, right) {
  let result = [];
 
  while (left.length && right.length) {
    if (left[0] < right[0])
      result.push(left.shift());
    else
      result.push(right.shift());
  }
 
  return result.concat(left, right);
}
 
function mergeSort(a) {
  if (a.length === 1)
    return a;
 
  let work = [];
  for (let i = 0, len = a.length; i < len; i++)
    work.push([a[i]]);
 
  work.push([]); // 如果数组长度为奇数,避免下面work[k+1]越界
 
  for (let lim = len; lim > 1; lim = Math.floor((lim + 1) / 2)) {
    for (let j = 0, k = 0; k < lim; j++, k += 2) 
      work[j] = merge(work[k], work[k + 1]);
 
    work[j] = []; //见下面注释
  }
 
  return work[0];
}
```

## 迭代时为什么要work[j] = []?

因为`merge`时比较大的值并未从数组中删去，虽然会被覆盖掉，但每趟合并的最后一个`work[k+1]`里的值如果有剩余，就会被保留下来，如果数组长度为奇数，则在下一趟合并时就会和最后一组有序子序列一起合并，导致出现多余的数字的错误。

![](debug-0.png)

如上图，1和3已经合并，但由于1比3小，所以执行了`merge`函数里`result.push(left.shift())`，而`right`其实是没有进行操作的，所以`work[1]`里的3还存在。

![](debug-1.png)

这里合并完后，分为了3组有序子序列`[1,2,3,4]`，`[0,5,8,10]`，`[4]`，但由于遗留下来的`work[3]`里的`[8,10]`，下一趟合并时会和索引为2的有序子序列`[4]`，进行合并。

![](debug-2.png)

从而导致排序后多了个8和10。所以在每趟合并后加入`work[j] = []`，将最后一组有序子序列索引+1的数据清空。