---
title: Cocos Creator 重力感应 手机休眠
comments: true
toc: true
excerpt: ' '
date: 2018-07-12 23:47:13
categories: Cocos Creator
tags: Cocos Creator
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： natural-law
原文标题： 怎么让手机不进入休眠状态？
链接： http://forum.cocos.com/t/topic/41000/6
来源： Cocos中文社区
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

在 `native` 环境中（C++ 代码），引擎封装了一个 `cocos2d::Device::setKeepScreenOn(true/false)` 的接口。并且该接口已绑定到 js 层。可以试一下 `cc.Device.setKeepScreenOn()` 来设置不锁屏。