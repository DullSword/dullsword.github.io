---
title: UE5 硬引用与软引用
date: 2023-12-07 19:54:02
comments: true
toc: true
categories:
  - Unreal
  - Unreal5
tags:
excerpt: ' '
sticky: 0
recommend: 0
---

从一个加载资源的例子入手：

## 硬引用

假设你正在制作一个游戏，游戏中有一个主角，这个主角有好几把武器。这些武器存储在你的项目资源文件夹中，是多个静态网格体。你可能会这样做：

```cpp
UCLASS()
class YOURGAME_API AYourCharacter : public ACharacter
{
    GENERATED_BODY()

public:
    // 武器模型组件
    UPROPERTY(VisibleAnywhere)
    UStaticMeshComponent* WeaponMeshComponent;

    // 武器模型的硬引用集合
    UPROPERTY(EditDefaultsOnly)
    TArray<UStaticMesh*> WeaponMeshes;

    void SwitchWeapon();
};

void AYourCharacter::BeginPlay()
{
    Super::BeginPlay();

    if (WeaponMeshes.Num() > 0)
    {
        UStaticMesh* SelectedWeaponMesh = WeaponMeshes[0];

        WeaponMeshComponent->SetStaticMesh(SelectedWeaponMesh);
    }
}

void AYourCharacter::SwitchWeapon()
{
    // 简单起见，随机选择一个武器模型
    if (WeaponMeshes.Num() > 0)
    {
        UStaticMesh* SelectedWeaponMesh = WeaponMeshes[FMath::RandRange(0, WeaponMeshes.Num() - 1)];

        WeaponMeshComponent->SetStaticMesh(SelectedWeaponMesh);
    }
}
```

这种做法就是硬引用，除了**简单直接**外，还有一个特点就是**立即加载**，即如果某个资源在类构造函数或在蓝图编辑中分配给硬引用，则该资源将在第一次加载该类的实例时加载。这意味着当你在实例中需要使用这个资源时，它已经在内存中准备好了，你不需要等待它被加载。如此看来，硬引用的方式很不错呀。

但是假设主角同一时刻手上只会显示一把武器，多余的武器则不需要显示（比如不放在背上）。这种情景下武器模型使用的是硬引用的话，即便不需要显示，也会在加载主角时就把所有武器模型加载到内存中。这样加载不必要的资源会浪费内存，甚至有可能影响游戏的启动速度。

因此我们需要一种在真正需要的时候进行加载的引用方式，那便是软引用。

## 软引用

### TSoftObjectPtr

虚幻引擎提供了 `FSoftObjectPath` 和 `TSoftObjectPtr` 两种加载资源的软引用。

`FSoftObjectPath` 是一个简单的结构体，其中有一个字符串包含资源的完整名称，速度较慢，因为它不缓存结果。 `TSoftObjectPtr` 基本上是包含了 `FSoftObjectPath` 的 `TWeakObjectPtr` ，将用于设置特定类的模板，这样就可以限制编辑器UI仅允许选择特定类。所以下面着重介绍 `TSoftObjectPtr` 的使用。

```cpp
UCLASS()
class YOURGAME_API AYourCharacter : public ACharacter
{
    GENERATED_BODY()

public:
    UPROPERTY(VisibleAnywhere)
    UStaticMeshComponent* WeaponMeshComponent;

    // 武器模型的软引用集合
    UPROPERTY(EditDefaultsOnly)
    TArray<TSoftObjectPtr<UStaticMesh>> WeaponMeshes;

    void SwitchWeapon();

private:
    void SetWeaponMesh(TSoftObjectPtr<UStaticMesh> SelectedWeaponMesh);
};

void AYourCharacter::BeginPlay()
{
    Super::BeginPlay();

    if (WeaponMeshes.Num() > 0)
    {
        TSoftObjectPtr<UStaticMesh> SelectedWeaponMesh = WeaponMeshes[0];
        SetWeaponMesh(SelectedWeaponMesh);
    }
}

void AYourCharacter::SwitchWeapon()
{
    if (WeaponMeshes.Num() > 0)
    {
        TSoftObjectPtr<UStaticMesh> SelectedWeaponMesh = WeaponMeshes[FMath::RandRange(0, WeaponMeshes.Num() - 1)];
        SetWeaponMesh(SelectedWeaponMesh);
    }
}

void AYourCharacter::SetWeaponMesh(TSoftObjectPtr<UStaticMesh> SelectedWeaponMesh)
{
    if(SelectedWeaponMesh.IsNull()){
        return;
    }

    // 如果武器模型没有被加载到内存中
    if (SelectedWeaponMesh.IsPending())
    {
        // 同步加载武器模型
        SelectedWeaponMesh.LoadSynchronous();
    }

    UStaticMesh* LoadedMesh = SelectedWeaponMesh.Get();

    if (LoadedMesh)
    {
        WeaponMeshComponent->SetStaticMesh(LoadedMesh);
    }
}
```

上面的代码示例是同步加载，虚幻引擎还提供了异步加载的方式：

