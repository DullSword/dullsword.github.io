---
title: 'JS obeject.key与object[key]的区别'
comments: true
toc: true
excerpt: ' '
date: 2018-10-31 20:59:35
categories: Javascript
tags: Javascript对象
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： kumfo & 条件状语从句 & xxxxxxxxx
原文标题： obeject.key与object[key]有什么区别
链接： https://segmentfault.com/q/1010000004225321
来源： SegmentFault 思否
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

`data[key]`适用于动态取`key`、`key`为特殊字符。

`data.key`访问的是`data`对象下的`key`。
`data[key]`访问的是`data`数组的下标为`key`的值（对象是可以以数组形式来访问的）。

`data.key`这里的`key`必须是引用值。

`data[key]`这里的`key`必须是字面量。

当你的属性名包含了**空格**时，**必须采用中括号的写法**。

``` javascript
let obj = {};
obj['first name'] = 'mike';
```

以上情形，只有通过`[]`语法才能获取'`first name`'，因为其中有空格，用`.`语法怎么也取不到。