---
title: JS 对象按照value排序的实现
comments: true
toc: true
excerpt: ' '
date: 2019-02-18 18:37:50
categories: Javascript
tags: Javascript对象
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 未知
原文标题： javascript对象按照value排序的实现
链接： https://segmentfault.com/a/1190000006708598
来源： SegmentFault 思否
转载篇幅： 未知
是否修改： 未知
{% raw %}</div></article>{% endraw %}

``` javascript
function objsortbyval(obj) {
  var keyArr = [],valArr = [];
  for (var key in obj) {
    keyArr.push(key);
    valArr.push(obj[key]);
  }
  for (var i = 0, len = valArr.length; i < len; i++) {
    for (var j = 0; j < len - i; j++) {
      var keyTemp, valTemp;
      if (valArr[j] < valArr[j + 1]) {
        valTemp = valArr[j];
        valArr[j] = valArr[j + 1];
        valArr[j+1] = valTemp;
        keyTemp = keyArr[j];
        keyArr[j] = keyArr[j + 1];
        keyArr[j+1] = keyTemp;
      }
    }
  }
  var newobj={};
  for(var i=0;i<valArr.length;i++){
    newobj['"'+keyArr[i]+'"']=valArr[i];
  }
  console.log(newobj);
}

var obj = {
  "apple": 124,
  "banana": 23,
  "melon": 19,
  "pear": 100,
  "orange": 68
};

objsortbyval(obj);
```