{% tabs style:boxed %}
<!-- tab id:lambda title:lambda active -->
```cpp
void AYourCharacter::SetWeaponMesh(TSoftObjectPtr<UStaticMesh> SelectedWeaponMesh)
{
    if(SelectedWeaponMesh.IsNull()){
        return;
    }

    // 需要引入 AssetManager.h
    FStreamableManager& Streamable = UAssetManager::GetStreamableManager();

    Streamable.RequestAsyncLoad(SelectedWeaponMesh.ToSoftObjectPath(), [this, SelectedWeaponMesh](){
        UStaticMesh* LoadedMesh = SelectedWeaponMesh.Get();

        if (LoadedMesh)
        {
            WeaponMeshComponent->SetStaticMesh(LoadedMesh);
        }
    });
}
```
<!-- endtab -->
<!-- tab id:FStreamableHandle title:FStreamableHandle -->
```cpp
void AYourCharacter::SetWeaponMesh(TSoftObjectPtr<UStaticMesh> SelectedWeaponMesh)
{
    if(SelectedWeaponMesh.IsNull()){
        return;
    }

    // 需要引入 AssetManager.h
    FStreamableManager& Streamable = UAssetManager::GetStreamableManager();

    // 保存返回的 Handle ，类型为TSharedPtr<FStreamableHandle>，需要声明并引入 StreamableManager.h
    Handle = Streamable.RequestAsyncLoad(SelectedWeaponMesh.ToSoftObjectPath(), FStreamableDelegate::CreateUObject(this, &AYourCharacter::OnWeaponLoaded));
}

void AYourCharacter::OnWeaponLoaded()
{
    // 检查 Handle 是否有效以及资源加载是否完成
    if (Handle.IsValid() && Handle->HasLoadCompleted())
    {
        UStaticMesh* LoadedMesh = Cast<UStaticMesh>(Handle->GetLoadedAsset());

        if (LoadedMesh)
        {
            WeaponMeshComponent->SetStaticMesh(LoadedMesh);
        }
    }
}
```
<!-- endtab -->
{% endtabs %}

`FStreamableHandle` 提供了更多的控制选项，例如检查资源是否已经加载完成、取消加载或者在资源加载完成后获取资源等。

### 注意事项

#### 垃圾回收

与弱指针一样，软引用不会阻止垃圾回收器回收资源，因为软引用并不增加对象的引用计数。因此在对象加载完成后，使用硬引用接收对象实例，避免对象实例被回收内存。

{% blockquote @UnrealEngineDocs https://docs.unrealengine.com/5.1/zh-CN/asynchronous-asset-loading-in-unreal-engine/ StreamableManager和异步加载 %}
`StreamableManager` 在调用委托之前保持对其加载的任何资源的硬引用，因此在调用委托之前，您就不必担心想要异步加载的对象会被垃圾回收。它在调用委托后释放这些引用，因此如果您仍希望保留引用，就需要在其他位置执行硬引用。
{% endblockquote %}

如果发现找不到硬引用来接收加载的对象，那么可以考虑该对象本身是否就应该使用硬引用。

在上面 `TSoftObjectPtr` 的示例中，是通过调用 WeaponMeshComponent 的 `SetStaticMesh` ，使得 WeaponMeshComponent 内部的硬引用 `TObjectPtr<class UStaticMesh> StaticMesh` 接管了加载后的武器模型。而 WeaponMeshComponent 本身使用的也是硬引用，因此只要主角存在，且没有显式地删除 WeaponMeshComponent 或卸载加载后的武器模型，那么这把武器模型就不会被垃圾回收器回收。当然如果你调用了 `SwitchWeapon` ，换了另一把武器，这时上一把武器的模型则不再被硬引用所引用，就可以被垃圾回收器回收了。

#### 软引用的类型

使用软引用时仍然会加载类型的元数据（包括它的属性、方法等），所以在类型的选择上尽可能依照最小依赖原则。

## 扩展

### TSoftClassPtr

`TSoftClassPtr` 是 `TSubClassOf` 的软引用版本，主要用途是在需要动态生成某个类的实例时，可以使用它来引用该类。

```cpp
// .h
UPROPERTY(EditDefaultsOnly)
TSoftClassPtr<AActor> MyActorClass;

// .cpp
if (!MyActorClass.IsNull() && MyActorClass.IsPending())
{
    UClass* ActorClass = MyActorClass.LoadSynchronous();
    if (ActorClass)
    {
        AActor* NewActor = GetWorld()->SpawnActor<AActor>(ActorClass);
        // 现在你可以使用NewActor了
    }
}
```

### "不同"的观点

在查阅资料时，感觉更多人是倾向于尽可能使用软引用，但也发现了“不同”的观点，即只在必要时使用软引用。之所以加了引号，是因为我不确定作者想表达的“确实需要软引用”的情况是否就是大家倾向于用软引用的情况。

{% blockquote Rama https://forums.unrealengine.com/t/question-about-soft-object-references-pointers/273863/2 %}
Unless you can make the case for truly needing a soft reference , I would say you should be managing the pointer yourself, nulling it when you don’t need it any more.

除非你能证明确实需要软引用 ，否则我会说你应该自己管理指针，当你不再需要它时将其清空。
{% endblockquote %}

## 参考

- [引用资源 - 虚幻引擎文档](https://docs.unrealengine.com/5.1/zh-CN/referencing-assets-in-unreal-engine/)
- [资产异步加载 - 虚幻引擎文档](https://docs.unrealengine.com/5.1/zh-CN/asynchronous-asset-loading-in-unreal-engine/)
- [Pointer Types](https://unrealcommunity.wiki/pointer-types-m33pysxg)
- [All about Soft and Weak pointers - Ari Arnbjörnsson](https://dev.epicgames.com/community/learning/tutorials/kx/unreal-engine-all-about-soft-and-weak-pointers)
- [Assets Streaming - Julien Delezenne](https://jdelezenne.github.io/Codex/UE4/Assets%20Streaming.html)
- [Object references in Blueprints - Morva Kristóf](https://medium.com/advanced-blueprint-scripting-in-unreal-engine/object-references-in-blueprints-4c0df7f4dc1d)
