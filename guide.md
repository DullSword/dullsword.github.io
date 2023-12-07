# 图片名字全小写，单词之间用下划线隔开，一般不需要添加说明

![](free_software_licenses-0.png)

# 缩写或专有名词保留大写

![upcoming_changes_to_Watcher_and_Star_APIs](upcoming_changes_to_Watcher_and_Star_APIs.png)

# 如果有序号，从0开始，与名字之间用 - 隔开

![code_running_process - 0](code_running_process-0.gif)

# 原文 使用link

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 淼淼真人
原文标题： 浅析javascript调用栈
链接： https://segmentfault.com/a/1190000010360316
来源： SegmentFault 思否
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

# 参考 放文末

- [常见几种排序之javascript实现 - 赵乘风_i](https://blog.csdn.net/zr15829039341/article/details/70598652)

# 相关信息或前提条件 使用info

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
最小值n，最大值m
{% raw %}</div></article>{% endraw %}

# 图片来源

<a class="tag is-dark is-medium" style="margin-bottom: 1rem" href="https://blog.csdn.net/guoweimelon/article/details/50904206" target="_blank">
<span class="icon"><i class="fas fa-camera"></i></span>&nbsp;&nbsp;
来源自 经典排序算法（4）——折半插入排序算法详解 郭威gowill
</a>

# 行内代码不包括标点符号（除了代码的保留词）、编程语言、标题或处在标题内以及一些大的概念，如括号、javascript、HTTP

# 粗体只使用在除行内代码外需要强调的内容

# 分类标签 首字母大写 单词之间空格隔开 尽量使用中文，别装逼，等等看不懂，除了一些不好翻译过来的

# 使用英文引号

# 标签页

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
{% raw %}</div>{% endraw %}

{% raw %}<div id="个人理解" class="tab-content">{% endraw %}
（1）运行`console.log("hi")`，该函数没有调用任何其他函数。
{% raw %}</div>{% endraw %}

---

<style type="text/css">
.content .tabs ul { margin: 0; }
.tab-content { display: none; margin-bottom: 1rem }
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

# 复制按钮 主要是样式copy

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