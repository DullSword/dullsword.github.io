---
title: UE5 GAS TargetActor之SingleLineTrace 分析篇
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2024-04-25 22:06:27
categories:
  - Unreal
  - Unreal5
tags:
  - GAS
  - TargetActor
---

由于类名有点偏长，为了方便理解，替换成了我自己简化的名字。

| 原始 | 简化 |
| --- | --- |
| UAbilityTask_WaitTargetData | WaitTargetData |
| AGameplayAbilityTargetActor_Trace | Trace |
| AGameplayAbilityTargetActor | TargetActor |
| AGameplayAbilityTargetActor_SingleLineTrace | SingleLineTrace |

调用栈：

```Text
WaitTargetData: FinalizeTargetActor()
            ├─ Trace: StartTargeting()
            └─ TargetActor: ConfirmTargeting()
                        └─ Trace: ConfirmTargetingAndContinue()
                                    └─ SingleLineTrace: PerformTrace()
                                                ├─ Trace: AimWithPlayerController()
                                                │           ├─ Trace: ClipCameraRayToAbilityRange()
                                                │           └─ Trace: LineTraceWithFilter()
                                                └─ Trace: LineTraceWithFilter()
```

本文主要关注 SingleLineTrace 的核心 **lineTrace** 。由于 `ConfirmationType` 和 `Reticle` 是属于 TargetActor 的部分，所以将 `ConfirmationType` 设为即时，并选择忽略源码中所有涉及 `Reticle` 的部分。

## WaitTargetData::FinalizeTargetActor()

在 FinalizeTargetActor() 中，可以观察到 TargetActor 的 `StartTargeting()` 被调用，紧接着执行 `ConfirmTargeting()` 。

```cpp
void UAbilityTask_WaitTargetData::FinalizeTargetActor(AGameplayAbilityTargetActor* SpawnedActor) const
{
    // ...

    SpawnedActor->StartTargeting(Ability);

    if (SpawnedActor->ShouldProduceTargetData())
    {
        // If instant confirm, then stop targeting immediately.
        // Note this is kind of bad: we should be able to just call a static func on the CDO to do this. 
        // But then we wouldn't get to set ExposeOnSpawnParameters.
        if (ConfirmationType == EGameplayTargetingConfirmation::Instant)
        {
            SpawnedActor->ConfirmTargeting();
        }
        else if (ConfirmationType == EGameplayTargetingConfirmation::UserConfirmed)
        {
            // Bind to the Cancel/Confirm Delegates (called from local confirm or from repped confirm)
            SpawnedActor->BindToConfirmCancelInputs();
        }
    }
}
```

## Trace::StartTargeting()

```cpp
void AGameplayAbilityTargetActor_Trace::StartTargeting(UGameplayAbility* InAbility)
{
    Super::StartTargeting(InAbility);
    SourceActor = InAbility->GetCurrentActorInfo()->AvatarActor.Get();

    // ... Reticle
}
```

`SourceActor` 指向当前技能的 AvatarActor 。

## TargetActor::ConfirmTargeting()

```cpp
void AGameplayAbilityTargetActor::ConfirmTargeting()
{
    // ...

    if (IsConfirmTargetingAllowed())
    {
        ConfirmTargetingAndContinue();

        // ...
    }
}
```

调用了 `ConfirmTargetingAndContinue()` 。

## Trace::ConfirmTargetingAndContinue()

```cpp
void AGameplayAbilityTargetActor_Trace::ConfirmTargetingAndContinue()
{
    check(ShouldProduceTargetData());
    if (SourceActor)
    {
        bDebug = false;
        FGameplayAbilityTargetDataHandle Handle = MakeTargetData(PerformTrace(SourceActor));
        TargetDataReadyDelegate.Broadcast(Handle);
    }
}
```

这里的判断用到了之前 `StartTargeting()` 当中的 `SourceActor` ，然后就是调用了关键的 `PerformTrace()` ，通过其返回的 FHitResult 对象构造出 `FGameplayAbilityTargetDataHandle` 对象 。至于 `bDebug` 和 `FGameplayAbilityTargetDataHandle` 暂且不展开来说，先关注 `PerformTrace()` 。

## SingleLineTrace::PerformTrace()

先展示原始代码，不用担心，“瘦完身”就没剩多少了。

