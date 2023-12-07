---
title: Unity UICountUp脚本
comments: true
toc: true
excerpt: ' '
date: 2020-05-13 18:52:20
categories: Unity
tags:
sticky: 0
recommend: 0
---
![](UI_count_up.jpg)

# 参数说明

- Txt：Text组件
- Duration：持续时间
- Count Up Curve：变化曲线

# 使用

点开折腾，例如：

![](use.png)

# 效果

![](effect.gif)

# 代码

``` csharp
using UnityEngine;
using System.Collections;
using UnityEngine.UI;
 
public class UICountUp : MonoBehaviour {
 
    public Text txt; //Text组件
    public float duration; //持续时间
    public AnimationCurve countUpCurve = new AnimationCurve(new Keyframe(0, 0), new Keyframe(1, 1)); //变化曲线
 
    public void CountUp() {
        int startValue = 0; //起始值
        int endValue = 100; //目标值
        StartCoroutine(HandleCountUp(startValue, endValue));
    }
 
    IEnumerator HandleCountUp(int startValue, int endValue) {
        for (float timer = 0; timer < duration; timer += Time.unscaledDeltaTime) {
            float progress = timer / duration;
            int value = (int)Mathf.Lerp(startValue, endValue, countUpCurve.Evaluate(progress));
            txt.text = value.ToString();
            yield return null;
        }
    }
}
```