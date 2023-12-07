---
title: JS 闭包
comments: true
toc: false
excerpt: '闭包的本质在于“闭”和“包”，即把一些变量封闭起来，使其它程序访问不到，同时把这个封闭的东西打成包甩出来，让大家可以直接用这个包（函数）。'
date: 2018-08-02 14:45:37
categories: Javascript
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 边城
原文标题： 立即执行函数和闭包有什么关系
链接： https://segmentfault.com/q/1010000005007819
来源： SegmentFault 思否
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

闭包的本质在于“闭”和“包”，即把一些变量封闭起来，使其它程序访问不到，同时把这个封闭的东西打成包甩出来，让大家可以直接用这个包（函数）。最典型的实现之一是对象（或类）的私有成员，如

``` JavaScript
function MyClass() {
    // 这是一个封闭在 MyClass 中的局部变量
    var _name;
    
    // 这是一个甩出来的“包”
    this.getName = function() {
        return _name;
    };
    
    // 这是另一个甩出来的“包”
    this.setName = function(name) {
        // 这保证了 _name 的第一个字母和空格后的第一个字母是大写
        // 而且因为闭包的原因，_name 不可能被 MyCLass() 外的程序访问到
        // 也就保证了上述命名规则的无例外执行
        _name = name.replace(/^.|\s./g, function(s) {
            return s.toUpperCase();
        });
    };
}
 
var p = new MyClass();
p.setName("james fancy");
console.log(p.getName());   // James Fancy
```

匿名函数通常只是用于给成员赋值，比如上例中的 `getName` 和 `setName`；也有可能用于立即执行函数，比如你的那段代码，这会将全局变量局部化，避免全局污染。

闭包常常会和匿名函数一起使用，但他们之间并没有密不可分的关系。

# 参考

- [立即执行函数和闭包有什么关系？](https://segmentfault.com/q/1010000005007819)
- [面试题：为什么要用闭包？](https://segmentfault.com/q/1010000007578832?_ea=1390620)
- [JS闭包是什么？ - 懿左左](https://blog.csdn.net/weixin_39194176/article/details/80909666)
- [ECMAScipt闭包（closure）](http://www.w3school.com.cn/js/pro_js_functions_closures.asp)
- [闭包是函数与声明函数的词法语境的组合](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)