```cpp
FHitResult AGameplayAbilityTargetActor_SingleLineTrace::PerformTrace(AActor* InSourceActor)
{
    bool bTraceComplex = false;
    TArray<AActor*> ActorsToIgnore;

    ActorsToIgnore.Add(InSourceActor);

    FCollisionQueryParams Params(SCENE_QUERY_STAT(AGameplayAbilityTargetActor_SingleLineTrace), bTraceComplex);
    Params.bReturnPhysicalMaterial = true;
    Params.AddIgnoredActors(ActorsToIgnore);

    FVector TraceStart = StartLocation.GetTargetingTransform().GetLocation();// InSourceActor->GetActorLocation();
    FVector TraceEnd;
    AimWithPlayerController(InSourceActor, Params, TraceStart, TraceEnd);       //Effective on server and launching client only

    // ------------------------------------------------------

    FHitResult ReturnHitResult;
    LineTraceWithFilter(ReturnHitResult, InSourceActor->GetWorld(), Filter, TraceStart, TraceEnd, TraceProfile.Name, Params);
    //Default to end of trace line if we don't hit anything.
    if (!ReturnHitResult.bBlockingHit)
    {
        ReturnHitResult.Location = TraceEnd;
    }
    if (AGameplayAbilityWorldReticle* LocalReticleActor = ReticleActor.Get())
    {
        const bool bHitActor = (ReturnHitResult.bBlockingHit && (ReturnHitResult.HitObjectHandle.IsValid()));
        const FVector ReticleLocation = (bHitActor && LocalReticleActor->bSnapToTargetedActor) ? FLightWeightInstanceSubsystem::Get().GetLocation(ReturnHitResult.HitObjectHandle) : ReturnHitResult.Location;

        LocalReticleActor->SetActorLocation(ReticleLocation);
        LocalReticleActor->SetIsTargetAnActor(bHitActor);
    }

#if ENABLE_DRAW_DEBUG
    if (bDebug)
    {
        DrawDebugLine(GetWorld(), TraceStart, TraceEnd, FColor::Green);
        DrawDebugSphere(GetWorld(), TraceEnd, 100.0f, 16, FColor::Green);
    }
#endif // ENABLE_DRAW_DEBUG
    return ReturnHitResult;
}
```

“瘦身”后：

```cpp
FHitResult AGameplayAbilityTargetActor_SingleLineTrace::PerformTrace(AActor* InSourceActor)
{
    // ... FCollisionQueryParams

    FVector TraceStart = StartLocation.GetTargetingTransform().GetLocation();// InSourceActor->GetActorLocation();
    FVector TraceEnd;
    AimWithPlayerController(InSourceActor, Params, TraceStart, TraceEnd);       //Effective on server and launching client only

    // ------------------------------------------------------

    FHitResult ReturnHitResult;
    LineTraceWithFilter(ReturnHitResult, InSourceActor->GetWorld(), Filter, TraceStart, TraceEnd, TraceProfile.Name, Params);
    //Default to end of trace line if we don't hit anything.
    if (!ReturnHitResult.bBlockingHit)
    {
        ReturnHitResult.Location = TraceEnd;
    }
    
    // ... Reticle

    // ... Draw debug
    return ReturnHitResult;
}
```

可以根据方法名初步推测这段代码的目的。显然，存在一个射线追踪的起点 `TraceStart` ，然后通过 `AimWithPlayerController()` 获取射线追踪的终点，接下来进行射线追踪，最终返回命中结果。

### StartLocation

这里的 `StartLocation` 是 TargetActor 的一个成员变量，类型为 `FGameplayAbilityTargetingLocationInfo` 。在常见的使用场景中，如下图所示，在蓝图使用 WaitTargetData 节点时通过 `MakeTargetLocationInfoFromOwnerActor` 节点传入。

![StartLocation](StartLocation.png)

由于这是射线追踪的起点，因此了解其具体指向什么是非常必要的，可以在蓝图中双击 `MakeTargetLocationInfoFromOwnerActor` 节点打开跳转到对应的C++代码。

```cpp
FGameplayAbilityTargetingLocationInfo UGameplayAbility::MakeTargetLocationInfoFromOwnerActor()
{
    FGameplayAbilityTargetingLocationInfo ReturnLocation;
    ReturnLocation.LocationType = EGameplayAbilityTargetingLocationType::ActorTransform;
    ReturnLocation.SourceActor = GetActorInfo().AvatarActor.Get();
    ReturnLocation.SourceAbility = this;
    return ReturnLocation;
}
```

可以观察到，代码中构造了一个 `FGameplayAbilityTargetingLocationInfo` 对象，并相应地设置了 `LocationType` 、 `SourceActor` 和 `SourceAbility` 。虽然可以进一步查看 `FGameplayAbilityTargetingLocationInfo` 结构体的详细信息，但没必要，因为 `StartLocation.GetTargetingTransform()` 所需的数据正是 `MakeTargetLocationInfoFromOwnerActor` 已经设置好的那些参数。

