---
title: UE4 射线检测
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2021-08-27 11:15:40
categories:
  - Unreal
  - Unreal4
tags:
---

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
[ Multi ] + [ Line | Sphere | Capsule | Box ] + Trace + [ ForObjects | ByChannel | ByProfile ]
{% raw %}</div></article>{% endraw %}

1. 第一部分指明返回追踪命中的**首个**物体还是**多个**物体
    - 如果不加Multi，就是返回首个阻挡命中
    - 加了Multi，就是返回遭遇的所有命中，包含首个阻挡命中，**需要注意的是，如果被阻挡了，后续射线就失效了，所以想要返回多个命中的话，前面的物体对应的地方一定要设置为Overlap**
2. 第二部分指明射线的**形状**
3. 第三部分指明追踪的**方式**

前两个部分还是比较好理解的，所以着重研究下第三部分。

{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
射线未失效部分默认显示为红色，失效部分默认显示为绿色。如果碰撞后射线失效，则碰撞点默认显示为红色方块，如果碰撞后射线未失效，则碰撞点默认显示为绿色方块。
{% raw %}</div></article>{% endraw %}

# ForObjects

## 结论

1. 物体碰撞类型要包含查询（纯查询和启用碰撞✔️无碰撞和纯物理❌）
2. 物体碰撞设置的对象类型和要检测的对象类型一致
3. 与物体碰撞响应的设置无关

## 测试过程

将 `LineTraceForObjects` 的ObjectTypes设置为WorldStatic（静态场景）。

![](LineTraceForObjects_BP.PNG)


将面前CubeMesh的碰撞类型设置为"无碰撞"，对象类型设置为"WorldStatic"。

可以发现射线穿过CubeMesh，没有产生碰撞点，碰撞点在后方的Ramp_StaticMesh上，也就是忽略了CubeMesh。

![](LineTraceForObjects_NoCollision.PNG)

将面前CubeMesh的碰撞类型设置为"纯物理（不查询碰撞）"，对象类型设置为"WorldStatic"。

和碰撞类型为"无碰撞"的结果是一样的。

![](LineTraceForObjects_OnlyPhysics.PNG)

将面前CubeMesh的碰撞类型设置为"纯查询（无物理碰撞）"，对象类型设置为"WorldStatic"，碰撞响应均设置为"忽略"。

在CubeMesh上产生了碰撞点，而后续射线呈绿色失效。

![](LineTraceForObjects_OnlyQuery.PNG)

**由此可得，碰撞类型要包含查询，与碰撞响应的设置无关。**

理所当然，物体碰撞设置的对象类型和要检测的对象类型一致。

![](LineTraceForObjects_DifferentObjects.PNG)

# ByChannel

## 结论

1. 物体碰撞类型要包含查询（纯查询和启用碰撞✔️无碰撞和纯物理❌）
2. 物体检测响应对应的通道需要设置为"阻挡"或"重叠"，需要注意，在不加Multi前缀的情况下"重叠"和"忽略"的表现是一样的
3. 与物体对象类型的设置无关

## 测试过程

将 `LineTraceByChannel` 的TraceChannel设置为Visibility（可见性）。

![](LineTraceByChannel_BP.PNG)

各个碰撞类型的表现和在 `ForObjects` 下的表现一致。

![](LineTraceByChannel_NoCollision.PNG)

![](LineTraceByChannel_OnlyPhysics.PNG)

![](LineTraceByChannel_OnlyQuery.PNG)

将面前CubeMesh的对象类型设置为"Pawn"。

结果无变化，所以与对象类型无关。

![](LineTraceByChannel_DifferentObjects.PNG)

接下来看看将Visibility通道设置为"重叠"和"忽略"会有什么表现。

![](LineTraceByChannel_Overlap.PNG)

![](LineTraceByChannel_Ignore.PNG)

看起来两者一样，但这边有一个需要注意的点就是，由于用的是 `LineTraceByChannel` ，而不是 `MultiLineTraceByChannel` ，`LineTraceByChannel`只会返回首个阻挡命中，所以CubeMesh的Visibility通道为"重叠"并不会产生碰撞点。

# ByProfile

## 结论

1. 物体碰撞类型要包含查询（纯查询和启用碰撞✔️无碰撞和纯物理❌）
2. 射线的碰撞类型、对象类型、碰撞响应均为对应的profile预设下的设置（除了"NoCollision"，不清楚为什么，需要看源码）

## 测试过程

我们先找个Profile来看看，我挑了个IgnoreOnlyPawn。

可以看到除了"Pawn"和"Vehicle"进行了"忽略"，其他均设置为"阻挡"。

![](IgnoreOnlyPawn.PNG)

将 `LineTraceByChannel` 的ProfileName设置为IgnoreOnlyPawn。

![](LineTraceByProfile_BP.PNG)

这边就不赘述碰撞类型了，测试过了，和 `ForObjects` 、 `ByChannel` 的情况一样，物体碰撞类型要包含查询。

将面前CubeMesh的对象类型设置为"Pawn"和"Vehicle",射线均忽略了CubeMesh。

![](LineTraceByProfile_ObjectTypePawn.PNG)

![](LineTraceByProfile_ObjectTypeVehicle.PNG)

那CubeMesh对象类型为"Pawn"和"Vehicle"以外的类型时，都会响应吗？

将CubeMesh物体响应的WorldDynamic设置为"重叠"和"忽略"时会发现射线不再在CubeMesh上产生碰撞点了。

**之所以是WorldDynamic是因为我们选择的"IgnoreOnlyPawn"，"IgnoreOnlyPawn"下的对象类型为"WorldDynamic"。**

![](LineTraceByProfile_WorldDynamicResponseOverlap.PNG)

![](LineTraceByProfile_WorldDynamicResponseIgnore.PNG)

当然"重叠"的情况看起来和"忽略"的情况一样，还是因为用的是 `LineTraceByChannel`，而不是 `MultiLineTraceByChannel` 。

# 参考

- [[玩转UE4动画系统＞基础篇] 之 什么是射线检测 - 开发游戏的老王](https://zhuanlan.zhihu.com/p/369560738)