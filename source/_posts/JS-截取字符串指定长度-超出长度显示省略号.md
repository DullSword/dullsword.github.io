---
title: 'JS 截取字符串指定长度 超出长度显示省略号'
comments: true
toc: true
excerpt: '看得懂的，可以自己写一下。看不懂的，直接复制代码调用函数即可。'
date: 2019-03-04 17:51:02
categories: Javascript
tags: Javascript字符串
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： Leo-hzj
原文标题： js截取字符串指定长度,超出长度显示省略号
链接： https://blog.csdn.net/q36835109/article/details/53009150
来源： CSDN博客
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

看得懂的，可以自己写一下。看不懂的，直接复制代码调用函数即可。

废话不多说，直接上代码了！

``` javascript
/**
    * 函数说明：获取字符串长度
    * 备注：字符串实际长度，中文2，英文1
    * @param:需要获得长度的字符串
    */
function getStrLength(str) {
    var realLength = 0, len = str.length, charCode = -1;
    for (var i = 0; i < len; i++) {
        charCode = str.charCodeAt(i);
        if (charCode >= 0 && charCode <= 128){
            realLength += 1;
        }else{
            realLength += 2;
        }
    }
    return realLength;
}
/**
    * js截取字符串，中英文都能用
    * @param str：需要截取的字符串
    * @param len: 需要截取的长度
    */
function cutstr(str, len) {
    var str_length = 0;
    var str_len = 0;
    str_cut = new String();
    str_len = str.length;
    for (var i = 0; i < str_len; i++) {
        a = str.charAt(i);
        str_length++;
        if (escape(a).length > 4) {
            //中文字符的长度经编码之后大于4
            str_length++;
        }
        str_cut = str_cut.concat(a);
        if (str_length >= len) {
            str_cut = str_cut.concat("...");
            return str_cut;
        }
    }
    //如果给定字符串小于指定长度，则返回源字符串；
    if (str_length < len) {
        return str;
    }
}

console.log(getStrLength('qwrtyuiop'));
console.log(cutstr('qwrtyuiop','5'));
```