已知 `StartLocation` 是个 `FGameplayAbilityTargetingLocationInfo` 对象，现在查看 `FGameplayAbilityTargetingLocationInfo::GetTargetingTransform()` 的具体实现：

```cpp
FTransform FGameplayAbilityTargetingLocationInfo::GetTargetingTransform() const
{
    //Return or calculate based on LocationType.
    switch (LocationType)
    {
    case EGameplayAbilityTargetingLocationType::ActorTransform:
        if (SourceActor)
        {
            return SourceActor->GetTransform();
        }
        break;
    case EGameplayAbilityTargetingLocationType::SocketTransform:
        if (SourceComponent)
        {
            // Bad socket name will just return component transform anyway, so we're safe
            return SourceComponent->GetSocketTransform(SourceSocketName);
        }
        break;
    case EGameplayAbilityTargetingLocationType::LiteralTransform:
        return LiteralTransform;
    default:
        check(false);
        break;
    }

    // It cannot get here
    return FTransform::Identity;
}
```

由于 `MakeTargetLocationInfoFromOwnerActor` 将 `LocationType` 设置为 `EGameplayAbilityTargetingLocationType::ActorTransform` ，我们只需关注第一个 Case。此时 `SourceActor` 再次出现，这次是为了获取它的 `Transform` 。事实上，这里就已经比较明了了，射线追踪的起点就是 `SourceActor` 的 Location，也就是 AvatarActor 的 Location。

之所以把其他 Case 也留着，是想说明 `EGameplayAbilityTargetingLocationType` 还有其他类型，其次除了 `MakeTargetLocationInfoFromOwnerActor` 以外，也可以使用 `MakeTargetLocationInfoFromOwnerSkeletalMeshComponent` 。在这种情况下，射线追踪的起点就不再是注释里的“InSourceActor->GetActorLocation();”了。

### AimWithPlayerController()

```cpp
void AGameplayAbilityTargetActor_Trace::AimWithPlayerController(const AActor* InSourceActor, FCollisionQueryParams Params, const FVector& TraceStart, FVector& OutTraceEnd, bool bIgnorePitch) const
{
    if (!OwningAbility) // Server and launching client only
    {
        return;
    }

    APlayerController* PC = OwningAbility->GetCurrentActorInfo()->PlayerController.Get();
    check(PC);

    FVector ViewStart;
    FRotator ViewRot;
    PC->GetPlayerViewPoint(ViewStart, ViewRot);

    const FVector ViewDir = ViewRot.Vector();
    FVector ViewEnd = ViewStart + (ViewDir * MaxRange);

    ClipCameraRayToAbilityRange(ViewStart, ViewDir, TraceStart, MaxRange, ViewEnd);

    FHitResult HitResult;
    LineTraceWithFilter(HitResult, InSourceActor->GetWorld(), Filter, ViewStart, ViewEnd, TraceProfile.Name, Params);

    const bool bUseTraceResult = HitResult.bBlockingHit && (FVector::DistSquared(TraceStart, HitResult.Location) <= (MaxRange * MaxRange));

    const FVector AdjustedEnd = (bUseTraceResult) ? HitResult.Location : ViewEnd;

    FVector AdjustedAimDir = (AdjustedEnd - TraceStart).GetSafeNormal();
    if (AdjustedAimDir.IsZero())
    {
        AdjustedAimDir = ViewDir;
    }

    if (!bTraceAffectsAimPitch && bUseTraceResult)
    {
        FVector OriginalAimDir = (ViewEnd - TraceStart).GetSafeNormal();

        if (!OriginalAimDir.IsZero())
        {
            // Convert to angles and use original pitch
            const FRotator OriginalAimRot = OriginalAimDir.Rotation();

            FRotator AdjustedAimRot = AdjustedAimDir.Rotation();
            AdjustedAimRot.Pitch = OriginalAimRot.Pitch;

            AdjustedAimDir = AdjustedAimRot.Vector();
        }
    }

    OutTraceEnd = TraceStart + (AdjustedAimDir * MaxRange);
}
```

```cpp
APlayerController* PC = OwningAbility->GetCurrentActorInfo()->PlayerController.Get();
```

获取玩家控制器。

```cpp
FVector ViewStart;
FRotator ViewRot;
PC->GetPlayerViewPoint(ViewStart, ViewRot);
```

获取摄像机位置及旋转。

