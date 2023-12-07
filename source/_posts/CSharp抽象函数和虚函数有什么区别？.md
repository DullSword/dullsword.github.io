---
title: C# 抽象函数和虚函数有什么区别？
comments: true
toc: true
excerpt: ' '
date: 2020-01-03 17:34:35
categories: CSharp
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： BFree
原文标题： What is the difference between an abstract method and a virtual method?
链接： https://stackoverflow.com/questions/391483/what-is-the-difference-between-an-abstract-method-and-a-virtual-method
来源： StackoverFlow
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

>**An abstract function cannot have functionality**. You're basically saying, any child class MUST give their own version of this method, however it's too general to even try to implement in the parent class.

**抽象函数不能具有功能**。基本上是在说，任何子类都必须提供自己的该方法的版本，但是它太笼统了，甚至无法尝试在父类中实现。

>**A virtual function**, is basically saying look, here's the functionality that may or may not be good enough for the child class. So if it is good enough, use this method, if not, then override me, and provide your own functionality.

**虚函数**基本上是在说，看，这里的功能对于子类来说可能足够好，也可能不够好。因此，如果足够好，请使用此方法；否则，请覆盖我并提供您自己的功能。