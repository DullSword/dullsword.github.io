---
title: UE5 GAS TargetActor之SingleLineTrace 崩溃篇
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2024-04-30 16:41:02
categories:
tags:
---

在尝试使用 `AGameplayAbilityTargetActor_SingleLineTrace` 时，引擎崩溃了。

```cpp
void AGameplayAbilityTargetActor_Trace::AimWithPlayerController(const AActor* InSourceActor, FCollisionQueryParams Params, const FVector& TraceStart, FVector& OutTraceEnd, bool bIgnorePitch) const
{
    // ...

    APlayerController* PC = OwningAbility->GetCurrentActorInfo()->PlayerController.Get();
    check(PC);

    // ...
}
```

在这段代码中， `check(PC);` 断言失败了，原因是 `PC` 为空 。经过检查，发现问题出在 `OwningAbility->GetCurrentActorInfo()->PlayerController.Get()` 这一行代码的 `PlayerController` 上。这个 `PlayerController` 是何许人也？

```cpp
struct GAMEPLAYABILITIES_API FGameplayAbilityActorInfo
{
    // ...

    /** PlayerController associated with the owning actor. This will often be null! */
    UPROPERTY(BlueprintReadOnly, Category = "ActorInfo")
    TWeakObjectPtr<APlayerController>    PlayerController;

    // ...
};
```

那么就寻找一下设置了这个 `PlayerController` 的地方。

```cpp
void FGameplayAbilityActorInfo::InitFromActor(AActor *InOwnerActor, AActor *InAvatarActor, UAbilitySystemComponent* InAbilitySystemComponent)
{
    // ...

    OwnerActor = InOwnerActor;

    // ...

    APlayerController* OldPC = PlayerController.Get();

    // Look for a player controller or pawn in the owner chain.
    AActor *TestActor = InOwnerActor;
    while (TestActor)
    {
        if (APlayerController * CastPC = Cast<APlayerController>(TestActor))
        {
            PlayerController = CastPC;
            break;
        }

        if (APawn * Pawn = Cast<APawn>(TestActor))
        {
            PlayerController = Cast<APlayerController>(Pawn->GetController());
            break;
        }

        TestActor = TestActor->GetOwner();
    }

    // ...
}

void FGameplayAbilityActorInfo::ClearActorInfo()
{
    OwnerActor = nullptr;
    AvatarActor = nullptr;
    PlayerController = nullptr;
    SkeletalMeshComponent = nullptr;
    MovementComponent = nullptr;
}
```

一共找到了三处设置 `PlayerController` 的地方，其中一处在 `FGameplayAbilityActorInfo::ClearActorInfo()` ，这显然不是我们要找的，而另外两处都在 `FGameplayAbilityActorInfo::InitFromActor()` 当中。

注释告诉我们，这段代码在所有者链中查找 PlayerController ~~或 pawn~~。原本以为是我没有给 Pawn 绑定 PlayerController ，但是检查后发现已经绑定了，那问题出在哪里呢？

经过调试，我发现调用栈是这样的：

```text
AGameModeBase::RestartPlayerAtPlayerStart()
            ├─ AGameModeBase::SpawnDefaultPawnFor()
            │           └─ ...
            │               └─ AActor::InitializeComponents()
            │                           └─ UAbilitySystemComponent::InitializeComponent()
            │                                       └─ UAbilitySystemComponent::InitAbilityActorInfo()
            │                                                   └─ FGameplayAbilityActorInfo::InitFromActor()
            ├─ ...
            └─ AGameModeBase::FinishRestartPlayer()
                        └─ AController::Possess()
                                    └─ AController::OnPossess()
                                                └─ APawn::PossessedBy()
```

上半部分就是在最后调用了 `FGameplayAbilityActorInfo::InitFromActor()` ，而 `FGameplayAbilityActorInfo::InitFromActor()` 已经看过了，下半部分主要就看下最后的 `APawn::PossessedBy()` 。

```cpp
void APawn::PossessedBy(AController* NewController)
{
    // ...

    Controller = NewController;
    
    // ...
}
```

所以问题是这样导致的：

首先 Pawn 的组件被初始化，然后在初始化 AbilitySystemComponent 时调用了 `initAbilityActorInfo()` ， `initAbilityActorInfo()` 又调用了 `InitFromActor()` ，但此时 OwnerActor 的 PlayerController 都还没设置上呢。因此 FGameplayAbilityActorInfo 里的 PlayerController 就被设置为了 nullptr。

那该怎么解决这个问题呢？其实调用栈里有一个熟悉的面孔—— `UAbilitySystemComponent::InitAbilityActorInfo()`。很多 GAS 的教程涉及 C++ 代码的部分都会在 Pawn 或 Character 的 `BeginPlay()` 中去调用它。

事实上 `UAbilitySystemComponent` 还提供了一个方法 `RefreshAbilityActorInfo()` 。

```cpp
void UAbilitySystemComponent::RefreshAbilityActorInfo()
{
    check(AbilityActorInfo.IsValid());
    AbilityActorInfo->InitFromActor(AbilityActorInfo->OwnerActor.Get(), AbilityActorInfo->AvatarActor.Get(), this);
}
```

它做的事情比 `InitAbilityActorInfo()` 简单，就是根据当前的 ActorInfo 刷新 AbilityActorInfo 。如注释所说：

>This will refresh the Ability's ActorInfo structure based on the current ActorInfo. That is, AvatarActor will be the same but we will look for new AnimInstance, MovementComponent, PlayerController, etc.
>
>这将根据当前的 ActorInfo 刷新技能的 ActorInfo 结构。也就是说，AvatarActor 将是相同的，但我们将寻找新的 AnimInstance、MovementComponent、PlayerController 等。

如果 AvatarActor 没有改变的话，可以使用 `RefreshAbilityActorInfo()` ，并且可以将 `RefreshAbilityActorInfo()` 的调用放在重写的 `PossessedBy()` 中。 

但如果 AvatarActor 会改变，那还是应该使用 `InitAbilityActorInfo()` 。比如说常见的重生，重生后控制的 pawn 就不是原来那个 pawn 了。

顺带一提，有些 GAS 教程在每次 `GiveAbility()` 后都执行一次 `UAbilitySystemComponent::InitAbilityActorInfo()` ，我觉得应该是不需要的。因为 `FGameplayAbilityActorInfo` 结构体是缓存与使用能力的参与者相关联的数据，而不是缓存能力相关联的数据。

最后你问我为什么不记得调用 `InitAbilityActorInfo()` ，得怪虚幻 5.3 不讲武德，虽然提供了蓝图版 AbilitySystemComponent 和蓝图节点 `GiveAbility()` ，却没有提供 `InitAbilityActorInfo()` 和 `RefreshAbilityActorInfo()` 的蓝图节点，我大意了啊，我想既然没有，那是不是帮我们处理好了，于是就没管它。直到在 GA 中使用 WaitTargetData 并试验 `AGameplayAbilityTargetActor_SingleLineTrace` 时，啪地一下就崩溃了，很快啊。
