---
title: JS this是什么
comments: true
toc: true
excerpt: '`this`是包含它的函数作为方法被调用时所属的对象。'
date: 2018-08-23 14:30:09
categories: Javascript
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 未知
原文标题： JavaScript：this是什么？
链接： http://hi.baidu.com/tkocn/blog/item/7c66bd02f7395b084afb5150.html
来源： 百度空间
转载篇幅： 未知
是否修改： 未知
{% raw %}</div></article>{% endraw %}

定义：`this`是包含它的函数作为方法被调用时所属的对象。

说明：这句话有点咬嘴，但一个多余的字也没有，定义非常准确，我们可以分3部分来理解它！

1、包含它的函数。2、作为方法被调用时。3、所属的对象。

看例子：

``` javascript
function to_green(){
    this.style.color = "green";
}
to_green();
```

上面函数中的`this`指的是谁？

分析：包含`this`的函数是，`to_green`

该函数作为方法被调用了

该函数所属的对象是。。？我们知道默认情况下，都是`window`对象。

OK，`this`就是指的`window`对象了，to_green中执行语句也就变为，`window.style.color="green"`

这让`window`很上火，因为它并没有`style`这么个属性，所以该语句也就没什么作用。

我们再改一下。

``` javascript
window.load = function(){
    var example = document.getElementById("example");
    example.onclick = to_green;
}
```

这时`this`又是什么呢？

我们知道通过赋值操作，`example`对象的`onclick`得到`to_green`的方法，那么包含`this`的函数就是onclick喽，

那么`this`就是`example`引用的`html`对象喽。

`this`的环境可以随着函数被赋值给不同的对象而改变！

下面是完整的例子：
 
``` javascript
<script type="text/javascript">
    function to_green(){
        this.style.color = "green";
    }
    function init_page(){
        var example = document.getElementById("example");
        example.onclick = to_green;
    }
    window.onload = init_page;
</script>
<a href="#" id="example">点击变绿</a>
```