```c++
const FVector ViewDir = ViewRot.Vector();
FVector ViewEnd = ViewStart + (ViewDir * MaxRange);
```

将旋转转换成方向，得到视野终点（ `ViewEnd` ） = 摄像机位置（ `ViewStart` ） + 摄像机方向（ `ViewDir` ） * 技能范围长度（ `MaxRange` ）。

#### ClipCameraRayToAbilityRange()

为了方便理解，下面 `ClipCameraRayToAbilityRange()` 源码中的形参名均被我替换成了实参名。

| 形参名 | 实参名 |
| --- | --- |
| CameraLocation | ViewStart |
| CameraDirection | ViewDir |
| AbilityCenter | TraceStart |
| AbilityRange | MaxRange |
| ClippedPosition | ViewEnd |

```cpp
bool AGameplayAbilityTargetActor_Trace::ClipCameraRayToAbilityRange(FVector ViewStart, FVector ViewDir, FVector TraceStart, float MaxRange, FVector& ViewEnd)
{
    FVector CameraToCenter = TraceStart - ViewStart;
    float DotToCenter = FVector::DotProduct(CameraToCenter, ViewDir);
    if (DotToCenter >= 0)       //If this fails, we're pointed away from the center, but we might be inside the sphere and able to find a good exit point.
    {
        float DistanceSquared = CameraToCenter.SizeSquared() - (DotToCenter * DotToCenter);
        float RadiusSquared = (MaxRange * MaxRange);
        if (DistanceSquared <= RadiusSquared)
        {
            float DistanceFromCamera = FMath::Sqrt(RadiusSquared - DistanceSquared);
            float DistanceAlongRay = DotToCenter + DistanceFromCamera;              //Subtracting instead of adding will get the other intersection point
            ViewEnd = ViewStart + (DistanceAlongRay * ViewDir);        //Cam aim point clipped to range sphere
            return true;
        }
    }
    return false;
}
```

```cpp
FVector CameraToCenter = TraceStart - ViewStart;
```

摄像机指向 AvatarActor 的向量。

```cpp
float DotToCenter = FVector::DotProduct(CameraToCenter, ViewDir);
```

摄像机指向 AvatarActor 的向量和摄像机方向进行点乘。由于 `ViewDir` 是个**单位**向量，所以点乘得到的结果是向量 `CameraToCenter` 在 `ViewDir` 方向上的投影长度。

```cpp
if (DotToCenter >= 0)
```

只考虑两者夹角为 0 到 90 度（包括 90 度）之间的情况。然而，在正常情况下，夹角大于 90 度的情况应该是不太可能出现的。

即便在第一人称模板中， `TraceStart` 是等同于 `ViewStart` 的，所以最后点乘的结果（ `DotToCenter` ）等于 0 ，但这也是符合条件的。

```cpp
float DistanceSquared = CameraToCenter.SizeSquared() - (DotToCenter * DotToCenter);
```

已知三角形中摄像机指向 AvatarActor 的向量（斜边 `CameraToCenter` ）和向量 `CameraToCenter` 在 `ViewDir` 方向上的投影长度（直角边 `DotToCenter` ），那么通过勾股定理可求出另一个直角边。

下图是摄像机在 AvatarActor 后上方的**侧视**示意图：

![DistanceSquared](DistanceSquared.png)

```cpp
float RadiusSquared = (MaxRange * MaxRange);
```

`MaxRange` 是技能的最大范围，和 `StartLocation` 一样是需要我们设置的。默认技能范围是以 AvatarActor 为中心的球体，所以 `MaxRange` 等于球体的半径， `RadiusSquared` 就是技能范围球体半径的平方。

```cpp
if (DistanceSquared <= RadiusSquared)
```

`DistanceSquared` 是 `TraceStart` 到 `ViewDir` 方向上的距离的平方。`RadiusSquared` 可以视为技能在 `DistanceSquared` 这个方向上能覆盖的最远距离的平方。

如果 `DistanceSquared` 小于 `RadiusSquared` ，说明 `TraceStart` 到 `ViewDir` 方向上的点在技能范围之内。

可以参考下图理解：

![RadiusSquared](RadiusSquared.png)

```cpp
float DistanceFromCamera = FMath::Sqrt(RadiusSquared - DistanceSquared);
```

`FMath::Sqrt(RadiusSquared - DistanceSquared)` 就是已知斜边的平方和一条直角边的平方求另一条直角边。别忘了 `RadiusSquared` 就是技能范围球体半径的平方。如图所示：

![DistanceFromCamera](DistanceFromCamera.png)

```cpp
float DistanceAlongRay = DotToCenter + DistanceFromCamera;              //Subtracting instead of adding will get the other intersection point
```

