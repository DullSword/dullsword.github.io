---
title: PM2与守护进程
comments: true
toc: true
excerpt: '线上一直都是用pm2 启动服务，然而并不知道有什么用🤷，后来知道是守护进程的，那守护进程又是什么呢?'
date: 2020-12-09 16:48:11
categories: Node.js
tags:
sticky: 0
recommend: 0
---
# PM2

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
PM2 is daemon process manager that will help you manage and keep your application online.

PM2是守护进程管理器，它将帮助您管理和保持应用程序在线
{% raw %}</div></article>{% endraw %}

线上一直都是用pm2 启动服务，然而并不知道有什么用🤷，后来知道是守护进程的，那守护进程又是什么呢?

# 守护进程

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 五月君
原文标题： Node.js进阶之进程与线程
链接： https://segmentfault.com/a/1190000019496158
来源： SegmentFault 思否
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

>关于守护进程，是什么、为什么、怎么编写？本节将解密这些疑点

守护进程运行在后台不受终端的影响，什么意思呢？Node.js 开发的同学们可能熟悉，当我们打开终端执行 <font color="#e83e8c">node app.js</font> 开启一个服务进程之后，这个终端就会一直被占用，如果关掉终端，服务就会断掉，即前台运行模式。如果采用守护进程进程方式，这个终端我执行 <font color="#e83e8c">node app.js</font> 开启一个服务进程之后，我还可以在这个终端上做些别的事情，且不会相互影响。