---
title: Cocos Creator发布微信小游戏问题
comments: true
toc: true
excerpt: ' '
date: 2018-07-08 20:19:36
categories: 
- [Cocos Creator]
- [微信小游戏]
tags: Cocos Creator
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： jare
原文标题： 小游戏在微信开发者工具中的问题汇总（FAQ）
链接： https://forum.cocos.com/t/faq/54828
来源： Cocos中文社区
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

“未找到入口`app.json`文件，或者文件读取失败，请检查后重新编译”，这个是微信平台的权限问题，微信公众平台那边必须设置成小游戏服务类目。也就是说，你的小程序 `appid` 类别应该是游戏。