{% tabs style:boxed %}
<!-- tab id:adding "icon:fa-solid fa-plus" title:相加 active -->

{% asset_img DistanceAlongRay_Adding.png %}

<!-- endtab -->
<!-- tab id:Subtracting "icon:fa-solid fa-minus" title:相减 -->
注释里提到如果相减而不是相加将得到另一个交点。

{% asset_img DistanceFromCamera_Subtracting.png %}

<!-- endtab -->
{% endtabs %}

```cpp
ViewEnd = ViewStart + (DistanceAlongRay * ViewDir);        //Cam aim point clipped to range sphere
```

相机的瞄准点被限制在了能力的有效范围内，换句话说， `ViewEnd` 就是视线与技能范围相交的点。

![ViewEnd](ViewEnd.png)

注意 `ClipCameraRayToAbilityRange()` 当中的 `ViewEnd` 参数是非 const 引用，所以其实 `ClipCameraRayToAbilityRange()` 就是为了把之前粗略计算的 `ViewEnd` 修正在能力的有效范围内。

回到 `AimWithPlayerController()` ：

```cpp
FHitResult HitResult;
LineTraceWithFilter(HitResult, InSourceActor->GetWorld(), Filter, ViewStart, ViewEnd, TraceProfile.Name, Params);
```

#### LineTraceWithFilter()

```cpp
void AGameplayAbilityTargetActor_Trace::LineTraceWithFilter(FHitResult& OutHitResult, const UWorld* World, const FGameplayTargetDataFilterHandle FilterHandle, const FVector& Start, const FVector& End, FName ProfileName, const FCollisionQueryParams Params)
{
    check(World);

    TArray<FHitResult> HitResults;
    World->LineTraceMultiByProfile(HitResults, Start, End, ProfileName, Params);

    OutHitResult.TraceStart = Start;
    OutHitResult.TraceEnd = End;

    for (int32 HitIdx = 0; HitIdx < HitResults.Num(); ++HitIdx)
    {
        const FHitResult& Hit = HitResults[HitIdx];

        if (!Hit.HitObjectHandle.IsValid() || FilterHandle.FilterPassesForActor(Hit.HitObjectHandle.FetchActor()))
        {
            OutHitResult = Hit;
            OutHitResult.bBlockingHit = true; // treat it as a blocking hit
            return;
        }
    }
}
```

这里相对简单就不逐行分析了。

```cpp
TArray<FHitResult> HitResults;
World->LineTraceMultiByProfile(HitResults, Start, End, ProfileName, Params);
```

根据之前得到的 `ViewStart` 和 `ViewEnd` 以及其他配置项进行射线追踪，获取 `HitResults` 。

```cpp
OutHitResult.TraceStart = Start;
OutHitResult.TraceEnd = End;
```

设置默认命中结果的起终点。

```cpp
for (int32 HitIdx = 0; HitIdx < HitResults.Num(); ++HitIdx)
{
    const FHitResult& Hit = HitResults[HitIdx];

    if (!Hit.HitObjectHandle.IsValid() || FilterHandle.FilterPassesForActor(Hit.HitObjectHandle.FetchActor()))
    {
        OutHitResult = Hit;
        OutHitResult.bBlockingHit = true; // treat it as a blocking hit
        return;
    }
}
```

因为使用的是 `LineTraceMultiByProfile()` ，可能会有多个命中结果，遍历判断命中结果是否有效以及是否能通过过滤器，返回首个符合条件的命中结果。

再次回到 `AimWithPlayerController()` 当中，剩余的部分除了一些工具方法以外，没什么其他的方法调用，就整体一块进行分析了。

```cpp
const bool bUseTraceResult = HitResult.bBlockingHit && (FVector::DistSquared(TraceStart, HitResult.Location) <= (MaxRange * MaxRange));

const FVector AdjustedEnd = (bUseTraceResult) ? HitResult.Location : ViewEnd;

FVector AdjustedAimDir = (AdjustedEnd - TraceStart).GetSafeNormal();
if (AdjustedAimDir.IsZero())
{
    AdjustedAimDir = ViewDir;
}

if (!bTraceAffectsAimPitch && bUseTraceResult)
{
    FVector OriginalAimDir = (ViewEnd - TraceStart).GetSafeNormal();

    if (!OriginalAimDir.IsZero())
    {
        // Convert to angles and use original pitch
        const FRotator OriginalAimRot = OriginalAimDir.Rotation();

        FRotator AdjustedAimRot = AdjustedAimDir.Rotation();
        AdjustedAimRot.Pitch = OriginalAimRot.Pitch;

        AdjustedAimDir = AdjustedAimRot.Vector();
    }
}

OutTraceEnd = TraceStart + (AdjustedAimDir * MaxRange);
```

