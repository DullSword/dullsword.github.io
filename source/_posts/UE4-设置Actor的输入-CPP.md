---
title: UE4 设置Actor的输入 C++
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 0
date: 2022-01-05 22:41:28
categories:
  - Unreal
  - Unreal4
tags:
---

## 实现

通过文档 [在 Actor 上设置输入](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/Input/ActorInput/) 可以得知想设置Actor的输入，需要为Actor调用 `EnableInput` 节点。

这一步很容易转换成C++代码实现：

``` cpp
// virtual void EnableInput(class APlayerController* PlayerController); 函数原型

APlayerController* PlayerController = GetWorld()->GetFirstPlayerController();   // 获取第一个玩家的控制器，也可以用 UGameplayStatics::GetPlayerController(GetWorld(), 0);
EnableInput(PlayerController);
```

启动Actor输入后，`InputComponent` 绑定操作即可。 `InputComponent` 是Actor的成员变量，类型是 `UInputComponent*` 。

``` cpp
InputComponent->BindAction("Print", IE_Pressed, this, &AMyActor::Print);
...
void AMyActor::Print() {}
```

如果我们在Actor的 `BeginPlay` 启用输入和绑定操作，那么玩家就可以在任何地方触发。通常来说，我们需要的是玩家靠近Actor后才允许触发。

``` cpp AMyActor.h
UPROPERTY()
APlayerController* PlayerController;    // 方便使用，因为 EnableInput 和 DisableInput 都需要传入 PlayerController

UPROPERTY()
class UBoxComponent* BoxTrigger;        // 需要一个碰撞组件

FInputActionBinding InputActionBinding; // BindAction 返回的结构体，移除绑定时需要用到

UFUNCTION()
void OnOverlapBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);

UFUNCTION()
void OnOverlapEnd(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);
```

``` cpp AMyActor.cpp
// On Constructor
BoxTrigger = CreateDefaultSubobject<UBoxComponent>(TEXT("Trigger"));
BoxTrigger->SetupAttachment(RootComponent);

// On BeginPlay
BoxTrigger->OnComponentBeginOverlap.AddDynamic(this, &AMyActor::OnOverlapBegin);
BoxTrigger->OnComponentEndOverlap.AddDynamic(this, &AMyActor::OnOverlapEnd);
BoxTrigger->SetHiddenInGame(false); // 因为Actor没有其他组件可以用来显示，为了方便测试，设置 BoxTrigger 在游戏中显示。

PlayerController = GetWorld()->GetFirstPlayerController();

...

void AMyActor::OnOverlapBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult)
{
    // 这里省略了对重叠对象的判断，一般来说是需要判断的
    EnableInput(PlayerController);
    InputActionBinding = InputComponent->BindAction("Print", IE_Pressed, this, &AMyActor::Print);
}

void AMyActor::OnOverlapEnd(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex)
{
    // 同上
    DisableInput(PlayerController);
    InputComponent->RemoveActionBindingForHandle(InputActionBinding.GetHandle());   // 移除绑定操作
}

void AMyActor::Print()
{
    UE_LOG(LogTemp, Warning, TEXT("Print in MyActor"));
}
```

这样就实现了只有人物进入 `BoxTrigger` 后，按 "Print" 对应的按键才会触发 `Print` 函数，在控制台打印出 "Print in MyActor" 。

## 扩展

需要注意Actor的 `InputComponent` 默认为空。之所以上面实现中没有进行判空，是因为 `EnableInput` 会判断Actor的 `InputComponent` 是否存在，不存在则创建它并绑定代理。

`EnableInput` 的主要作用是调用传入的PlayerController中的 `PushInputComponent` 函数，将Actor的 `InputComponent` 压入到PlayerController的 `CurrentInputStack` 中。压入时会从 `CurrentInputStack` 的栈顶开始遍历，找到其 `Priority` 比压入的 `InputComponent` 的 `Priority` 低的元素，把压入的 `InputComponent` 插入到其后面，也就是更接近栈顶的位置。

`Priority` 优先级从高往低为：

- 启用输入的Actor，从最晚启用者到最早启用者
- 控制器
- 关卡脚本
- Pawn

可以结合文档中的这张图进行理解。

![](input_processing_procedure.png)

<a class="tag is-dark is-medium" style="margin-bottom: 1rem" href="https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Input/" target="_blank">
<span class="icon"><i class="fas fa-camera"></i></span>&nbsp;&nbsp;
来源自 创建交互体验-输入-InputComponent 虚幻引擎文档
</a>

还可以在运行时输入命令 `ShowDebug INPUT` 来验证。下图蓝线线框中的 INPUT STACK 内的 PlayerController >> Level Blueprint >> Pawn 顺序跟上图保持一致。

![](input_stack-0.png)

再看看存在启动输入Actor的情况。和预期一致，启动输入的Actor位于栈顶。

![](input_stack-1.png)

如果其中一个 `InputComponent` 获得了输入，剩下的 `InputComponent` 就接收不到该输入了，因为 `InputActionBinding` ( `BindAction` 返回的结构体)中 `bConsumeInput` 默认为 `true` 。

可以在上面的实现中测试一下，在角色的 `SetupPlayerInputComponent` 中添加以下代码：

``` cpp
// On SetupPlayerInputComponent
PlayerInputComponent->BindAction("Print", IE_Pressed, this, &ATestProjectCharacter::Print);

...

void ATestProjectCharacter::Print()
{
    UE_LOG(LogTemp, Warning, TEXT("Print in TestProjectCharacter"));
}
```

运行游戏，按 "Print" 对应的按键时会打印 "Print in TestProjectCharacter" ，而进入Actor的 `BoxTrigger` 后再按就只打印 "Print in MyActor" ，并不会两句都打印，而离开后按就又恢复为打印 "Print in TestProjectCharacter" 。

有时候我们希望可以同时触发怎么办呢？可以通过 `InputActionBinding.bConsumeInput = false` 来允许将输入传递给下一个 `InputComponent` （ `InputActionBinding` 是个结构体，成员变量默认都是公开的，所以可以直接修改，而且似乎也并没有提供公开方法可以对其进行修改）。

## 参考

- [输入 - 虚幻引擎文档](https://docs.unrealengine.com/4.27/zh-CN/InteractiveExperiences/Input/)
- [在 Actor 上设置输入 - 虚幻引擎文档](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/Input/ActorInput/)
- [Input processing architecture diagram flow - Gamedev Guide](https://ikrima.dev/ue4guide/gameplay-programming/master-engine-flow/input-processing-architecture-diagram-flow/)