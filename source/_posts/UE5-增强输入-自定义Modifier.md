---
title: UE5 增强输入 自定义Modifier
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 3
date: 2023-11-06 20:45:29
categories:
  - Unreal
  - Unreal5
tags:
- Enhanced Input
---
在虚幻旧版输入系统中实现人物的基础移动，运行时同时按住 WS 或 AD ，是会互相抵消，人物停止移动。但在新的增强输入系统中则不会，人物会沿着输入映射情境的 Mappings 数组中两者的后者方向移动。

例如下图，同时按住 WS ，人物会沿着 S 方向移动，因为 S 在 W 之后。

![previous_mapping](previous_mapping.png)

于是为了实现旧版输入系统的效果，我参考了 [Enhanced Input System when pressing both keys for an Axis1D](https://www.reddit.com/r/unrealengine/comments/15dqngd/enhanced_input_system_when_pressing_both_keys_for/) 讨论中 [Tezenari 的思路](https://www.reddit.com/r/unrealengine/comments/15dqngd/comment/ju47la1/?utm_source=share&utm_medium=web2x&context=3) ，创建了一个自定义修改器 "MutexKey" 来解决此问题。（当然你也可以把移动输入操作 `IA_Move` 拆成四个单独方向的输入操作来解决）

![recent_mapping](recent_mapping.png)

![running_result](running_result.gif)

## 思路

思路就是执行 "MutexKey" 修改器时，如果互斥键位处于按下的状态，那么返回 `(0,0,0)` 。这样绑定 `IA_Move` 输入动作的回调函数传入参数的值为 `(0,0)` ，人物自然不会移动。

注意：两者都需要添加该修改器。如果只有其中一者添加的话，另一方将接管控制权。

## 蓝图实现

![enhanced_input_mutex_key_modifier](enhanced_input_mutex_key_modifier.png)

### blueprintUE

[blueprintUE](https://blueprintue.com/)：一个虚幻蓝图分享网站。

如果下方显示不出来的话，可以自行点击 [Enhanced Input MutexKey Modifier](https://blueprintue.com/blueprint/p6uxpvwj/) 打开 blueprintUE 进行复制。

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
blueprintUE 会把返回节点 Return Node 显示为函数名。复制粘贴到虚幻编辑器蓝图中会正常解析成返回节点 Return Node 。
{% raw %}</div></article>{% endraw %}

<iframe src="https://blueprintue.com/render/p6uxpvwj/" width="100%" height="600" scrolling="no" allowfullscreen></iframe>

## C++实现

```cpp InputModifierMutexKey.h
#pragma once

#include "CoreMinimal.h"
#include "InputModifiers.h"
#include "InputModifierMutexKey.generated.h"

/** 
 * Mutex Key
 * If the mutex key is pressed, returns ZeroVector.
 */
UCLASS(meta = (DisplayName = "MutexKey"))
class TEST_API UInputModifierMutexKey : public UInputModifier
{
    GENERATED_BODY()

public:
    UPROPERTY(EditInstanceOnly, BlueprintReadWrite, Category = Settings, meta = (DisplayName = "Key"))
    FKey Key = EKeys::Invalid;

protected:
    virtual FInputActionValue ModifyRaw_Implementation(const UEnhancedPlayerInput* PlayerInput, FInputActionValue CurrentValue, float DeltaTime) override;
};
```

```cpp InputModifierMutexKey.cpp
#include "InputModifierMutexKey.h"
#include "EnhancedPlayerInput.h"

FInputActionValue UInputModifierMutexKey::ModifyRaw_Implementation(const UEnhancedPlayerInput* PlayerInput, FInputActionValue CurrentValue, float DeltaTime)
{
    const APlayerController* PC = PlayerInput->GetOuterAPlayerController();

    if (!PC)
    {
        return CurrentValue;
    }

    FVector Value = CurrentValue.Get<FVector>();
    if (Key.IsValid() && PC->IsInputKeyDown(Key))
    {
        Value = FVector::ZeroVector;
    }

    return FInputActionValue(CurrentValue.GetValueType(), Value);
}
```
