---
title: Unity UI进入脚本
comments: true
toc: true
excerpt: ' '
date: 2020-05-12 11:10:42
categories: Unity
tags:
sticky: 0
recommend: 0
---
可设置位置、不透明度的变化以及完成整个过程所需要的时间。

![](UI_enter.png)

上图参数，效果表现为从下往上移入，渐显，整个过程持续一秒。

# 代码

``` csharp
using UnityEngine;
 
public class UIEnter : MonoBehaviour {
    [Header("位置")]
    public Vector2 startPosition;
    public Vector2 endPosition;
    [Header("不透明度")]
    [Range(0, 1)]
    public float startOpacity;
    [Range(0, 1)]
    public float endOpacity;
    [Header("持续时间")]
    public float duration;
 
    RectTransform m_RectTransform;
    CanvasGroup m_CanvasGroup;
    float m_XMoveSpeed;
    float m_YMoveSpeed;
    float m_FadeSpeed;
 
    void Awake() {
        m_RectTransform = GetComponent<RectTransform>();
        m_CanvasGroup = GetComponent<CanvasGroup>();
    }
 
    void OnEnable() {
        m_RectTransform.anchoredPosition = startPosition;
        if (m_CanvasGroup != null) {
            m_CanvasGroup.alpha = startOpacity;
            m_FadeSpeed = CalculateSpeed(startOpacity, endOpacity);
        } else {
            Debug.LogWarning("For the opacity changes to take effect, please add CanvasGroup component");
        }
 
        m_XMoveSpeed = CalculateSpeed(startPosition.x, endPosition.x);
        m_YMoveSpeed = CalculateSpeed(startPosition.y, endPosition.y);
    }
 
    void Update() {
        if (m_RectTransform.anchoredPosition != endPosition) {
            float desX = m_RectTransform.anchoredPosition.x + m_XMoveSpeed * Time.unscaledDeltaTime;
            float desY = m_RectTransform.anchoredPosition.y + m_YMoveSpeed * Time.unscaledDeltaTime;
            if (JudgeOutOfRange(startPosition.x, endPosition.x, desX)) {
                desX = endPosition.x;
            }
            if (JudgeOutOfRange(startPosition.y, endPosition.y, desY)) {
                desY = endPosition.y;
            }
            Vector2 desPosition = new Vector2(desX, desY);
            m_RectTransform.anchoredPosition = desPosition;
 
            if (m_CanvasGroup == null) return;
            float desOpacity = m_CanvasGroup.alpha + m_FadeSpeed * Time.unscaledDeltaTime;
            if (JudgeOutOfRange(startOpacity, endOpacity, desOpacity)) {
                desOpacity = endOpacity;
            }
            m_CanvasGroup.alpha = desOpacity;
        }
    }
 
    float CalculateSpeed(float start, float end) {
        float Offset = end - start;
        float speed = 0;
        if (duration > 0) {
            speed = Offset / duration;
        }
        return speed;
    }
 
    bool JudgeOutOfRange(float start, float end, float des) {
        return Mathf.Sign(end - start) != Mathf.Sign(end - des);
    }
}
```