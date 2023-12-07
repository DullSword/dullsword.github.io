---
title: Unity 选择具有不同概率的项
comments: true
toc: true
excerpt: '有时需要随机选择项，但有些项比其他项被选中的几率更高。'
date: 2020-08-31 18:10:47
categories: Unity
tags:
sticky: 0
recommend: 0
---
# 思路

假设权重分别为3，2，5

![](interval-0.png)

随机一个**0-10**之间的数  

``` javascript
Math.random()*10    // [0，10)  区间开闭看实际情况
```

数字落在 **[0,3)**，**[3,5)**，**[5,10)** 哪个区间就对应选择谁

判断落在哪个区间可以通过依次减去区间长度后是否有余

例如随机数为7.856：

- 7.856 - 3 = 4.856  说明属于后面的区间则继续
- 4.856 - 2 = 2.856  说明依旧属于后面的区间
- 2.856 - 5 < 0  说明属于该区间

例如随机数为2.856：

- 2.856 - 3 < 0  说明属于该区间

# 文档

查看文档时发现文档也有提到这个，[Adding Random Gameplay Elements](https://docs.unity3d.com/2019.3/Documentation/Manual/RandomNumbers.html)

## 选择具有不同概率的项

有时需要随机选择项，但有些项比其他项被选中的几率更高。例如，NPC 在遇到玩家时可能会以几种不同的方式做出反应：

- 友好问候的几率为 50%
- 逃跑的几率为 25%
- 立即攻击的几率为 20%
- 提供金钱作为礼物的几率为 5%
可将这些不同的结果可视化为一张纸条，该纸条分成几个部分，每个部分占据纸条总长度的一个比例。占据的比例等于选择结果的概率。选择行为相当于沿着纸条的长度选择一个随机点（例如通过投掷飞镖），然后查看该点处于哪个部分。

![](interval-1.png)

在脚本中，纸条实际上是一个浮点数组，其中的浮点数按顺序包含项的不同概率。随机点是通过将 `Random.value` 乘以数组中所有浮点数的总和得到的（这些数值不需要加起来等于 1；重点是不同值的相对大小）。要找到该点"位于"哪个数组元素，首先要检查它是否小于第一个元素中的值。如果是，则第一个元素便是选中的元素。否则，从该点值中减去第一个元素的值，然后将其与第二个元素进行比较，依此类推，直到找到正确的元素。在代码中表示为以下所示的内容：

<div class="tabs is-boxed"><ul>
<li class="is-active"><a onclick="onTabClick(event)">
<span class="icon is-small"><i class="far fa-file-alt" aria-hidden="true"></i></span>
<span>javascript</span>
</a></li>
<li><a onclick="onTabClick(event)">
<span class="icon is-small"><i class="far fa-file-alt" aria-hidden="true"></i></span>
<span>csharp</span>
</a></li>
</ul></div>

{% raw %}<div id="javascript" class="tab-content" style="display: block;">{% endraw %}
``` javascript
function Choose(probs: float[]) {
    var total = 0;
    
    for (elem in probs) {
        total += elem;
    }
    
    var randomPoint = Random.value * total;
    
    for (i = 0; i < probs.Length; i++) {
        if (randomPoint < probs[i])
            return i;
        else
            randomPoint -= probs[i];
    }
    
    return probs.Length - 1;
}
```
{% raw %}</div>{% endraw %}

{% raw %}<div id="csharp" class="tab-content">{% endraw %}
``` csharp
float Choose (float[] probs) {

    float total = 0;

    foreach (float elem in probs) {
        total += elem;
    }

    float randomPoint = Random.value * total;

    for (int i= 0; i < probs.Length; i++) {
        if (randomPoint < probs[i]) {
            return i;
        }
        else {
            randomPoint -= probs[i];
        }
    }
    return probs.Length - 1;
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

请注意，最后的 `return` 语句是必要的，因为 `Random.value` 可以返回 `1` 的结果。在这种情况下，搜索将无法在任何地方找到随机点。将以下行

``` c
    if (randomPoint < probs[i])
```

…更改为"小于或等于"测试将避免额外的 `return` 语句，但也会允许偶尔选择某个项，即使其概率为零也是如此。