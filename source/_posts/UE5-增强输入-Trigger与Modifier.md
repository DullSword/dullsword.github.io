---
title: UE5 增强输入 Trigger与Modifier
comments: true
toc: true
excerpt: ' '
sticky: 0
recommend: 3
date: 2023-11-06 17:02:45
categories:
  - Unreal
  - Unreal5
tags:
- Enhanced Input
---

## InputMappingContext 和 InputAction 都设置有 Trigger 、 Modifier

执行顺序：

1. InputMappingContext Trigger
2. InputMappingContext Modifier
3. InputAction Trigger
4. InputAction Modifier

## Trigger

以下的 `ActuationThreshold` 、`TapReleaseTimeThreshold` 等为代码中的变量命名，在编辑器 InputAction 面板中有对应的设置。

- **Down** ：超过 `ActuationThreshold` 后 -> 触发n次
- **Pressed** ：超过 `ActuationThreshold` 后 -> 触发1次
- **Released** ：超过 `ActuationThreshold` 后 + 松开(值小于 `ActuationThreshold`) -> 触发一次
- **Tap** ：超过 `ActuationThreshold` 后 + 点按时间小于 `TapReleaseTimeThreshold` -> 触发一次
- **Pulse** ：超过 `ActuationThreshold` 后 + 触发次数小于 `TriggerLimit` + 输入时间大于 `间隔 * (首次是否触发 ? 触发次数 : 触发次数 + 1)` -> 触发
  - Completed 仅在达到重复限制或触发后立即释放输入时触发。否则，当释放输入时会触发 Canceled。
- **Hold** ：超过 `ActuationThreshold` 后 + 输入时间大于 `HoldTimeThreshold` + `(是否首次触发 || 非仅一次性)` -> 触发
- **Hold And Release** ：超过 `ActuationThreshold` 后 + 输入时间大于 `HoldTimeThreshold` + 松开(值小于 `ActuationThreshold`) -> 触发
- **ChordAction** ：仅由另一个输入动作触发
  - 该触发器本身的 `ActuationThreshold` 并不起作用，也符合文档中的说法：
    >The one exception to this rule is the "Chorded Action" Input Trigger, which is only triggered with another Input Action. By default, any user activity on an input will trigger on every tick.
    >
    >此规则的一个例外是"和弦动作"输入触发器，该触发器仅通过另一个输入动作触发。默认情况下，输入上的任意用户活动都会在每个更新函数上触发。

    ```cpp
    ETriggerState UInputTriggerChordAction::UpdateState_Implementation(const UEnhancedPlayerInput* PlayerInput, FInputActionValue ModifiedValue, float DeltaTime)
    {
        // Inherit state from the chorded action
        const FInputActionInstance* EventData = PlayerInput->FindActionInstanceData(ChordAction);
        return EventData ? EventData->TriggerStateTracker.GetState() : ETriggerState::None;
    }
    ```

  - 如果只单独使用 ChordAction 触发器，一旦触发，后续将输入值慢慢调整为0（例如轻推手柄摇杆），也仍然会继续触发。

    ![IMC_Test](IMC_Test.png)
    ![IA_Test0](IA_Test0.png)
    {% tabs style:boxed %}
    <!-- tab id:ChordActionInCorrect "icon:fa-solid fa-book" title:错误用法 active -->
    单独使用 ChordAction 触发器

    ![IA_Test1_Incorrect](2023/11/06/UE5-增强输入-Trigger与Modifier/IA_Test1_Incorrect.png)
    <!-- endtab -->
    <!-- tab id:ChordActionCorrect "icon:fa-solid fa-user" title:正确用法 -->
    同时使用其他触发器来控制该动作除了前置条件（ChordAction）以外自身的触发条件

    ![IA_Test1_Correct](2023/11/06/UE5-增强输入-Trigger与Modifier/IA_Test1_Correct.png)
    <!-- endtab -->
    ...
    {% endtabs %}
- **Combo** ：取消 Combo 的输入动作未触发 + Combo 按顺序输入 + 当前输入动作的输入时间小于对应的 `TimeToPressKey` + 已完成 Combo 中的所有输入动作 -> 触发1次
  - 不会检查第一个输入动作的 `TimeToPressKey`

    ```cpp
    // Reset if we take too long to hit the action
    if (CurrentComboStepIndex > 0)
    {
        CurrentTimeBetweenComboSteps += DeltaTime;
        if (CurrentTimeBetweenComboSteps >= ComboActions[CurrentComboStepIndex].TimeToPressKey)
        {
            CurrentComboStepIndex = 0;
            CurrentAction = ComboActions[CurrentComboStepIndex].ComboStepAction;    // Reset for fallthrough
        }
    }
    ```

### 类型

>输入触发器分为三种类型：
>
>- 显式（Explicit） 类型将导致输入在输入触发器成功时成功。
>- 隐式（Implicit） 类型将导致输入仅在输入触发器和所有其他隐式类型输入触发器都成功时成功。
>- 阻碍（Blocker） 类型将导致输入在输入触发器成功时失败。

