---
title: JS 数组打乱 Fisher–Yates shuffle(费舍尔-耶茨 洗牌)
comments: true
toc: true
excerpt: ' '
date: 2020-11-12 18:23:32
categories: Javascript
tags: Javascript数组
sticky: 0
recommend: 0
article:
    licenses:
---
# 原理

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： Lucas HC
原文标题： 如何将一个 JavaScript 数组打乱顺序？
链接： https://www.zhihu.com/question/68330851/answer/266506621
来源： 知乎
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

**Fisher–Yates shuffle 洗牌算法是什么，为什么满足需求**？

这里，我们简单借助图形来理解，非常简单直观。你接下来就会明白为什么这是理论上的完全乱序（图片来源于网络）。

首先我们有一个已经排好序的数组：

![](principle-0.png)

## Step1

第一步需要做的就是，从数组末尾开始，选取最后一个元素。

![](principle-1.png)

在数组一共 9 个位置中，随机产生一个位置，该位置元素与最后一个元素进行交换。

![](principle-2.png)

![](principle-3.png)

![](principle-4.png)

## Step2

上一步中，我们已经把数组末尾元素进行随机置换。
接下来，对数组倒数第二个元素动手。在除去已经排好的最后一个元素位置以外的8个位置中，随机产生一个位置，该位置元素与倒数第二个元素进行交换。

![](principle-5.png)

![](principle-6.png)

![](principle-7.png)

## Step3

理解了前两部，接下来就是依次进行，如此简单。

![](principle-8.png)

# 代码

## 测试代码

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： troy351
原文标题： 如何将一个 JavaScript 数组打乱顺序？
链接： https://www.zhihu.com/question/68330851/answer/266506621
来源： 知乎
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

``` javascript
const Test = func => {
    // 以下均测试10000000次打乱，记录最后的结果于一个二维数组count
    // count[i][j]即表示数字i在j位置出现的次数
    const total = 10000000;
    const count = new Array(5).fill(0).map(() => new Array(5).fill(0));
    for (let i = 0; i < total; i++) {
        func().forEach((n, i) => count[n][i]++);
    }
    // 应题主要求输出改为百分比，四舍五入保留2位小数
    console.table(count.map(n => n.map(n => (n / total * 100).toFixed(2) + '%')));
};
```

## 从前往后，可能保持原位

- 因为可能和自己交换，相当于没交换，留在原位，同时其他项在选择时也都错开了该项，那么该项则有可能最后保持原位
- 尤其是第一项，如果选择了和自己交换，那么后续其他项是不可能选择到和它交换，则第一项最后保持原位
- 极端情况，每项都和自己交换，最后数组未改变

``` javascript
let shuffle = () =>{
    const arr = [0, 1, 2, 3, 4];
    const length = arr.length;
    const lastIndex = length - 1;
    //i小于lastIndex，即从0开始到倒二个数，因为是跟自己或者剩余的数组项换，而最后一个元素后面没有其他数组项可以交换，跟自己交换又没有意义
    for (let i = 0; i < lastIndex; i++) {
        //random的取值区间为[i,lastIndex]
        //Math.floor(Math.random() * (m- n + 1)) + n;
        //Math.random() * (m- n + 1)的取值为[0,m-n+1)，再加n就是[n,m+1)，向下取整就是[n,m],套取公式就获得下面的式子
        const random = Math.floor(Math.random() * (lastIndex - i + 1)) + i;
        console.log(i, random);
        let temp = arr[random];
        arr[random] = arr[i];
        arr[i] = temp;
        //[arr[random], arr[i]]=[arr[i], arr[random]] 
        //交换2个数组项的变量解构赋值写法 https://es6.ruanyifeng.com/#docs/destructuring#%E7%94%A8%E9%80%94
        console.log(arr);
    }
};
```

### 常规情况

![](from_front_to_back-0.png)

### 第一项保持原位

![](from_front_to_back-1.png)

![](from_front_to_back_test.png)

## 从前往后，不存在保持原位（变体）

- 非完全乱序
- random的下限改为i+1
- 因为是跟剩余的数组项换，后面的项被交换到前面，就没有机会再被交换回来

``` javascript
let shuffle = () =>{
    const arr = [0, 1, 2, 3, 4];
    const length = arr.length;
    const lastIndex = length - 1;
    //i小于lastIndex，即从0开始到倒二个数，因为是跟剩余的数组项换，而最后一个元素后面没有其他数组项可以交换
    for (let i = 0; i < lastIndex; i++) {
        //random的取值区间为[i+1,lastIndex]
        //Math.floor(Math.random() * (m- n + 1)) + n;
        //Math.random() * (m- n + 1)的取值为[0,m-n+1)，再加n就是[n,m+1)，向下取整就是[n,m],套取公式就获得下面的式子
        const random = Math.floor(Math.random() * (lastIndex - (i + 1) + 1)) + (i + 1);
        console.log(i, random);
        let temp = arr[random];
        arr[random] = arr[i];
        arr[i] = temp;
        //[arr[random], arr[i]]=[arr[i], arr[random]] 
        //交换2个数组项的变量解构赋值写法 https://es6.ruanyifeng.com/#docs/destructuring#%E7%94%A8%E9%80%94
        console.log(arr);
    }
};
```

