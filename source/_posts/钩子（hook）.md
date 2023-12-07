---
title: 钩子（hook）
comments: true
toc: false
excerpt: '钩子来源于Hook，在windows系统中，一切都是消息，按了一下键盘，就是一个消息，Hook的意思就是勾住，在消息来的时候把消息勾住，不让其执行，然后自己优先处理。也就是这个技术提供了一个入口，能够针对不同的消息或者api在执行前或后（前置钩子或后置钩子），先执行你的操作。"你的操作"就是钩子函数。'
date: 2019-03-04 17:43:56
categories: 其他
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 仇诺伊
原文标题： 钩子（hook）是啥
链接： https://www.jianshu.com/p/27374378baca
来源： 简书
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

钩子来源于Hook，在windows系统中，一切都是消息，按了一下键盘，就是一个消息，Hook的意思就是勾住，在消息来的时候把消息勾住，不让其执行，然后自己优先处理。也就是这个技术提供了一个入口，能够针对不同的消息或者api在执行前或后（前置钩子或后置钩子），先执行你的操作。"你的操作"就是钩子函数。

---

# e.g.

React生命周期：componentDidMount(),（后置钩子）componentWillUnmount()（前置钩子）

Javascript：回调函数（后置钩子）

# 参考

- [钩子（hook）是啥 - 仇诺伊](https://www.jianshu.com/p/27374378baca)
- [麻烦帮解释一下，什么叫“钩子”？](https://segmentfault.com/q/1010000004335505)