```cpp
const bool bUseTraceResult = HitResult.bBlockingHit && (FVector::DistSquared(TraceStart, HitResult.Location) <= (MaxRange * MaxRange));
```

如果 `HitResult.bBlockingHit` 为 true ，说明有符合条件的命中结果，那么判断 `TraceStart` （AvatarActor） 到命中目标的距离是否小等于 `MaxRange` （技能范围长度），即命中的目标是否在技能范围内。两者条件都满足，则使用射线追踪的结果。

```cpp
const FVector AdjustedEnd = (bUseTraceResult) ? HitResult.Location : ViewEnd;
```

如果射线追踪结果可用，那么 `AdjustedEnd` （之后射线追踪要用的终点）则为命中目标的位置，否则仍为之前得到的视线与技能范围的相交点。

```cpp
FVector AdjustedAimDir = (AdjustedEnd - TraceStart).GetSafeNormal();
if (AdjustedAimDir.IsZero())
{
    AdjustedAimDir = ViewDir;
}
```

求 `TraceStart` 指向 `AdjustedEnd` 的方向并检验有效性，否则 `AdjustedAimDir` 仍为视线方向。

```cpp
if (!bTraceAffectsAimPitch && bUseTraceResult)
{
    FVector OriginalAimDir = (ViewEnd - TraceStart).GetSafeNormal();

    if (!OriginalAimDir.IsZero())
    {
        // Convert to angles and use original pitch
        const FRotator OriginalAimRot = OriginalAimDir.Rotation();

        FRotator AdjustedAimRot = AdjustedAimDir.Rotation();
        AdjustedAimRot.Pitch = OriginalAimRot.Pitch;

        AdjustedAimDir = AdjustedAimRot.Vector();
    }
}
```

这部分跟 `bTraceAffectsAimPitch` 这个配置项有关，一般为 true ，暂且跳过，以免影响理解。

```cpp
OutTraceEnd = TraceStart + (AdjustedAimDir * MaxRange);
```

现在有 AvatarActor 的位置（`TraceStart`）、`TraceStart` 指向 `AdjustedEnd` 的方向（`AdjustedAimDir`）以及技能范围半径长度（`MaxRange`），那么就可以求得 `AdjustedAimDir` 与技能范围球体的相交点，即沿着 `AdjustedAimDir` 在技能范围内最远的那个点。

可以参考下图（有符合条件的命中的情况）进行理解：

![OutTraceEnd](OutTraceEnd.png)

终于回到了 `SingleLineTrace::PerformTrace()` ，剩余的部分如下：

```cpp
FHitResult ReturnHitResult;
LineTraceWithFilter(ReturnHitResult, InSourceActor->GetWorld(), Filter, TraceStart, TraceEnd, TraceProfile.Name, Params);
//Default to end of trace line if we don't hit anything.
if (!ReturnHitResult.bBlockingHit)
{
    ReturnHitResult.Location = TraceEnd;
}

// ... Reticle

// ... Draw debug
return ReturnHitResult;
```

```cpp
FHitResult ReturnHitResult;
LineTraceWithFilter(ReturnHitResult, InSourceActor->GetWorld(), Filter, TraceStart, TraceEnd, TraceProfile.Name, Params);
```

怎么又进行了一次射线追踪？在 `AimWithPlayerController()` 当中不是进行了一次了吗，而且按上图展示的那种有符合条件的命中的情况，直接拿那次的命中结果不就行了吗？

图样图森破，视线上命中的目标不一定是 AvatarActor 沿着 `AdjustedAimDir` 上的第一个目标。

![TooYoungTooSimple](TooYoungTooSimple.png)

`AimWithPlayerController()` 中的 `LineTraceWithFilter()` 确实已经进行了一次射线追踪，并且可能已经找到了一个有效的目标。然而，这个方法的主要目的是为了提供一个射线追踪的终点来调整玩家的瞄准方向，并不关心在这过程中找到的目标。

而在 `PerformTrace()` 中再次进行 `LineTraceWithFilter()` 的目的则是通过 `AimWithPlayerController()` 提供的射线追踪终点来找到这个瞄准方向上的第一个阻挡物，这个阻挡物将会是攻击或者技能影响的目标。这个方法是为了确认最终的目标并提供相关的命中结果。

