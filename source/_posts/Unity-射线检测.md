---
title: Unity 射线检测
comments: true
toc: true
excerpt: '本文简要分析了Unity中各类 射线检测 的基本原理及用法，及不同检测手段的性能对比。'
date: 2020-05-13 10:27:56
categories: Unity
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： SouthBegonia
原文标题： Unity - 射线检测
链接： https://www.cnblogs.com/SouthBegonia/p/11732340.html
来源： 博客园
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

本文简要分析了Unity中各类 **射线检测** 的基本原理及用法，及不同检测手段的性能对比。内容包括：

- Ray 射线
- RaycastHit 光线投射碰撞信息
- Raycast 光线投射
- BoxCast/SphereCast/CapsuleCast 体投射
- OverlapBox/OverlapSphere/OverlapCapsule 相交体
- OverlapBoxNonAlloc/OverlapSphereNonAlloc/OverlapCapsuleNonAlloc 无GC相交体
- 常用射线Debug方法
- GC消耗分析

**项目地址**：[Raycast - SouthBegonia](https://github.com/SouthBegonia/UnityWorld/tree/master/Raycast)

![raycast - 0](raycast-0.gif)

![raycast - 1](raycast-1.gif)

# Ray 射线

- 含义：官方解释为一条无穷的线，开始于origin点，朝向direction方向（但是，根据项目验证来看其默认长度为单位向量，只有对direction进行乘以倍率，才可实现延长射线，而非无穷）
- 用法：
    - `Ray ray = new Ray(transform.position,transform.forward)`：从物体中心创建一条指向前方的射线ray
    - `Ray camerRay = Camera.main.ScreenPointToRay(Input.mousePosition)`：产生一条从摄像机产生、经过屏幕上光标的射线。当相机为perspective模式，射线在相机梯形视野内发散；若为orthoGraphic，则为垂直与相机面的直线段（见上图）

# RaycastHit 光线投射碰撞信息

- 含义：取得从Raycast函数中得到的碰撞信息（注意不是collider哈，是包含collider信息）
- 关键变量：point、collider、rigidbody、transform

# 检测方法 - 线型检测

## Physics.Raycast 光线投射

- 功能：在已有一条射线（也可无）的基础上，使用射线（新建射线）进行一定距离内的定向检测。可修改射线长度，限制其检测的Layer层，并且可以得到射线检测到的碰撞信息。但仅能检测到**第一个被射线碰撞的物体**，后面的物体无法被检测到
- 用法：
    - `Raycast(transform.position, Vector.forward, distance, LayerMask.GetMask("Enemy"))`： 从物体中心点起，朝向Vector3.forward方向发射一条射线，该射线长度为distance，射线可检测到的层为Enemy层，返回bool类型
    - `Raycast(transform.position, Vector.forward, distance, out RaycastHitInformation ,LayerMask.GetMask("Enemy")）`：从物体中心点起，朝向Vector3.forward方向发射一条射线，该射线长度为distance，将碰撞信息反馈到RaycastHitInformation上，射线可检测到的层为Enemy层，返回bool类型
    - `Raycast (MyRay, distance, LayerMash.GetMask("Enemy"))`：从已有的射线MyRay出发，长度延伸至distance，射线可检测到的层为Enemy层，返回bool类型
    - `Raycast (MyRay, out RaycastHitInformation, distance, LayerMask.GetMask("Enemy"))`
- 适用场合：配合相机坐标转换实现各类交互

---

## Physics.RaycastAll 所有光线投射

- 功能：机理用法大致同Raycast，区别在于可检测射线路径上的**所有物体**，**返回RaycastHit\[\]** 。其他带All后缀的方法也同理
- 用法：
    - `RaycastHit[] hits = RaycastAll(Vector3.zero, Vector.forward, distance, LayerMask.GetMask("Enemy"))`
    - `RaycastHit[] hits = RaycastAll(MyRay, distance, LayerMash.GetMask("Enemy"))`
- 适用场合：穿透性检测

---

## Physics.Linecast 线段投射

- 功能：建立**某两点之间**的射线进行检测，返回bool类型
- 用法：
    - `Linecast(startPos, endPos, LayerMask.GetMask("Enemy"))`
    - `Linecast(startPos, endPos, out RaycastHit, LayerMask.GetMask("Enemy"))`
- 适用场合：特定地点局部距离射线检测

# 检测方法 - 体型检测

## Physics.XXXCast 体投射

1. **BoxCast 立方体投射**：
    - 功能：检测范围是正立方，返回bool。但是该投射用法需要万分小心，见下
    - 用法：`BoxCast(originPos, halfExtents, direction, out RaycastHit, distance, LayerMask.GetMask("Enemy"))`：含义是在originPos点创建半径halfBoxLength的立方体（Vector3型，代表为正立方体在三个方向上的大小，一般用localScale/2）；以朝向direction方向的平面为起始面（另一面舍弃），移动distance距离，期间经过的区域即为检测区域。（见下补充分析）
    - 适用场合：检测目的地是否可抵达，从而判断可移动性

2. **SphereCast 球体投射**：
    - 功能：扩展检测范围为球形，返回bool类型。
    - 用法：
        - `SphereCast(originPos, radius, direction, out RaycastHit, distance, LayerMask.GetMask("Enemy"))`：含义是在originPos点创建半径为radius的球体；以朝向direction方向的球面为起始面（另一面舍弃），移动distance距离，期间半球面经过的区域即为检测区域。那么originPos到originPos+radius内的半球区域呢？答案是舍弃，用官方的话来说，**是边界而不是包围体** 。(立体结构：以左右球球心为轴线，建立半径为radius、高为distance的圆柱体，左球挖去右半体积，右球添加右半体积）
        - `SphereCast (Ray, radius, out RaycastHit, distance, LayerMask.GetMask("Enemy"))`

3. **CapsuleCast 胶囊体投射**：
    - 功能：检测范围是胶囊体，返回bool
    - 用法：`Physics.CapsuleCast(pos1, pos2, radius, direction, out RaycastHit, maxDistance, LayerMask.GetMask("Anchor"))`：机理和SphereCast类似，在pos1、pos2两点创建半径为0.5f的球体，以此作为胶囊体模型两端；以朝向direction方向的半胶囊体面为起始面，移动maxDistance距离，期间该面经过的区域即为检测区域。（注：maxDistance和上面的distance必须非0否则无用）
    - 项目示例：见下图，pos1、pos2均为胶囊体两球坐标，视角右侧为forward，移动距离为0.1f，从实验结果我们不仅可以直观感受maxDistance的含义，更印证上面所说的 **是边界不是包围体** ，代码：`Physics.CapsuleCast(pos1, pos2, 0.5f, Vector3.forward, out RaycastHit, 0.1f, LayerMask.GetMask("Anchor"))`

4. **XXXCastAll 穿透投射**：
    - 功能：上述三种投射都只返回bool，只能检测单个物体，但是All方法的可检测射线上的所有物体，返回 **RaycastHit\[\]**，**产生GC极多**

![Physics.XXXCastAll](Physics.XXXCastAll.gif)

---

## Physics.OverlapXXX 相交体

1. **OverlapBox 相交盒**：
    - 功能：检测与正立方体接触、重叠、或者处于其内的所有collider
    - 用法：`Collider[] hits = OverlapBox(Pos, halfExtents, Quaternion.identity, LayerMask.GetMask("Enemy"))`：以Pos点为中心创建三维半径halfExtents的正立方体，不对其进行旋转，检测层为Enemy
    - 适用场合：检测挂载物体范围内是否存在碰撞，常用方法

2. **OverlapSphere 相交球**：
    - 功能：检测与球体接触、重叠、或者处于其内的所有collider，即**包围体**。但注意，**自身collider也会被检测**到（下列Overlap方法都是）
    - 用法：`Collider[] hits = Physics.OverlapSphere(Pos, radius, LayerMask.GetMask("Enemy"))`：以Pos为原点，创建半径为radius的球形，检测区域为整个球形包围体（实心），检测Enemy层上的物体，返回所有碰撞物体的collider而不是RaycastHit（注意：存在于球内部的物体也会被检测到）

3. **OverlapCapsule 相交胶囊体**：
    - 功能：检测与胶囊体接触、重叠、或者处于其内的所有collider
    - 用法：`Collider[] hits = OverlapCapsule(pos1, pos2, radius, LayerMask.GetMask("Enemy"))`：在pos1、pos2两点创建半径为radius的球体，加上中间部分组成胶囊体，检测Enemy层

---

## Physics.OverlapXXXNonAlloc 无GC相交体

1. **OverlapBoxNonAlloc 无GC相交盒**：
    - 功能：实现OverlapBox的所有功能，但是另**传递进colliders\[\]** ，**返回相交物体数量**，从而**杜绝GC的产生**
    - 用法：`CollAmount = Physics.OverlapBoxNonAlloc(Pos, halfExtents, colliders, Quaternion.identity, LayerMask.GetMask("Enemy"))`

2. **OverlapSphereNonAlloc 无GC相交球**：
    - 功能：参考上面
    - 用法：`CollAmount = OverlapSphereNonAlloc(Pos, radius, colliders, LayerMask.GetMask("Enemy"))`

3. **OverlapCapsuleNonAlloc 无GC相交胶囊体**：
    - 功能：参考上面
    - 用法：`CollAmount = OverlapCapsuleNonAlloc(pos1, pos2, radius, colliders, LayerMask.GetMask("Enemy"))`

---

## Physics.CheckXXX 检验体

1. **CheckBox 检验盒**：
    - 功能：创建检测盒，检测是否被碰撞。较比与上面的检测方法，该类方法特点在于**检验是否发生了碰撞**，而**不是取得碰撞体信息**，效率最高。注此方法同样也会检验自身collider
    - 用法：`IsOverlapAnyCollider = Physics.CheckBox(transform.position, transform.localScale / 2, Quaternion.identity, LayerMask.GetMask("Enemy"))`：在物体中心创建检验盒，一定大小，不旋转，检测Enemy层，若有检测到碰撞则返回True

2. **CheckSphere 检验球**：
    - 功能：参考上面
    - 用法：`IsOverlapAnyCollider = Physics.CheckSphere(transform.position, radius, LayerMask.GetMask("Enemy"))`

3. **CheckCapsule 检验胶囊体**：
    - 功能：参考上面
    - 用法：`IsOverlapAnyCollider = Physics.CheckCapsule(pos1, pos2，radius, LayerMask.GetMask("Enemy"))`

---

## Physics.IgnoreCollision 忽略碰撞

- 功能：屏蔽两个collider的碰撞，第三个参数为bool
- 用法： `IgnoreCollision (collider1, collider2, ignore)`

# DEBUG手段

- **绘制线段**：
    - `DrawLine(startPos, endPos, color)`：绘制一条从startPos到endPos点、颜色为color的线段
- **绘制射线**：
    - `DrawRay(startPos, direction, color)`：绘制一条从startPos出发，指向direction的、颜色color的射线（默认长度为单位向量，再乘以倍率即可边长；在下一次绘制才会覆盖上一次的射线）
    - `Debug.DrawRay(startPos , direction, color, duration)` ：同理绘制一定方向射线，但射线持续时间为duration ：
- **Gimos.DrawXXX方法**：
    - `void OnDrawGizmos() { Gizmos.DrawCube(transform.position, transform.localScale );}`

# GC开销问题

从上面对几种检测方法的分析及对比其**返回值**不难发现，**不同方法产生GC情况相差甚远**，因此在工程项目上应该慎重使用。此处引用网友 [HONT](https://www.cnblogs.com/hont/p/6180822.html)的测试作为GC情况参考：

- 同方法下不同模型GC开销：**Box < Sphere < Capsule**
- 同模型下不同方法GC开销：**CheckXXX < OverlapXXX < XXXCast**

# 参考

- [Physics.Raycast - UnityDocumentation](https://docs.unity3d.com/2018.3/Documentation/ScriptReference/Physics.Raycast.html)
- [Unity常用射线检测方法 - 昵称好难写](https://blog.csdn.net/qq_36274965/article/details/79456449)
- [Unity的学习笔记(射线检测) - 小鸟游](https://www.cnblogs.com/takanashi/p/11028786.html)
- [Unity3DAPI射线检测 - 码迷](http://www.mamicode.com/info-detail-2514998.html)
- [Unity中各类物理投射性能横向比较 - HONT](https://www.cnblogs.com/hont/p/6180822.html)