官网文档中有关于每种触发器类型在针对其他触发器类型的情形下如何交互的逻辑示例，但感觉说得不够详细：

>隐式 == 0，显式 == 0 - 始终触发，除非值为0
>
>Implicits == 0, Explicits == 0 - Always fires, unless the value is 0.

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
"隐式 == 0"中的"0"是指什么？"除非值为0"的"值(the value)"又是指什么值？
{% raw %}</div></article>{% endraw %}

在多轮询问 Bing Chat 后，记录下算是比较能信服的说法：

>在增强型输入系统中，每个输入动作都可以有多个触发器，这些触发器可以是显式的、隐式的或阻碍的。
>
>当我们说"隐式 == 0，显式 > 0"时，在这里，"0"和">0"是开发者在一个特定的输入动作（InputAction）下创建的触发器的数量。"隐式"和"显式"是指触发器的类型。
>
>以下四种情况描述了不同类型和数量的触发器如何影响输入动作的触发：
>
>- 隐式 == 0，显式 == 0：这意味着没有任何触发器，输入动作将始终触发，除非输入值为0。
>- 隐式 == 0，显式 > 0：这意味着只有显式触发器，只要有一个显式触发器被触发，输入动作就会被触发。
>- 隐式 > 0，显式 == 0：这意味着只有隐式触发器，所有隐式触发器都必须被触发，输入动作才会被触发。
>- 隐式 > 0，显式 > 0：这意味着既有隐式触发器又有显式触发器，所有隐式触发器和至少一个显式触发器必须被触发，输入动作才会被触发。

其实只有当你需要一个输入动作添加多个触发器或是自定义触发器时，才需要考虑触发器的类型。

## Modifier

- **DeadZone** ：
  >LowerThreshold -> UpperThreshold 范围内的输入值将从 0 -> 1 重新映射。

  类型：
  - Axis（轴）：单独计算每个维度，用于处理一维输入，如键盘按键。
  - Radial（径向）：用于处理二维输入，如手柄摇杆。
- **Negate** ：对特定输入轴的值取反。
- **SwizzleAxis** ：按指定顺序交换轴的值。
- **Scalar** ：乘以标量来缩放输入值。可以用来调整互动的灵敏度。
- **ScaleByDeltaTime** ：乘以 DeltaTime 来缩放输入值。
- **FOVScaling** ： 根据玩家相机的 FOV 值来缩放输入值。
- **ToWorldSpace** ：
  >自动将输入操作值内的轴转换为世界空间，允许将结果直接插入到采用世界空间值的函数中。

  ```cpp
  FInputActionValue UInputModifierToWorldSpace::ModifyRaw_Implementation(const UEnhancedPlayerInput* PlayerInput, FInputActionValue CurrentValue, float DeltaTime)
    {
        FVector Converted = CurrentValue.Get<FVector>();
        switch (CurrentValue.GetValueType())
        {
        case EInputActionValueType::Axis3D:
            // Input Device Z = World Forward (X), Device X = World Right (Y), Device Y = World Up (Z)
            Converted = FVector(Converted.Z, Converted.X, Converted.Y);
            break;
        case EInputActionValueType::Axis2D:
            // Swap axes so Input Device Y axis becomes World Forward (X), Device X becomes World Right (Y)
            Swap(Converted.X, Converted.Y);
            break;
        case EInputActionValueType::Axis1D:
        case EInputActionValueType::Boolean:
            // No conversion required
            break;
        }
        return FInputActionValue(CurrentValue.GetValueType(), Converted);
    }
  ```

- **ResponseCurveExponential** ：根据设置对输入值做对应的次方。
- **ResponseCurveUser** ：根据用户定义曲线对输入值做对应的次方。
- **Smooth** ：
  >在多个帧上平滑输入

## 参考

- [增强输入 - 虚幻引擎文档](https://docs.unrealengine.com/5.1/zh-CN/enhanced-input-in-unreal-engine/)
- [[中文直播]第39期 | 虎跳龙拿--新一代增强输入框架EnhancedInput | Epic 大钊](https://www.bilibili.com/video/BV14r4y1r7nz/?spm_id_from=333.880.my_history.page.click&vd_source=58b361f2927e97f2b7ccd1221fc04ea7)
- [Enhanced Input in UE5 - Ben Hoffman](https://dev.epicgames.com/community/learning/tutorials/eD13/unreal-engine-enhanced-input-in-ue5)
- [Unreal Engine 5.1 - Migrating to Enhanced Input System - CodeLikeMe](https://dev.epicgames.com/community/learning/tutorials/KwE1/unreal-engine-5-1-migrating-to-enhanced-input-system)
- [UnrealEngine：输入框架 - 花游倩](https://zhuanlan.zhihu.com/p/605137767)
- [UE5 -- EnhancedInput(增强输入系统) - 易米八一](https://zhuanlan.zhihu.com/p/470949422)
- [Enhanced Input - What you need to know - unrealdirective](https://www.unrealdirective.com/articles/enhanced-input-what-you-need-to-know)