```cpp
//Default to end of trace line if we don't hit anything.
if (!ReturnHitResult.bBlockingHit)
{
    ReturnHitResult.Location = TraceEnd;
}
```

如注释所说，如果没有命中任何内容，则返回的命中结果的 Location 默认为追踪终点。

## 扩展

实际上 SingleLineTrace 当中关于 **lineTrace** 的核心部分已经讲得差不多了，接下来补充一些被跳过的内容吧。

### FGameplayAbilityTargetDataHandle

```cpp
void AGameplayAbilityTargetActor_Trace::ConfirmTargetingAndContinue()
{
    check(ShouldProduceTargetData());
    if (SourceActor)
    {
        bDebug = false;
        FGameplayAbilityTargetDataHandle Handle = MakeTargetData(PerformTrace(SourceActor));
        TargetDataReadyDelegate.Broadcast(Handle);
    }
}

FGameplayAbilityTargetDataHandle AGameplayAbilityTargetActor_Trace::MakeTargetData(const FHitResult& HitResult) const
{
    /** Note: This will be cleaned up by the FGameplayAbilityTargetDataHandle (via an internal TSharedPtr) */
    return StartLocation.MakeTargetDataHandleFromHitResult(OwningAbility, HitResult);
}
```

当 `PerformTrace()` 返回一个 `FHitResult` 对象，会被传入到 `MakeTargetData()` 。在 `MakeTargetData()` 中又会被传入 `MakeTargetDataHandleFromHitResult()` 。

```cpp
FGameplayAbilityTargetDataHandle FGameplayAbilityTargetingLocationInfo::MakeTargetDataHandleFromHitResult(TWeakObjectPtr<UGameplayAbility> Ability, const FHitResult& HitResult) const
{
    TArray<FHitResult> HitResults;
    HitResults.Add(HitResult);
    return MakeTargetDataHandleFromHitResults(Ability, HitResults);
}

FGameplayAbilityTargetDataHandle FGameplayAbilityTargetingLocationInfo::MakeTargetDataHandleFromHitResults(TWeakObjectPtr<UGameplayAbility> Ability, const TArray<FHitResult>& HitResults) const
{
    FGameplayAbilityTargetDataHandle ReturnDataHandle;

    for (int32 i = 0; i < HitResults.Num(); i++)
    {
        /** Note: These are cleaned up by the FGameplayAbilityTargetDataHandle (via an internal TSharedPtr) */
        FGameplayAbilityTargetData_SingleTargetHit* ReturnData = new FGameplayAbilityTargetData_SingleTargetHit();
        ReturnData->HitResult = HitResults[i];
        ReturnDataHandle.Add(ReturnData);
    }

    return ReturnDataHandle;
}
```

这部分就不赘述了，主要看下 `FGameplayAbilityTargetData_SingleTargetHit` 。

```cpp
/** Target data with a single hit result, data is packed into the hit result */
USTRUCT(BlueprintType)
struct GAMEPLAYABILITIES_API FGameplayAbilityTargetData_SingleTargetHit : public FGameplayAbilityTargetData
{
    // ...

    virtual TArray<TWeakObjectPtr<AActor> > GetActors() const override
    {
        TArray<TWeakObjectPtr<AActor> > Actors;
        if (HitResult.HasValidHitObjectHandle())
        {
            Actors.Push(HitResult.HitObjectHandle.FetchActor());
        }
        return Actors;
    }

    // ...

    /** Hit result that stores data */
    UPROPERTY()
    FHitResult HitResult;

    // ...
};
```

可以看到存在一个成员变量用于存储 `FHitResult` 对象，因此 `FGameplayAbilityTargetData_SingleTargetHit` 本质上就是一个存储命中结果的结构体。

值得注意的是 `FGameplayAbilityTargetData_SingleTargetHit` 是从 `FGameplayAbilityTargetData` 继承而来的，所以 `FGameplayAbilityTargetDataHandle` 实际上就是一个用于存储 `FGameplayAbilityTargetData` 对象的结构体。只不过在这里，它具体存储的是 `FGameplayAbilityTargetData_SingleTargetHit` 对象，也就是说，它存储的是用于存储命中结果的对象。

然后在 `AGameplayAbilityTargetActor_Trace::ConfirmTargetingAndContinue()` 的最后一步，将这个 `FGameplayAbilityTargetDataHandle` 对象广播了出去，以便我们在后续的步骤中使用命中结果。

如果有留意的话，会发现在上面 `FGameplayAbilityTargetData_SingleTargetHit` 的源码中，除了成员变量 `HitResult` ，我还保留了 `GetActors()` 。`GetActors()` 是一个比较常用的方法，经过蓝图库的封装后，它在蓝图中长这样：

