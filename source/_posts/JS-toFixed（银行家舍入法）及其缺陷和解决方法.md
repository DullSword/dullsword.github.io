---
title: JS toFixed（银行家舍入法）及其缺陷和解决方法
comments: true
toc: true
excerpt: '所谓银行家舍入法，其实质是一种四舍六入五取偶（又称四舍六入五留双）法。'
date: 2018-11-12 12:20:27
categories: Javascript
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
所谓银行家舍入法，其实质是一种四舍六入五取偶（又称四舍六入五留双）法。

据说,大部分的编程软件都使用的是这种方法，也算是一种国际标准。 所谓银行家舍入法，其实质是一种四舍六入五取偶（又称四舍六入五留双）法。其规则是：当舍去位的数值小于5时，直接舍去该位；当舍去位的数值大于等于6时，在舍去该位的同时向前位进一；当舍去位的数值等于5时，如果前位数值为奇，则在舍去该位的同时向前位进一，如果前位数值为偶，则直接舍去该位。

简单的说，就是：**四舍六入五考虑，五后非零就进一，五后为零看奇偶，五前为偶应舍去，五前为奇要进一**。

![](line_segment.png)

<a class="tag is-dark is-medium" style="margin-bottom: 1rem" href="https://www.cnblogs.com/mapc/p/4848476.html" target="_blank">
<span class="icon"><i class="fas fa-camera"></i></span>&nbsp;&nbsp;
来源自 四舍五入VS银行家舍入法 - mapme
</a>

看上图，要更精确的取整，那么就产生了要舍位或者进位的问题。究竟是进位还是舍位，那还是得看概率的。P（小于0.5）=P（大于0.5）。这两种情况概率相等的结果就是，小于0.5理应舍位取整，大于0.5则进位取整。那么问题来了，万一某位数的小数点位为0.5呢？那到底向前进位还是向后舍位呢？为了更精确起见，python等一些语言的Round 函数则采用 [Banker's rounding（银行家舍入）算法](https://wiki.c2.com/?BankersRounding)，为0.5的时候进位舍位看小数点前个位是奇数还是偶数，是奇进位，是偶舍位。这也是银行家舍入算法的思想。

# 定义和用法

`toFixed()` 方法可把 `Number` 四舍五入为指定小数位数的数字。

# 语法

``` javascript
NumberObject.toFixed(num)
```

|参数|描述|
| ---- | ------ |
| num | 必需。规定小数的位数，是 0 ~ 20 之间的值，包括 0 和 20，有些实现可以支持更大的数值范围。如果省略了该参数，将用 0 代替。 |

# 返回值

返回 `NumberObject` 的字符串表示，不采用指数计数法，小数点后有固定的 `num` 位数字。如果必要，该数字会被舍入，也可以用 0 补足，以便它达到指定的长度。如果 `num` 大于 `le+21`，则该方法只调用 `NumberObject.toString()`，返回采用指数计数法表示的字符串。

# 抛出

当 `num` 太小或太大时抛出异常 `RangeError`。0 ~ 20 之间的值不会引发该异常。有些实现支持更大范围或更小范围内的值。

当调用该方法的对象不是 `Number` 时抛出 `TypeError` 异常。

# 解决方法

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： Scott
原文标题： toFixed计算错误(依赖银行家舍入法的缺陷)解决方法
链接： http://www.chengfeilong.com/toFixed
来源： Scott's Blog
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

5后为零(5为最后一位)： 由于这种情况比较特殊，是`toFixed`方法出现计算错误的情况。

``` javascript
var num = 0.005;
console.log(num.toFixed(2));  //0.01
```

``` javascript
var num = 0.015;
console.log(num.toFixed(2));  //0.01
```

通过重写`toFixed`的方法：

``` javascript
Number.prototype.toFixed = function(length)
{
    var carry = 0; //存放进位标志
    var num,multiple; //num为原浮点数放大multiple倍后的数，multiple为10的length次方
    var str = this + ''; //将调用该方法的数字转为字符串
    var dot = str.indexOf("."); //找到小数点的位置
    if(str.substr(dot+length+1,1)>=5) carry=1; //找到要进行舍入的数的位置，手动判断是否大于等于5，满足条件进位标志置为1
    multiple = Math.pow(10,length); //设置浮点数要扩大的倍数
    num = Math.floor(this * multiple) + carry; //去掉舍入位后的所有数，然后加上我们的手动进位数
    var result = num/multiple + ''; //将进位后的整数再缩小为原浮点数
    /*
    * 处理进位后无小数
    */
    dot = result.indexOf(".");
    if(dot < 0){
        result += '.';
        dot = result.indexOf(".");
    }
    /*
    * 处理多次进位
    */
    var len = result.length - (dot+1);
    if(len < length){
        for(var i = 0; i < length - len; i++){
            result += 0;
        }
    }
    return result;
}
```

该方法的大致思路是首先找到舍入位，判断该位置是否大于等于5，条件成立手动进一位，然后通过参数大小将原浮点数放大10的参数指数倍，然后再将包括舍入位后的位数利用`floor`全部去掉，根据我们之前的手动进位来确定是否进位。