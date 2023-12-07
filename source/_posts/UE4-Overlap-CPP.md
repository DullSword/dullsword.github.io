---
title: UE4 Overlap C++
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2022-01-04 15:26:52
categories:
  - Unreal
  - Unreal4
tags:
---

## Component

### OnComponentBeginOverlap

官方示例中常使用的就是 `OnComponentBeginOverlap` 。

``` cpp
// .h
UFUNCTION()
void OnOverlapBegin(
    UPrimitiveComponent* OverlappedComponent,
    AActor* OtherActor,
    UPrimitiveComponent* OtherComp,
    int32 OtherBodyIndex,
    bool bFromSweep,
    const FHitResult& SweepResult
);

// .cpp
TriggerComponent->OnComponentBeginOverlap.AddDynamic(this, &UMyObject::OnOverlapBegin);
```

怎么看起来有点像委托，跳转到 `OnComponentBeginOverlap` 声明，可以发现它是 `FComponentBeginOverlapSignature` 类型，而 `FComponentBeginOverlapSignature` 就是一个动态多播委托。

``` cpp PrimitiveComponent.h
/** Delegate for notification of start of overlap with a specific component */
DECLARE_DYNAMIC_MULTICAST_SPARSE_DELEGATE_SixParams(
    FComponentBeginOverlapSignature, UPrimitiveComponent, OnComponentBeginOverlap,
    UPrimitiveComponent*, OverlappedComponent,
    AActor*, OtherActor,
    UPrimitiveComponent*, OtherComp,
    int32, OtherBodyIndex,
    bool, bFromSweep,
    const FHitResult &, SweepResult
);

/** 
 *  Event called when something starts to overlaps this component, for example a player walking into a trigger.
 *  For events when objects have a blocking collision, for example a player hitting a wall, see 'Hit' events.
 *
 *  @note Both this component and the other one must have GetGenerateOverlapEvents() set to true to generate overlap events.
 *  @note When receiving an overlap from another object's movement, the directions of 'Hit.Normal' and 'Hit.ImpactNormal'
 *  will be adjusted to indicate force from the other object against this object.
 */
UPROPERTY(BlueprintAssignable, Category="Collision")
FComponentBeginOverlapSignature OnComponentBeginOverlap;
```

![](OnComponentBeginOverlap.jpg)

注释里告诉我们，此组件和另一个组件都必须将 `GetGenerateOverlapEvents()` 设置为 `true` 以生成重叠事件，对应蓝图中的“生成重叠事件（Generate Overlap Events）”。

除此之外，绑定的函数需要标记为 `UFUNCTION` ，不然会报错。因为 `UFUNCTION` 可以将函数加入反射系统，而**动态**委托需要通过反射系统按函数名称查找。

![](error.jpg)

### 扩展

其实这里我挺奇怪的，都传入函数指针了，为什么还需要通过反射系统按函数名称查找？于是我打算看看 `AddDynamic` 怎么处理这个函数指针。

![](AddDynamic.jpg)

在这里生成了函数名字符串？继续跟下去看看。

![](Internal_AddDynamic.jpg)

>// 注意：我们实际上并没有存储传入的函数指针或调用它。我们只是出于类型安全的原因需要它。

我不信，快让我康康 `__Internal_BindDynamic` 到底把传入的函数指针怎么了。

![](Internal_BindDynamic.jpg)

嘿，还真的就没存也没调用，就只对它做了个判空操作，感觉它的主要作用就是生成函数名字符串。

这样只存了函数名的话，运行时就需要通过反射系统去调用对应这个函数名的函数了，所以绑定的函数需要标记为 `UFUNCTION` ，不然反射系统不知道有这么个函数。

为什么 `OnComponentBeginOverlap` 不使用普通委托而是使用动态委托？我觉得应该是为了可序列化，这个函数是可以在蓝图中进行绑定事件的。

## Actor

### OnActorBeginOverlap

如果对象是Actor，不关心是哪个组件触发了重叠，可以使用 `OnActorBeginOverlap` 代替，但获得的信息较少。同样是动态委托，绑定的函数需要标记为 `UFUNCTION` 。

``` cpp Actor.h
DECLARE_DYNAMIC_MULTICAST_SPARSE_DELEGATE_TwoParams(
    FActorBeginOverlapSignature, AActor, OnActorBeginOverlap, 
    AActor*, OverlappedActor,
    AActor*, OtherActor
);

/** 
  * Called when another actor begins to overlap this actor, for example a player walking into a trigger.
  * For events when objects have a blocking collision, for example a player hitting a wall, see 'Hit' events.
  * @note Components on both this and the other Actor must have bGenerateOverlapEvents set to true to generate overlap events.
  */
UPROPERTY(BlueprintAssignable, Category="Collision")
FActorBeginOverlapSignature OnActorBeginOverlap;
```

``` cpp
// .h
...
UFUNCTION()
void OnActorBeginOverlap(AActor* OverlappedActor, AActor* OtherActor);

// .cpp
OnActorBeginOverlap.AddDynamic(this, &AMyActor::OnActorBeginOverlap);
```

### NotifyActorBeginOverlap

还可以重写 `NotifyActorBeginOverlap` ，更为快捷。

``` cpp Actor.h
/** 
 *  Event when this actor overlaps another actor, for example a player walking into a trigger.
 *  For events when objects have a blocking collision, for example a player hitting a wall, see 'Hit' events.
 *  @note Components on both this and the other Actor must have bGenerateOverlapEvents set to true to generate overlap events.
 */
 virtual void NotifyActorBeginOverlap(AActor* OtherActor);
```

``` cpp
// .h
virtual void NotifyActorBeginOverlap(AActor* OtherActor) override;

// .cpp
void AMyActor::NotifyActorBeginOverlap(AActor* OtherActor) {
    Super::NotifyActorBeginOverlap(OtherActor);
    ...
}
```

## 参考

- [Common Snippets In C++](https://unrealcommunity.wiki/common-snippets-in-cpp-ui4jhevx)
- [NotifyActorBeginOverlap vs OnComponentBeginOverlap](https://forums.unrealengine.com/t/notifyactorbeginoverlap-vs-oncomponentbeginoverlap/132604)
- [Two questions about delegates](https://forums.unrealengine.com/t/two-questions-about-delegates/131201)
- [When to use UFUNCTION for delegates](https://stackoverflow.com/questions/53417144/when-to-use-ufunction-for-delegates)