![GetActorsFromTargetData](GetActorsFromTargetData.png)

### bDebug

`AGameplayAbilityTargetActor_Trace` 中的 `bDebug` 是个坑。当你在蓝图中使用 WaitTargetData 节点并选择类为 `AGameplayAbilityTargetActor_Trace` 及其子类时，你可能会发现勾选 WaitTargetData 节点上对应暴露出来的 Debug 选项，完全没有如预期中那样绘制出调试线。

```cpp
void AGameplayAbilityTargetActor_Trace::ConfirmTargetingAndContinue()
{
    check(ShouldProduceTargetData());
    if (SourceActor)
    {
        bDebug = false;
        
        // ...
    }
}
```

因为在 `ConfirmTargetingAndContinue()` 当中把 `bDebug` 设置为了 false ，这导致在蓝图上的勾选变得无效。一个简单的解决方法就是继承 `AGameplayAbilityTargetActor_Trace` 或其子类，重写 `ConfirmTargetingAndContinue()` ，从而移除这行代码。

### bTraceAffectsAimPitch

```cpp
if (!bTraceAffectsAimPitch && bUseTraceResult)
{
    FVector OriginalAimDir = (ViewEnd - TraceStart).GetSafeNormal();

    if (!OriginalAimDir.IsZero())
    {
        // Convert to angles and use original pitch
        const FRotator OriginalAimRot = OriginalAimDir.Rotation();

        FRotator AdjustedAimRot = AdjustedAimDir.Rotation();
        AdjustedAimRot.Pitch = OriginalAimRot.Pitch;

        AdjustedAimDir = AdjustedAimRot.Vector();
    }
}
```

之前跳过了这部分内容，现在回过来看看是怎么回事。

```cpp
if (!bTraceAffectsAimPitch && bUseTraceResult)
```

如果 `bTraceAffectsAimPitch` 为 false 并且使用追踪结果。

```cpp
FVector OriginalAimDir = (ViewEnd - TraceStart).GetSafeNormal();
```

这里的 `ViewEnd` 是经过 `ClipCameraRayToAbilityRange()` 处理过的，也就是视线与技能范围相交的点。`GetSafeNormal()` 是求单位向量，所以 `OriginalAimDir` 是 AvatarActor 指向 `ViewEnd` 的方向。

```cpp
if (!OriginalAimDir.IsZero())
    {
        // Convert to angles and use original pitch
        const FRotator OriginalAimRot = OriginalAimDir.Rotation();

        FRotator AdjustedAimRot = AdjustedAimDir.Rotation();
        AdjustedAimRot.Pitch = OriginalAimRot.Pitch;

        AdjustedAimDir = AdjustedAimRot.Vector();
    }
```

检验 `OriginalAimDir` 的有效性，然后把 `AdjustedAimDir` 在 pitch 上的旋转修改为 `OriginalAimDir` 在 pitch 上的旋转。

![OriginalAimDir](OriginalAimDir.png)

需要注意，我的侧视图刚好就是 Pitch 这个面，不代表这些点的 Yaw 和 Roll 的值为 0 。因此代码中只修改了 `AdjustedAimDir` 的 Pitch 的值，保留了其 Yaw 和 Roll 的值。

于是我们知道当 `bTraceAffectsAimPitch` 为 false 且使用追踪结果时， AvatarActor（ `TraceStart` ）指向 视线方向命中目标的位置（ `AdjustedEnd` ） 的方向 的 Pitch 改为了 AvatarActor（ `TraceStart` ）指向 视线与技能范围相交的点（ `ViewEnd` ） 的方向 的 Pitch 。文字描述是会挺绕的，主要还是结合着图来看。

然而这有什么用呢？想象一下，这是个第三人称射击游戏，你瞄准了前方一个物体，这个物体的中心就是 `AdjustedEnd` ，这时子弹的方向的 Pitch 应该为 `AdjustedAimDir` 的 Pitch，这样才能保证打中所瞄准的目标（如果这之间没有其他物体阻挡的话），这种情况应该将 `bTraceAffectsAimPitch` 设置为 true 。

那什么情况应该将 `bTraceAffectsAimPitch` 设置为 false 呢？还是想象一下，这是个第三人称 RPG 游戏，你瞄准了一个方向，只是刚好这个方向上有个物体，但这不重要，我就希望我的角色发射的法术的**最终落点**在我瞄的方向上且是能飞行的**最远位置**，这时法术的方向的 Pitch 应该为 `OriginalAimDir` 的 Pitch 。
