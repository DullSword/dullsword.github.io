---
title: JS 判断是否同一天、同一周
comments: true
toc: true
excerpt: ' '
date: 2019-03-01 18:20:57
categories: Javascript
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
**判断是否同一天** ：

``` javascript
isSameDay(timeStampA, timeStampB) {
    let dateA = new Date(timeStampA);
    let dateB = new Date(timeStampB);
    return (dateA.setHours(0, 0, 0, 0) == dateB.setHours(0, 0, 0, 0));
}
```

**判断是否同一周**：

<div class="tabs is-boxed"><ul>
<li class="is-active"><a onclick="onTabClick(event)">
<span class="icon is-small"><i class="far fa-file-alt" aria-hidden="true"></i></span>
<span>思路1</span>
</a></li>
<li><a onclick="onTabClick(event)">
<span class="icon is-small"><i class="far fa-file-alt" aria-hidden="true"></i></span>
<span>思路2</span>
</a></li>
</ul></div>

{% raw %}<div id="思路1" class="tab-content">{% endraw %}

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： WoodLikeWater
原文标题： js判断两个输入日期为同一周的
链接： https://bbs.csdn.net/topics/350061349
来源： CSDN社区
转载篇幅： 部分
是否修改： 是
{% raw %}</div></article>{% endraw %}

计算出 现在距离1970年1月1日的总天数，因为1970年1月1 是周4   所以（总天数+7）/7 取整 就是周数  如果相同就是同一周反之就不是。

``` javascript
isSameWeek(timeStampA, timeStampB) {
    let A = new Date(timeStampA).setHours(0, 0, 0, 0);
    let B = new Date(timeStampB).setHours(0, 0, 0, 0);
    var oneDayTime = 1000 * 60 * 60 * 24;
    var old_count = parseInt(A / oneDayTime);
    var now_other = parseInt(B / oneDayTime);
    return parseInt((old_count + 4) / 7) == parseInt((now_other + 4) / 7);
}
```
{% raw %}</div>{% endraw %}

{% raw %}<div id="思路2" class="tab-content" style="display: block;">{% endraw %}

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： eric_gg_m4
原文标题： 单机游戏实现周签到，判断是否在同一周
链接： https://blog.csdn.net/weixin_41783625/article/details/82706680
来源： CSDN博客
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

获取到目前的时间，然后转化到今天的凌晨的时间点的毫秒数，然后再去拉取今天星期几，再往前推对应的天数，找到当前天数所在的周一的凌晨点毫秒数，比对之前存储的数值，相同的话就是同一周，处理。不同的话就说明不是同一周，再覆盖存储周一的值，再处理。

``` javascript
whetherSameWeek(myDate){
    var myDate = new Date() ;
    let nowWeek = myDate.getDay(); //获取当前星期X(0-6,0代表星期天)
    if(nowWeek == 0) nowWeek = 7;
    nowWeek --;

    let nowTime = myDate.getTime();  //获取当前时间(从1970.1.1开始的毫秒数)
    let hour = myDate.getHours();  //获取当前小时数(0-23)
    let minute = myDate.getMinutes();  //获取当前分钟数(0-59)
    let second = myDate.getSeconds();  //获取当前秒数(0-59)
    let milSecond = myDate.getMilliseconds();  //获取当前毫秒数(0-999)
    let mondayDate = cc.sys.localstorage.getltem("MondayDate"); //周一的时间  用来判断是否是新的一周

    //获取目前天的凌晨毫秒数
    let todayMidnight = nowTime - milSecond - second*1000 - minute*60*1000 - hour*60*60*1000;
    //获取当前天所在周的周一凌晨毫秒数
    let nowWeekMonday = todayMidnight - nowWeek*(24*60*60*1000);
    if (nowWeekMonday == mondayDate){  //同一周
        return true;
    }
    else{
        //同步
        cc.sys.localstorage.setltem("MondayDate",nowWeekMonday)
        return false;
    }
}
```
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