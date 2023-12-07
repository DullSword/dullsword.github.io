---
title: JS 匿名函数
comments: true
toc: true
excerpt: ' '
date: 2018-08-14 14:21:54
categories: Javascript
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 一口没水的枯井 & 红绿蓝的那个黄 & lanlingxueyu
原文标题： javascript 匿名函数的用途到底是啥？
链接： https://zhidao.baidu.com/question/242686709298633004.html?sort=9&rn=5&pn=0#wgt-answers
来源： 百度知道
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

1. 分析: 函数为何要有名字? 是为了方便下次使用. 匿名函数, 即没有名字.
2. 用途: 通常不希望再次使用(即只使用一次的)的函数可以定义为匿名函数.
3. 使用示例:
   
    ``` javascript
// 定义并使用一个匿名函数来打印从1到10的整数
(function (n) {
    for (var i = 1; i <= n; i++)
        console.log(i);
})(10);
```

4. 扩展: 当然, 如果还是想再次使用匿名函数的话, 也有方法. 即把匿名函数赋给一个变量(`funtion`类型的变量), 想再次使用时, 使用该变量来调用即可.示例如下:
   
    ``` javascript
// 定义一个匿名函数并将它赋给变量printN
var printN = function (n) {
    for (var i = 1; i <= n; i++)
        console.log(i);
};
// 通过变量printN来再次使用该匿名函数
printN(10);
```

---

匿名函数可以有效的保证在页面上写入Javascript，而不会造成全局变量的污染。说白了就是隔离作用域，避免命名上的冲突。

---

JS中的匿名函数的用法及优缺点
匿名函数可以有效的保证在页面上写入Javascript，而不会造成全局变量的污染。

这在给一个不是很熟悉的页面增加Javascript时非常有效，也很优美。

1. **什么是匿名函数**？

    在Javascript定义一个函数一般有如下三种方式：

    函数关键字(`function`)语句：

    ``` javascript
function fnMethodName(x){
    alert(x);
}
```

    函数字面量(`Function Literals`)：

    ``` javascript
var fnMethodName = function(x){
    alert(x);
}
```

    `Function()`构造函数：

    ``` javascript
var fnMethodName = new Function('x','alert(x);')
```

    上面三种方法定义了同一个方法函数`fnMethodName`，

    第1种就是最常用的方法，后两种都是把一个函数复制给变量`fnMethodName`，而这个函数是没有名字的，即匿名函数。

    实际上，相当多的语言都有匿名函数。

2. **函数字面量和Function()构造函数的区别**

    虽然函数字面量是一个匿名函数，但语法允许为其指定任意一个函数名，当写递归函数时可以调用它自己，使用`Function()`构造函数则不行。

    ```javascript
    var f = function fact(x) { if (x < = 1) return 1; else return x*fact(x-1); };
    ```

    `Function()`构造函数允许运行时Javascript代码动态的创建和编译。在这个方式上它类似全局函数`eval()`。

    `Function()`构造函数每次执行时都解析函数主体，并创建一个新的函数对象。所以当在一个循环或者频繁执行的函数中调用`Function()`构造函数的效率是非常低的。相反，函数字面量却不是每次遇到都重新编译的。

    用`Function()`构造函数创建一个函数时并不遵循典型的作用域，它一直把它当作是顶级函数来执行。

    ``` javascript
var y = "global";
 
function constructFunction() {
  var y = "local"; 
  return new Function("return y"); // 无法获取局部变量 
}

alert(constructFunction()()); // 输出 “global”和函数关键字定义相比Function()构造器有自己的特点且要难以使用的多
```

    所以这项技术通常很少使用。

    而函数字面量表达式和函数关键字定义非常接近。

    考虑前面的区别，虽然有消息说字面量的匿名函数在`OS X 10.4.3`下的某些`webkit`的引擎下有bug，

    但我们平常所说的匿名函数均指采用函数字面量形式的匿名函数。

3. **匿名函数的代码模式**

    错误模式：其无法工作，浏览器会报语法错。

    ``` javascript
function () { 
    alert(1); 
}();
```

    函数字面量：首先声明一个函数对象，然后执行它。

    ``` javascript
(function () { 
    alert(1);
})();
```

    优先表达式：

    ``` javascript
(function () {
  alert(2);
}());
```

    void操作符：

    ``` javascript
void function () {
  alert(3);
}()
```

    这三种方式是等同的，hedger wang因为个人原因比较喜欢第3种，而在实际应用中我看到的和使用的都是第1种。

1. **匿名函数的应用**

    《Javascript的一种模块模式》中的第一句话就是“全局变量是魔鬼”。

     配合var关键字，匿名函数可以有效的保证在页面上写入Javascript，而不会造成全局变量的污染。

    这在给一个不是很熟悉的页面增加Javascript时非常有效，也很优美。

    实际上，YUI以及其相应的范例中大量使用匿名函数，其他的Javascript库中也不乏大量使用。

    Javascript的函数式编程(`functional programming`)的基石。