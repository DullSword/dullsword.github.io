---
title: Unity SerializeField和Serializable
comments: true
toc: true
excerpt: ' '
date: 2021-03-03 15:49:50
categories: Unity
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 赵青青
原文标题： SerializeField和Serializable
链接： https://www.cnblogs.com/zhaoqingqing/p/3995304.html
来源： Qing's Blog
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

# Serialize功能

Unity3D 中提供了非常方便的功能可以帮助用户将 成员变量 在Inspector中显示，并且定义Serialize关系。

简单的说，在没有自定义Inspector的情况下所有显示在Inspector 中的属性都同时具有Serialize功能。

换句话说，就是你在Inspector看到什么，保存游戏的时候，这些值就会被保存成二进制文件。

本文说说可被Serialize的变量的定义方法

## 1. public 变量

在没有加入任何Attribute的前提下， `public` 变量是默认被视为可以被Serialize的。

```csharp
public int MaxExp;
```

## 2. [SerializeField] Attribute

**强制unity去序列化一个私有域**

这是一个内部的unity序列化功能，有时候我们需要Serialize一个 `private` 或者 `protected` 的属性，这个时候可以使用[SerializeField]这个Attribute:

```csharp
[SerializeField]
protected int foobar = 0;
```

注意: 这样定义出的成员变量也是会在Inspector中显示出来。

在Unity最新的UI系统中，UI属性上方全部添加[SerializeField] ，如下所示

```csharp
[SerializeField]
private Button btn1;
```

SerializeField参考文档：[SerializeField](https://docs.unity3d.com/2021.1/Documentation/ScriptReference/SerializeField.html)

## 3. 单独的class或struct

**Serializable是.Net自带的序列化**

有时候我们会自定义一些单独的 `class/struct` , 由于这些类并没有从 `MonoBehavior` 派生所以默认并不被Unity3D识别为可以Serialize的结构。自然也就不会在Inspector中显示。我们可以通过添加 [System.Serializable]这个Attribute使Unity3D检测并注册这些类为可Serialize的类型。具体做法如下：

```csharp
[System.Serializable]
public class FooBar {
    public int foo = 5;
    public int bar = 10;
}
```

注意：Serializable只可以对`class`,`struct`,`enum`,`delegate`进行序列化，不可以对属性序列化

## 4. ScriptableObject

ScriptableObject 类型经常用于存储一些unity3d本身不可以打包的一些object，比如字符串，一些类对象等。用这个类型的子类型，则可以用BuildPipeline打包成assetbundle包供后续使用，非常方便，具体请参考 [[cb]ScriptableObject 序列化](https://www.cnblogs.com/zhaoqingqing/p/3775069.html)

# NonSerialize的变量的定义方法

## 1. protected, private, internal 变量

默认情况下， `protected` , `private` , `internal` 变量将不会被serialize.

## 2. [System.NonSerialized] Attribute

有时候我们需要定义一些 `public` 变量方便操作，但是又不希望这些变量保留。这个时候可以利用[System.NonSerialized]来完成这个操作:

```csharp
[System.NonSerialized] public float foobar = 1.0f;
```

## 3. readonly, const, static 修饰符

如果变量加入了 `readonly`, `const`, `static` 等修饰符，无论他的serialize设置如何，都将不会进行serialize

## 4. Dictionary<T,K>

Unity3D可以对 `List<T>` 进行序列化显示，但是由于他们的程序员偷懒或不够强大，以至于我们到现在都不能serialize `Dictionary<T,K>` 这么一个较为常用的类型。通常我们会通过Serialize一份 `List<T>` ，然后在 Awake中初始化Dictionary的方法来完成Dictionary的serialize操作。如:

```csharp
[System.Serializable]
public class NameToID {
    public string name = "";
    public int ID = -1;
}
public List<NameToID> nameToIDList = new List<NameToID>();  Dictionary<string,int> nameToID = new Dictionary<string,int>();  //
void Awake () {
    foreach ( NameToID info in nameToIDList ) {
        nameToID[info.name] = info.ID;
    }
    nameToIDList = null; // put it null make garbage collect it (I wish)
}
```

# 在Inspector中的显示

变量在Inspector中会根据变量的大写字母来隔开来显示，并且会将首字母强制大写的方式显示。 如:

```csharp
public int myFooBar = 0;
```

在GUI将会显示为: `My Foo Bar`