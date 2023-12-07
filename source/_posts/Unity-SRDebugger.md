---
title: Unity SRDebugger
comments: true
toc: true
excerpt: '本章要来介绍这款超好用的插件『SRDebugger』。插件支援Real-time Console，运作游戏时如果出现Bug，或是程式撰写的Debug.Log，都会出现在Console画面。另外，还有Options Tab的功能，类似游戏作弊用控制台，可以撰写类似『无限血量』、『自动跳关』等方便测试的功能，亦能输出一些重要的讯息。'
date: 2020-11-17 15:29:30
categories: Unity
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： ANONEKO
原文标题： SRDebugger
链接： https://3dactionrpg.blogspot.com/2018/05/6-24-srdebugger.html
来源： blogger
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

本章要来介绍这款超好用的插件『SRDebugger』。插件支援Real-time Console，运作游戏时如果出现Bug，或是程式撰写的Debug.Log，都会出现在Console画面。另外，还有Options Tab的功能，类似游戏作弊用控制台，可以撰写类似『无限血量』、『自动跳关』等方便测试的功能，亦能输出一些重要的讯息。

[SRDebugger - Console & Tools On-Device](https://assetstore.unity.com/packages/tools/gui/srdebugger-console-tools-on-device-27688)

![](SRDebugger.jpg)

首先，当你安装好SRDebugger插件后，他预设会自动启用，你不需要额外撰写任何的程式码。如果你想用程式码的方式决定是否开启SRDebugger也行。请在`Window/SRDebugger/Settings Window`开启设定画面。

![](settings_window.png)

在`Loading`选项中预设是`Automatic`，你可以使用`Prefab`或用程式码`SRDebug.Init()`进行开启。这部分的操作大家自己试看看就好，本章的重点会放在撰写Options Tab上。

![](options_tab.png)

进入游戏后，SRDebugger的开启方式，连续点击左上角的区域3次，如下图由红色圈框起来的地方。

![](open_method.png)

开启后的画面如下，会看见一些System相关的资讯。

![](system.png)

如果程式有`Exception`，或是有使用`Debug.Log`的地方，都会显示在这个Console画面。

![](console.png)

Profiler的部分则可以看见显示`FPS`、`Memory`的使用情形。

![](profiler.png)

再来是Options的部分，由于我们现在还没有撰写任何程式码，所以会显示如下资讯『`No options found in SROptions.`』。

![](options.png)

好的，接下来开始撰写程式码吧！

**Methods**

首先我想增加『+100000 HP』、『+100000 MP』的按钮，也会需要一个『取消HP/MP作弊』的按钮。在SROptions中撰写按钮的方法，是**宣告一个无输入参数、回传值为`void`的`public`方法**，SROptions会自动转化为按钮。

所以，接着撰写`CheatHealth`、`CheatEnergy`、`RevertPlayerHealthAndEnergy`等方法，这边我是透过一个自定义`Game`物件统一管理与其他系统的交互事件。类似中介者模式(`Mediator`)的简化版做法。

![](CheatHealth_etc.png)

写好以后执行游戏测试，打开介面就会出现如下图的三颗按钮啦！按下去就可以作弊了。

![](buttons.png)

**Supported Type**

除了按钮以外，我们也可以控制变数，SROptions支援`Boolean`, `Enum`, `int`, `uint`, `short`, `ushort`, `byte`, `sbyte`, `float`, `double`, `string`。比方说我想制作一个『金钱作弊』的功能，要在SROptions中调整金钱数量，控制作弊变数有两种方式：

1. **官方建议使用第一种方式**。如下图，宣告变数`CheatCoin`并设定`get/set`，再有需要的地方使用`SROptions.Current.CheatCoin`取得即可。

![](argument.png)

1. **官方不建议使用第二种方式**。于get/set中使用FindObjectOfType取得指定脚本及参数，直接进行设定。

![](FindObjectOfType.png)

不过依照前面的例子，我的作法是在设定完`cheatCoin`之后，仍旧由`Game`类别来处理金钱的修改事件。

![](argument-0.png)

接着来测试看看，我在Options介面将金钱设定成80000。

![](before_change.png)

接着打开背包介面后，发现金钱确实被更改成80000了。

![](after_change.png)

**OnPropertyChanged**

如果有些变数在游戏中会实时变化的呢？我们可以用`OnPropertyChanged`这个方法，让SRDebugger检查变数的新值。如果没有使用`OnPropertyChanged`的话，则SROptions属性的更新只会在每次打开选项时才会反映出来。通过这个方法，变数的任何更新都会立即显示出来。由于我的游戏目前没有这个需求，大家就自行参考范例试看看吧。

![](OnPropertyChanged.png)

**Attributes**

大家可能会觉得按钮的顺序不好，或是金钱作弊增加钮每次只能+1，又或者想要依照不同作弊类型来分类功能等等。以下跟大家介绍SROptions可以使用的`Attributes`吧。如下图，我们可以在`Method`或是`Properties`上面使用`Attributes`来达成一些功能。

![](attributes.png)

`Category`：分类名称
`DisplayName`：介面显示名称
`Sort`：功能排序
`NumberRange`：如为数字类型的变数，可设定数字范围
`Increment`：数字每一次增减的量

如果嫌这样分行撰写太占行数，可以使用逗号将每个`Attributes`隔开并撰写于同一行中，比方

``` csharp
[Category("Cheat Inventory"), DisplayName("金钱"), Sort(1), NumberRange(0, 1000000), Increment(10000)]
```

**Pinning**

如果觉得每次要作弊的时候，开启SRDebugger介面很麻烦的话，也可以将作弊按钮钉选到画面上唷。开启Options后，可以看见右上角有一个图钉按钮，按下后会提示你选择要留在画面上的功能，如下图，我选择了三个功能。

![](pinning.png)

关闭SRDebugger后，就能看见三颗按钮出现在右下角，我测试金钱作弊的功能后，确实可以正常使用唷！真是太方便了。

![](pinning_result.png)

如果还想看更多参数的使用方式，SRDebugger也有提供范例文件，位置在`StompyRobot/SRDebugger/Scripts/SROptions.Test.cs`

最后，如果大家想要了解更多使用SRDebugger的说明的话，就请参考官方文件吧：
[https://www.stompyrobot.uk/tools/srdebugger/documentation/](https://www.stompyrobot.uk/tools/srdebugger/documentation/)