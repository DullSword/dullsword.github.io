---
title: 浅析javascript调用栈
comments: true
toc: true
excerpt: '代码在运行过程中，会有一个叫做调用栈(`call stack`)的概念。调用栈是一种栈结构,它用来存储计算机程序执行时候其活跃子程序的信息。（比如什么函数正在执行，什么函数正在被这个函数调用等等信息）。调用栈是解析器的一种机制。'
date: 2018-09-28 22:24:27
categories: Javascript
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 淼淼真人
原文标题： 浅析javascript调用栈
链接： https://segmentfault.com/a/1190000010360316
来源： SegmentFault 思否
转载篇幅： 全文
是否修改： 是
{% raw %}</div></article>{% endraw %}

看原文[一个更复杂的例子](#一个更复杂的例子)的时候真的是一头雾水，说明和图怎么有点搭不上，后面看图去揣测才明白是什么意思。为了下次方便理解，稍微修改了下说明，让自己不会理解错导致又花时间。

# 什么是调用栈

代码在运行过程中，会有一个叫做调用栈(`call stack`)的概念。调用栈是一种栈结构,它用来存储计算机程序执行时候其活跃子程序的信息。（比如什么函数正在执行，什么函数正在被这个函数调用等等信息）。调用栈是解析器的一种机制。[call stack](https://en.wikipedia.org/wiki/Call_stack)

我们以一段简单代码为示例，来看一看到底什么是调用栈，它是一个怎么样的运行机制

``` Javascript
function boo (a) {
    return a * 3
}
function foo (b) {
    return boo(4) * 2
}
console.log(foo(3))
```

# 详解代码执行

下面我们来分析一下上述代码的执行过程

（1）`console.log(foo(3))` 执行，形成一个栈帧，调用`foo`函数，再形成另一个栈帧。

（2）新的栈帧压在上一个栈帧之上，继续执行代码，`foo`函数中又调用了`boo`函数，形成了另一个栈帧压在旧栈帧之上。然后执行`boo`。

（3）当执行完`boo`时候，返回值给`foo`函数之后，`boo`被推出调用栈，`foo`函数继续执行，然后`foo`函数执行完，返回值给`console.log`,`foo`函数被推出调用栈，`console.log`得到`foo`函数的返回值，运行，输出结果，最后`console.log`也被推出调用栈，该段程序执行完成。

图解代码运行过程：

![](code_running_process-0.gif)

# 一个更复杂的例子

``` Javascript
// 省略一部分html
<button>click</button>
$.on('button', 'click', function onClick() {
    setTimeout(function timer() {
        console.log('You clicked the button!');    
    }, 0);
});
 
console.log("Hi!");
 
setTimeout(function timeout() {
    console.log("Click the button!");
}, 5000);
 
console.log("My Name Is Chirs.")
```

大家看看上叙的代码，结合一下前面的的分析，思考一下调用栈是怎么工作的？

<div class="tabs is-boxed"><ul>
<li class="is-active"><a onclick="onTabClick(event)">
<span class="icon is-small"><i class="far fa-file-alt" aria-hidden="true"></i></span>
<span>原文</span>
</a></li>
<li><a onclick="onTabClick(event)">
<span class="icon is-small"><i class="far fa-file-alt" aria-hidden="true"></i></span>
<span>个人理解</span>
</a></li>
</ul></div>

{% raw %}<div id="原文" class="tab-content" style="display: block;">{% endraw %}
（1）先运行绑定事件函数，把`onClick`事件绑定在`button`标签上。该函数没有没有调用其他函数。

（2）接下来运行`console.log("hi")`,该函数没有调用任何其他函数。

（3）然后继续执行下面的`setTimeout`，`setTimeout`是一个异步函数，经过5秒之后，在运行队列里面插入这个回调函数，然后如果该队列之前没有其他函数，就执行该队列，有则等待前面的函数执行完成，再执行。

（4）`console.log("My Name Is Chirs")`不会等待5s之后，再执行，因为`settimeout`并不会在调用栈中执行5秒，实际上它在调用栈中是立即执行完的。

（5）假设在这个时候，我们点击了按钮，按钮绑定的回调事件被添加到运行队列中。（运行队列中的代码要等调用栈被清空之后才会执行）由于调用栈中还有代码需要执行，所以会继续执行下面的`console.log()`

（6）然后执行完`console.log`之后，由于时间还没有经过5s,所以点击的回调事件会被先压入栈中去执行，由于该回调事件里面又是一个`settimeout`事件，由于它的事件间隔只有0s，所以这个`settimeout`的回调会先被压入运行队列。先输出"`You clicked the button!`" 再过几秒之后，间隔为5s的`settimeout`把回调函数压入队列，这时候调用栈中没有代码在执行，所以会执行这个代码，输出"`Click the button`"。结束代码运行。
{% raw %}</div>{% endraw %}

{% raw %}<div id="个人理解" class="tab-content">{% endraw %}
（1）运行`console.log("hi")`，该函数没有调用任何其他函数。

（2）然后继续执行下面的`setTimeout`，`setTimeout`是一个异步函数，经过5秒之后，在运行队列里面插入这个回调函数，然后如果该队列之前没有其他函数，就执行该队列，有则等待前面的函数执行完成，再执行。

（3）`console.log("My Name Is Chirs")`不会等待5s之后，再执行，因为`settimeout`并不会在调用栈中执行5秒，实际上它在调用栈中是立即执行完的。

（4）假设在要`console.log("My Name Is Chirs")`时，我们点击了按钮，按钮绑定的回调事件被添加到运行队列中。（运行队列中的代码要等调用栈被清空之后才会执行）由于调用栈中还有代码需要执行，所以会继续执行下面的`console.log("My Name Is Chirs")`。

（5）然后执行完`console.log("My Name Is Chirs")`之后，由于时间还没有经过5s,所以点击按钮的回调事件会被先压入栈中去执行，由于该回调事件里面又是一个`settimeout`事件，由于它的事件间隔只有0s，所以这个`settimeout`的回调会先被压入运行队列。先输出"`You clicked the button!`"，然后5s间隔到了的`settimeout`把回调函数压入队列，这时候调用栈中没有代码在执行，所以会执行这个代码，输出"`Click the button`"。结束代码运行。
{% raw %}</div>{% endraw %}

---

<style type="text/css">
.content .tabs ul { margin: 0; }
.tab-content { display: none; }
</style>

<script>
function onTabClick (event) {
    var tabTitle = $(event.currentTarget).children('span:last-child').text();
    $('.article .content .tab-content').css('display', 'none');
    $('.article .content .tabs li').removeClass('is-active');
    $('#' + tabTitle).css('display', 'block');
    $(event.currentTarget).parent().addClass('is-active');
}
</script>

同样来看一个运行示意图

![](code_running_process-1.gif)

# 总结

调用栈其实就是一种解析器去处理程序的机制，它是栈数据结构。它能追踪子程序的运行状态。

（1）当脚本要调用一个函数时，解析器把该函数添加到栈中并且执行这个函数。并形成一个栈帧

（2）任何被这个函数调用的函数会进一步添加到调用栈中，形成另一个栈帧,并且运行到它们被上个程序调用的位置。

（3）当执行完这个函数后，如果它没有调用其他函数，则它会从调用栈中推出。然后调用栈继续运行其他部门。

（4）异步函数的回调函数一般都会被添加到运行队列里面，如`settimeout`会在响应的时间后把回调函数放入队列中，队列里的函数需要等栈为空时才会被推入栈中执行。如果队列中有其他函数，需要等队列前面的函数被堆入调用栈中之后才会运行。