![](from_front_to_back_variant.png)

![](from_front_to_back_variant_test.png)

## 从后往前，可能保持原位

- 与上述从前往后可能保持原位的原理一致

``` javascript
let shuffle = () =>{
    const arr = [0, 1, 2, 3, 4];
    let i = arr.length;
    //可以改为while(i>1)，因为i=1时是轮到第一项，random一定会为0，自己跟自己交换没有意义
    while (i) {
        let random = Math.floor(Math.random() * i--);
        //相当于
        //let random = Math.floor(Math.random() * i);
        //i--;
        console.log(i, random);
        let temp = arr[i];
        arr[i] = arr[random];
        arr[random] = temp;
        //[arr[random], arr[i]]=[arr[i], arr[random]] 
        //交换2个数组项的变量解构赋值写法 https://es6.ruanyifeng.com/#docs/destructuring#%E7%94%A8%E9%80%9
        console.log(arr)
    }
}
```

![](from_back_to_front_test.png)

## 从后往前，不存在保持原位（变体）

- 非完全乱序
- 与上述从前往后不存在保持原位的原理一致

``` javascript
let shuffle = () =>{
    const arr = [0, 1, 2, 3, 4];
    let i = arr.length;
    //可以改为while(i>1)，因为i=1时是轮到第一项，random一定会为0，自己跟自己交换没有意义
    while (i) {
        let random = Math.floor(Math.random() * (i - 1));
        i--;
        console.log(i, random);
        let temp = arr[i];
        arr[i] = arr[random];
        arr[random] = temp;
        //[arr[random], arr[i]] = [arr[i], arr[random]] 
        //交换2个数组项的变量解构赋值写法 https://es6.ruanyifeng.com/#docs/destructuring#%E7%94%A8%E9%80%9
        console.log(arr);
    }
}
```

![](from_back_to_front_variant_test.png)

# 扩展

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： troy351
原文标题： 如何将一个 JavaScript 数组打乱顺序？
链接： https://www.zhihu.com/question/68330851/answer/266506621
来源： 知乎
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

``` javascript
const arr = [0, 1, 2, 3, 4];
for (let i = 1; i < arr.length; i++) {
    const random = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[random]] = [arr[random], arr[i]];
}
```

思路就是开头那张图的思路，但却是从前往后的顺序。上述的实现都是从哪边开始，则第一轮"洗牌"后，那边第一位的"牌"就已经确定下来了，就像是单独拿出来了，而这种实现则是"洗"完了下一轮继续参与"洗牌"。

我的理解是，从可能性上来看，该实例的可能性数也是5的阶乘，就是有点独特。

## sort

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： Lucas HC
原文标题： 如何将一个 JavaScript 数组打乱顺序？
链接： https://www.zhihu.com/question/68330851/answer/266506621
来源： 知乎
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

同时，很多答案提到了：

``` javascript
[12,4,16,3].sort(function() {
    return .5 - Math.random();
});
```

这样使用 `sort` 的方法。某些场景下，这样的方法可以使用。但是这不是真正意义上的完全乱序，一些需求中（比如抽奖）这样的写法会出大问题。

**为什么借助 sort 方法不是真正意义上的完全乱序**？

先证明不完全性。为此实现一个脚本，我对

``` javascript
var letters = ['A','B','C','D','E','F','G','H','I','J'];
```

 `letters` 这样一个数组使用 `array.sort` 方法进行了 10000 次乱序处理，并把乱序的每一次结果可视化输出。每个元素（ABCD...）出现的位置次数进行记录：

![](element_position_record.jpg)

具体脚本实现：[HOUCe/shuffle-array](https://github.com/HOUCe/shuffle-array)

不管点击按钮几次，你都会发现整体乱序之后的结果**绝对不是"完全随机"**。

比如 A 元素大概率出现在数组的头部，J 元素大概率出现在数组的尾部，**所有元素大概率停留在自己初始位置**。

究其原因，在[Chrome v8](https://github.com/v8/v8/blob/master/src/js/array.js)引擎源码中，可以清晰看到

>v8 在处理 `sort` 方法时，使用了插入排序和快排两种方案。当目标数组长度小于10时，使用插入排序；反之，使用快排。

其实不管用什么排序方法，大多数排序算法的时间复杂度介于 O(n) 到 O(n2) 之间，元素之间的比较次数通常情况下要远小于 n(n-1)/2，也就意味着有一些元素之间根本就没机会相比较（也就没有了随机交换的可能），这些 sort 随机排序的算法自然也不能真正随机。

通俗的说，其实我们使用 `array.sort` 进行乱序，理想的方案或者说纯乱序的方案是：数组中每两个元素都要进行比较，这个比较有 50% 的交换位置概率。如此一来，总共比较次数一定为 n(n-1)。

而在 `sort` 排序算法中，大多数情况都不会满足这样的条件。因而当然不是完全随机的结果了。

## lodash 库 _.shuffle

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： Lucas HC
原文标题： 如何将一个 JavaScript 数组打乱顺序？
链接： https://www.zhihu.com/question/68330851/answer/266506621
来源： 知乎
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

这也是正解，事实上翻开 lodash 源码相关部分，**这个方法正是采用了 Fisher–Yates shuffle 洗牌算法**。感兴趣的同学可以进行参阅。