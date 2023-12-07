---
title: Cocos Creator在手机浏览器中运行时点击会导致全屏
comments: true
toc: true
excerpt: ' '
date: 2018-08-12 00:27:14
categories: Cocos Creator
tags: Cocos Creator
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 李某龙
原文标题： cocos creator踩坑日记
链接： https://www.cnblogs.com/slly/p/6382159.html
来源： 博客园
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

问题：项目在构建成`Web Mobile`后运行在浏览器和微信中，点击页面任何地方都会导致自动全屏

解决：在构建之后的`main.js`中，去掉`cc.view.enableAutoFullScreen(true)`或者手动改写成`cc.view.enableAutoFullScreen(false)`

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
实际测试中，在微信中打开并不会有该问题，可能已经调整过
{% raw %}</div></article>{% endraw %}