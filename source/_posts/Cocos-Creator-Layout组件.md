---
title: Cocos Creator Layout组件
comments: true
toc: false
excerpt: ' '
date: 2018-09-25 21:44:20
categories: Cocos Creator
tags: Cocos Creator
sticky: 0
recommend: 0
---
**ResizeMode**为**NONE**的话，**动态加载**子物体的话，容器大小是**不会**改变的。除此之外，使用**网格**布局时，**动态加载**子物体，如果是要同时动态改变容器的高度（常见），那么**ResizeMode**需要设置为**CONTAINER**，让容器被子物体“撑开”，但这样就需要注意**子物体的大小以及容器Padding、SpacingX、SpacingY**的设置，**CHILDREN**实现不了，**CHILDREN**适用于容器大小已经定下来的。

下面是Cocos Creator官方对Layout组件的详细说明，有一些注意点。

# 详细说明
添加 Layout 组件之后，默认的布局类型是 NONE，它表示容器不会修改子物体的大小和位置，**当用户手动摆放子物体时，容器会以能够容纳所有子物体的最小矩形区域作为自身的大小**。

通过修改 属性检查器 里面的Type可以切换布局容器的类型，可以切换成水平，垂直或者网格布局。

另外，所有的容器均支持 ResizeMode（NONE 容器只支持 NONE 和 CONTAINER）。

- 当 ResizeMode 设置为 NONE 时，子物体和容器的大小变化互不影响。

- 设置为 CHILDREN 则子物体大小会随着容器的大小而变化。

- 设置为 CONTAINER 则容器的大小会随着子物体的大小变化。

## 注意

- Layout **不会**考虑子节点的**缩放和旋转**。
- Layout 设置后的结果需要到**下一帧**才会更新，除非你设置完以后**手动调用 '`updateLayout`' API**。