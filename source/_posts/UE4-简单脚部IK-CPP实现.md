---
title: UE4 简单脚部IK C++实现
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 2
date: 2021-12-13 17:32:59
categories:
  - Unreal
  - Unreal4
tags:
- IK
---
如题，这只是很简单的脚步IK实现，动画部分使用的是蓝图。

实际项目可以使用 [Power IK](https://www.unrealengine.com/marketplace/en-US/product/power-ik) 或UE5 [基于全身位置的IK](https://docs.unrealengine.com/5.0/zh-CN/AnimationFeatures/PBIK/)。

![](IK.gif)

# 项目地址

[UE4-SimpleFootIK](https://github.com/DullSword/UE4-SimpleFootIK)

# 思路
就是让离地远的脚着地，另一只脚抬起相应的距离。

如下图所示，**1**是IK前的左右脚状态，**2**是把骨骼网格体整体往下移动至离地远的脚着地时左右脚的状态，**3**是离地近的脚抬起后的状态，**4**是将前3个过程结合在一起。

![](idea.png)

所以最主要的就是求骨骼网格体下移的距离和离地近的脚需要抬起的距离，即图所示的绿线和红线。

![](lift_and_move_down.png)

我们可以如下图所示从双脚脚底往下发出长度为 `IKTraceDistance` 的射线，通过 `HitDistance` 可以得到哪一只脚是离地远的脚。因为离地远的脚不做IK处理，姿势不变，所以所得到的 `HitDistance` 与骨骼网格体下移的距离（绿线）相等。离地近的脚则需要抬起双脚 `HitDistance` 差值的绝对值，而不是离地远的脚的 `HitDistance`。这点可以从图上看出，双脚 `HitDistance` 差值的绝对值就是台阶的高度。只是在一脚着地的情况下，也就是该脚 `HitDistance` 为0，离地远的脚的 `HitDistance`刚好等于双脚 `HitDistance` 差值的绝对值，这点要注意。

![](various_distance.png)

# 实现

## 射线长度
射线长度为半腿高，也就是胶囊体半高的一半，射线终点就是离地远的脚最多能下到的位置。

``` cpp
    //in constructor
    CapsuleHalfHeight = GetCapsuleComponent()->GetScaledCapsuleHalfHeight();
    IKTraceDistance = CapsuleHalfHeight/2;  //如果人物大小会变化，还得再乘以scale
```

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
为什么射线长度设置为半腿高？
{% raw %}</div></article>{% endraw %}

我的想法是让脚最多就只能抬半条腿的高度，高于这个程度的话，看起来就有点不合理了。比如下图：

![](lift_unreasonable.png)

![](lift_unreasonable_in_ue4.png)

## 射线检测
主要是调用`UKismetSystemLibrary::LineTraceSingleForObjects`。

### 准备工作
在骨骼的左右脚部分增加插槽。

![](foot_socket.png)

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
为什么不直接使用foot骨骼或者ball骨骼？
{% raw %}</div></article>{% endraw %}

也可以，使用现有的foot骨骼的话，到时候设置射线起点时记得加上偏移，或者使用现有的ball骨骼。使用插槽的话，比较直观，方便调整。

### 检测代码

```cpp
    FVector Start;
    FVector End;

    FVector SocketLocation = GetMesh()->GetSocketLocation(Socket);
    float X = SocketLocation.X;
    float Y = SocketLocation.Y;
    float FootZ = GetActorLocation().Z - CapsuleHalfHeight; //1️⃣ 
    Start.Set(X, Y, FootZ);
    End.Set(X, Y, FootZ - IKTraceDistance); //不能是-IKTraceDistance，因为接触面不一定是地面，即便是地面，其世界坐标Z为不一定为0

    //要跟踪的对象类型数组
    TArray<TEnumAsByte<EObjectTypeQuery>> ObjectTypes;
    ObjectTypes.Add(UEngineTypes::ConvertToObjectType(ECC_WorldStatic));
    //要忽略的对象类型数组
    TArray<AActor*> ActorsToIgnore;
    //碰撞结果
    FHitResult OutHit;

    UKismetSystemLibrary::LineTraceSingleForObjects(
        GetWorld(),
        Start,
        End,
        ObjectTypes,
        false,
        ActorsToIgnore,
        EDrawDebugTrace::ForOneFrame,   //EDrawDebugTrace::None 不显示射线
        OutHit,
        true
    );

    float HitDistance;
    if( OutHit.bBlockingHit )
    {
        HitDistance = OutHit.Distance;
    }
    else
    {
        //没有命中的话，说明把脚往下放半条腿的距离也够不到地，把HitDistance赋值为-1，用于跟HitDistance刚好等于0的情况区分开
        HitDistance = -1;
    }
```

#### 1️⃣  float FootZ = GetActorLocation().Z - CapsuleHalfHeight;
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
为什么不直接用SocketLocation.Z？
{% raw %}</div></article>{% endraw %}

`SocketLocation.Z` 的值会随着脚的移动发生改变，不单单IK时会被影响，人物网格体下移时也会被影响，甚至人物移动时都会影响到。所以最好找个相对于整个过程中位置不变的脚的位置，也可以理解为IK前的初始位置。

## 求得IKOffset

```cpp
    if( HitDistanceL < 0 || HitDistanceR < 0 )
    {
        //HitDistance有为负数的情况，说明至少一只脚离地高于半条腿，那么将IK效果取消掉
        IKOffsetLeftFoot = 0;
        IKOffsetRightFoot = 0;
    }
    else
    {
        //将离地远的脚放至地上，另一只脚抬高HitDistanceDifference的距离
        float HitDistanceDifference = FMath::Abs(HitDistanceL - HitDistanceR);
        if( HitDistanceL < HitDistanceR )
        {
            IKOffsetLeftFoot = HitDistanceDifference;
            IKOffsetRightFoot = 0;
        }
        else
        {
            IKOffsetLeftFoot = 0;
            IKOffsetRightFoot = HitDistanceDifference;
        }
    }
```

## 应用IKOffset
虚幻引擎在动画蓝图中提供了`Two Bone IK`节点，具体讲解可以参考：

- [IK设置 - 虚幻引擎4文档](https://docs.unrealengine.com/4.26/zh-CN/AnimatingObjects/SkeletalMeshAnimation/IKSetups/)
- [Two Bone IK - 虚幻引擎4文档](https://docs.unrealengine.com/4.26/zh-CN/AnimatingObjects/SkeletalMeshAnimation/NodeReference/SkeletalControls/TwoBoneIK/)
- [[UE4蓝图]虚幻4中完整实现脚部IK（一）- 架狙只打脚](https://zhuanlan.zhihu.com/p/84399021)
- [IK节点详解 - 知乎用户uwtE6F](https://zhuanlan.zhihu.com/p/41425611)

`Two Bone IK` 节点需要 `Effector Location`，当**执行器位置空间**选择**骨骼空间**时，执行器目标本地坐标系朝上方向的大小就是对应先前求得的 `IKOffset`。

![](two_bone_IK_setup-0.png)

因为我们求出的 `IKOffset` 是标量，不带方向，**执行器位置空间**又选择的是**骨骼空间**，所以得注意执行器目标的本地坐标系朝向。从下图可以看出右脚对应的 `Two Bone IK` 的执行器目标 `foot_r` 本地坐标系 `X` 的反方向对应世界坐标系的 `Z` 的正方向，所以处理时需要乘以-1，转换成我们需要的上方向。

![](foot_r_direction.png)

![](assignment_effector_location.png)

而 `Joint Target Location` 是为了确定3个骨骼所处在的平面，这个自己调的就行了。需要注意所填的值必须得是对应所选的**关节目标位置空间**以及**关节目标**。

![](two_bone_IK_setup-1.png)

到这一步，可以运行下看看。

![](IK_effect.png)

## 求得MeshOffset
脚部已经有了IK效果，接下来处理骨骼网格体下移的部分。之前分析过，离地远的脚的 `HitDistance` 与骨骼网格体下移的距离相等，所以只要加上这么一行代码就可以了。

``` cpp
    MeshOffsetZ = FMath::Max(HitDistanceL, HitDistanceR);
```

## 应用MeshOffset
虚幻引擎在动画蓝图中提供了 `Transform Bone` 节点，`Transform Bone` 节点将处理骨骼的变换——即为平移、旋转或缩放。

![](transform_bone.png)

一样要注意方向的问题。

![](pelvis.png)

![](assignment_mesh_offset.png)

## 阶段效果

![](IK.png)

效果基本已经实现，但变化过程是瞬间的，上下楼梯很鬼畜，可以使用插值让变化过渡得更自然。

``` cpp
    float AIK_PracticeCharacter::FInterp(float CurrentValue, float TargetValue)
    {
        return FMath::FInterpTo(CurrentValue, TargetValue, GetWorld()->GetDeltaSeconds(), IKInterpSpeed);
    }
```

``` cpp
    if( HitDistanceL < 0 || HitDistanceR < 0 )
    {
        IKOffsetLeftFoot = FInterp(IKOffsetLeftFoot, 0);
        IKOffsetRightFoot = FInterp(IKOffsetRightFoot, 0);
    }
    else
    {
        float HitDistanceDifference = FMath::Abs(HitDistanceL - HitDistanceR);
        if( HitDistanceL < HitDistanceR )
        {
            IKOffsetLeftFoot = FInterp(IKOffsetLeftFoot, HitDistanceDifference);
            IKOffsetRightFoot = FInterp(IKOffsetRightFoot, 0);
        }
        else
        {
            IKOffsetLeftFoot = FInterp(IKOffsetLeftFoot, 0);
            IKOffsetRightFoot = FInterp(IKOffsetRightFoot, HitDistanceDifference);
        }
    }

    float _MeshOffsetZ = FMath::Max(HitDistanceL, HitDistanceR);
    MeshOffsetZ = FInterp(MeshOffsetZ, _MeshOffsetZ);
```

## 调整胶囊体
骨骼网格体下移后，胶囊体就不再贴合了，如下图所示。

![](show_collision.png)

我们需要对应调整胶囊体的半高。如下图所示，绿线就是胶囊体要调整的幅度，等于 `MeshOffset`。

![](capsule_adjustment_range.png)

因为我们只能设置胶囊体的半高，所以调整的幅度得除以2。

``` cpp
    float _CapsuleHalfHeight = FInterp(CapsuleHalfHeight, CapsuleHalfHeight - MeshOffsetZMax / 2);
    GetCapsuleComponent()->SetCapsuleHalfHeight(_CapsuleHalfHeight);
```

![](after_capsule_adjustment.png)

其实效果并不是很好，因为胶囊体调整高度时是上下两头同时进行，而不是单单上头。目前没找到什么好的解决方案。

## 脚部贴合地面

TODO

# 参考

- [[UE4蓝图]虚幻4中完整实现脚部IK - 架狙只打脚](https://zhuanlan.zhihu.com/p/84399021)

# 相关

- [[玩转UE4/UE5动画系统＞应用篇＞功能模块] 之 Foot IK系统（ALS V4实现方案详解） - 开发游戏的老王](https://zhuanlan.zhihu.com/p/386921458)
- [虚幻引擎插件：使用Power IK轻松愉快地实现脚底板位置矫正 - 开发游戏的老王](https://zhuanlan.zhihu.com/p/342095610)
- [UE4实现基于预测的FootIK - sincezxl](https://zhuanlan.zhihu.com/p/380222928)