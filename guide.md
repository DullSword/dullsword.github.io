# 图片名字全小写，单词之间用下划线隔开

![free_software_licenses](free_software_licenses.png)

# 缩写或专有名词保留大写

![upcoming_changes_to_Watcher_and_Star_APIs](upcoming_changes_to_Watcher_and_Star_APIs.png)

# 如果有序号，从0开始，与名字之间用 - 隔开

![code_running_process - 0](code_running_process-0.gif)

# 原文 使用link

https://ppoffice.github.io/hexo-theme-icarus/uncategorized/%E8%87%AA%E5%AE%9A%E4%B9%89hexo%E6%A0%87%E7%AD%BE%E6%8F%92%E4%BB%B6/#tab_title_boxed_1

{% message color:primary %}
作者： 淼淼真人
原文标题： 浅析javascript调用栈
链接： https://segmentfault.com/a/1190000010360316
来源： SegmentFault 思否
转载篇幅： 全文
是否修改： 否
{% endmessage %}

# 参考 放文末

- [常见几种排序之javascript实现 - 赵乘风_i](https://blog.csdn.net/zr15829039341/article/details/70598652)

# 相关信息或前提条件 使用info

https://ppoffice.github.io/hexo-theme-icarus/uncategorized/%E8%87%AA%E5%AE%9A%E4%B9%89hexo%E6%A0%87%E7%AD%BE%E6%8F%92%E4%BB%B6/#tab_title_boxed_1

{% message color:info %}
最小值n，最大值m
{% endmessage %}

# 图片来源

<a class="tag is-dark is-medium" style="margin-bottom: 1rem" href="https://blog.csdn.net/guoweimelon/article/details/50904206" target="_blank">
<span class="icon"><i class="fas fa-camera"></i></span>&nbsp;&nbsp;
来源自 经典排序算法（4）——折半插入排序算法详解 郭威gowill
</a>

# 行内代码不包括标点符号（除了代码的保留词）、编程语言、标题或处在标题内以及一些大的概念，如括号、javascript、HTTP

# 粗体只使用在除行内代码外需要强调的内容

# 分类标签 首字母大写 单词之间空格隔开 尽量使用中文，别装逼，等等看不懂，除了一些不好翻译过来的

# 使用中文引号，最后显示效果中英文引号是一致的

# 标签页

https://ppoffice.github.io/hexo-theme-icarus/uncategorized/%E8%87%AA%E5%AE%9A%E4%B9%89hexo%E6%A0%87%E7%AD%BE%E6%8F%92%E4%BB%B6/#tab_title_boxed_1

{% tabs style:boxed %}
<!-- tab id:original "icon:fa-solid fa-book" title:原文 active -->
（1）先运行绑定事件函数，把`onClick`事件绑定在`button`标签上。该函数没有没有调用其他函数。

（2）接下来运行`console.log("hi")`,该函数没有调用任何其他函数。

（3）然后继续执行下面的`setTimeout`，`setTimeout`是一个异步函数，经过5秒之后，在运行队列里面插入这个回调函数，然后如果该队列之前没有其他函数，就执行该队列，有则等待前面的函数执行完成，再执行。

（4）`console.log("My Name Is Chirs")`不会等待5s之后，再执行，因为`settimeout`并不会在调用栈中执行5秒，实际上它在调用栈中是立即执行完的。

（5）假设在这个时候，我们点击了按钮，按钮绑定的回调事件被添加到运行队列中。（运行队列中的代码要等调用栈被清空之后才会执行）由于调用栈中还有代码需要执行，所以会继续执行下面的`console.log()`

（6）然后执行完`console.log`之后，由于时间还没有经过5s,所以点击的回调事件会被先压入栈中去执行，由于该回调事件里面又是一个`settimeout`事件，由于它的事件间隔只有0s，所以这个`settimeout`的回调会先被压入运行队列。先输出"`You clicked the button!`" 再过几秒之后，间隔为5s的`settimeout`把回调函数压入队列，这时候调用栈中没有代码在执行，所以会执行这个代码，输出"`Click the button`"。结束代码运行。
<!-- endtab -->
<!-- tab id:personal "icon:fa-solid fa-user" title:个人理解 -->
（1）运行`console.log("hi")`，该函数没有调用任何其他函数。

（2）然后继续执行下面的`setTimeout`，`setTimeout`是一个异步函数，经过5秒之后，在运行队列里面插入这个回调函数，然后如果该队列之前没有其他函数，就执行该队列，有则等待前面的函数执行完成，再执行。

（3）`console.log("My Name Is Chirs")`不会等待5s之后，再执行，因为`settimeout`并不会在调用栈中执行5秒，实际上它在调用栈中是立即执行完的。

（4）假设在要`console.log("My Name Is Chirs")`时，我们点击了按钮，按钮绑定的回调事件被添加到运行队列中。（运行队列中的代码要等调用栈被清空之后才会执行）由于调用栈中还有代码需要执行，所以会继续执行下面的`console.log("My Name Is Chirs")`。

（5）然后执行完`console.log("My Name Is Chirs")`之后，由于时间还没有经过5s,所以点击按钮的回调事件会被先压入栈中去执行，由于该回调事件里面又是一个`settimeout`事件，由于它的事件间隔只有0s，所以这个`settimeout`的回调会先被压入运行队列。先输出"`You clicked the button!`"，然后5s间隔到了的`settimeout`把回调函数压入队列，这时候调用栈中没有代码在执行，所以会执行这个代码，输出"`Click the button`"。结束代码运行。
<!-- endtab -->
{% endtabs %}

# 参数说明+使用+效果+代码

# 有步骤顺序的标题才用"1.空格"这样的标注

"## 1. 点击续费购买证书"

# 隐藏文字

{% raw %}
<style type="text/css">
.heimu { color: #000; background-color: #000; }
.heimu:hover { color: #fff; }
</style>
{% endraw %}
**iMaeGoo** 出自独立游戏 [World of Goo](https://store.steampowered.com/app/22000/) 里小粘球的叫声，读作 /ɪ'mæɡu/ {% raw %}<span class="heimu">不是爱妹狗啊</span>{% endraw %}，在家里电脑还是个大头（CRT）的时候就在玩了，其实头像也是在当时设定的，一直沿用至今。{% raw %}<span class="heimu">找不到女朋友誓不改头像</span>{% endraw %}

# 复制按钮 主要是需要添加copy样式

<button class="button copy" data-clipboard-text="YH3WN-NKHFT-61FSS-E49FT"><i class="fas fa-copy"></i></button>

# 引用

https://github.com/MaxChang3/hexo-reference-plus/blob/main/README_CN.md

front-matter添加refplus: true

测试{% ref mcf %}

{% references %}
[mcf]  Max C. Foo.  A way to write an article.[J] Journal of Kelaideng University Samwin School. 2021.3 300-321.
{% endreferences %}

# B站视频

替换aid、bid、cid

<div style="position: relative; width: 100%; height: 0; padding-bottom: 75%;">
    <iframe src="//player.bilibili.com/player.html?aid=294966176&bvid=BV12F411q7CA&cid=464566582&page=1"  scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" style="position: absolute; width: 100%; height: 100%; left: 0; top: 0;"></iframe>
</div>