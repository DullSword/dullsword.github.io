---
title: JS 获取对象的长度
date: 2018-07-07 15:49:12
comments: true
toc: true
categories: Javascript
tags: Javascript对象
excerpt: ' '
sticky: 0
recommend: 0
---
除了`for in`循环，还可以通过`Object.keys(obj).length`获取。

``` javascript
let obj = {
    "a":0,
    "b":1
};
console.log(Object.keys(obj).length);   // 输出结果为2 
```