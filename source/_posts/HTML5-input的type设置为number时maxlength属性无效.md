---
title: HTML5 input的type设置为number时maxlength属性无效
comments: true
toc: true
excerpt: ' '
date: 2019-05-31 10:54:33
categories: HTML5
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： André Dion
原文标题： maxlength ignored for input type=“number” in Chrome
链接： https://stackoverflow.com/questions/18510845/maxlength-ignored-for-input-type-number-in-chrome
来源： StackoverFlow
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

来自 [MDN 的文档`<input>`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input#attr-maxlength)

>如果type属性的值为`text`、`email`、`search`、`password`、`tel`、 或`url`，则该属性指定用户可以输入的最大字符数（以 Unicode 代码点表示）；对于其他控件类型，它被忽略。

所以`maxlength`被`<input type="number">`的设计忽略了。

根据您的需要，您可以使用他/她的答案中建议的min和max属性（注意：这只会定义一个受限范围，而不是值的实际字符长度，尽管 -9999 到 9999 将涵盖所有 0-4 digit numbers），或者您可以使用常规文本输入并使用pattern属性对字段执行验证：

``` html
<input type="text" pattern="\d*" maxlength="4">
```