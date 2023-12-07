---
title: Unity Inspector ReorderableList
comments: true
toc: true
excerpt: ' '
date: 2020-06-24 18:24:53
categories: Unity
tags:
sticky: 0
recommend: 1
---
# 简单的ReorderableList

![](simple_reorderable_list.png)

`Test.cs`

``` csharp
using System;
using UnityEngine;
 
public class Test : MonoBehaviour {
    [Serializable]
    public class Item {
        public string name;
        public GameObject[] objs;
    }
 
    public Item[] itemList;
}
```

`TestEditor.cs`

``` csharp
using UnityEditor;
using UnityEditorInternal; //ReorderableList所在命名空间
using UnityEngine;
 
[CustomEditor(typeof(Test))]
public class TestEditor : Editor {
    SerializedProperty itemList;
    ReorderableList reorderableList;
 
    //EditorGUIUtility.standardVerticalSpacing 默认情况下控件之间的垂直间距使用的高度
    readonly float space = EditorGUIUtility.standardVerticalSpacing;
 
    void OnEnable() {
        itemList = serializedObject.FindProperty("itemList");    //通过名称查找序列化的属性
        reorderableList = new ReorderableList(serializedObject, itemList, true, true, true, true);
        //下列回调皆可写成Lambda表达式
        reorderableList.drawHeaderCallback = DrawListHeader;
        reorderableList.drawElementCallback = DrawListElement;
        reorderableList.elementHeightCallback = ListElementHeight;
    }
 
    //绘制表头
    void DrawListHeader(Rect rect) {
        EditorGUI.LabelField(rect, "itemList");
    }
 
    //绘制元素
    void DrawListElement(Rect rect, int index, bool selected, bool focused) {
        SerializedProperty element = itemList.GetArrayElementAtIndex(index);
        rect.y += space;
 
        SerializedProperty name = element.FindPropertyRelative("name");    //在相对于当前属性的相对路径中检索SerializedProperty
        EditorGUI.PropertyField(new Rect(rect.x, rect.y, Screen.width * .8f, EditorGUI.GetPropertyHeight(name)), name);
 
        SerializedProperty objs = element.FindPropertyRelative("objs");
        EditorGUI.PropertyField(new Rect(rect.x + 12, rect.y + EditorGUI.GetPropertyHeight(name), Screen.width * .8f, EditorGUI.GetPropertyHeight(objs, true)), objs, true);
    }
 
    //设置元素高度
    float ListElementHeight(int index) {
        SerializedProperty element = itemList.GetArrayElementAtIndex(index);
        float height = space;
 
        SerializedProperty name = element.FindPropertyRelative("name");
        SerializedProperty objs = element.FindPropertyRelative("objs");
        height += EditorGUI.GetPropertyHeight(name) + EditorGUI.GetPropertyHeight(objs, true);
        return height + space;
    }
 
    public override void OnInspectorGUI() {
        serializedObject.Update();    //更新序列化对象的表示形式
        reorderableList.DoLayoutList();    //自动布局绘制列表
        serializedObject.ApplyModifiedProperties();    //应用属性修改
    }
}
```

# 复杂的ReorderableList

![](complex_reorderable_list.png)

`Test.cs`

``` csharp
using System;
using UnityEngine;
 
public class Test : MonoBehaviour {
    [Serializable]
    public class ItemOne {
        public string name;
        public GameObject[] objs;
    }
 
    [Serializable]
    public class ItemTwo {
        public BoxCollider trigger;
        public string[] types;
        public float[] amounts;
        public bool fold;
    }
 
    public ItemOne[] itemOneList;
    public ItemTwo[] itemTwoList;
}
```

`TestEditor.cs`

``` csharp
using UnityEditor;
using UnityEditorInternal; //ReorderableList所在命名空间
using UnityEngine;
 
[CustomEditor(typeof(Test))]
public class TestEditor : Editor {
    SerializedProperty itemOneList;
    ReorderableList reorderableOneList;
    SerializedProperty itemTwoList;
    ReorderableList reorderableTwoList;
 
    //EditorGUIUtility.standardVerticalSpacing 默认情况下控件之间的垂直间距使用的高度
    readonly float space = EditorGUIUtility.standardVerticalSpacing;
    readonly float foldHeight = 15;
    readonly float amountElementHeight = 20;
 
    void OnEnable() {
        itemOneList = serializedObject.FindProperty("itemOneList");    //通过名称查找序列化的属性
        reorderableOneList = new ReorderableList(serializedObject, itemOneList, true, true, true, true);
        //下列回调皆可写成Lambda表达式
        reorderableOneList.drawHeaderCallback = DrawOneListHeader;
        reorderableOneList.drawElementCallback = DrawOneListElement;
        reorderableOneList.elementHeightCallback = OneListElementHeight;
 
        itemTwoList = serializedObject.FindProperty("itemTwoList");
        reorderableTwoList = new ReorderableList(serializedObject, itemTwoList, true, true, true, true);
        reorderableTwoList.drawHeaderCallback = DrawTwoListHeader;
        reorderableTwoList.drawElementCallback = DrawTwoListElement;
        reorderableTwoList.elementHeightCallback = TwoListElementHeight;
    }
 
    //绘制表头
    void DrawOneListHeader(Rect rect) {
        EditorGUI.LabelField(rect, "ItemOneList");
    }
 
    void DrawTwoListHeader(Rect rect) {
        EditorGUI.LabelField(rect, "ItemTwoList");
    }
 
    //绘制元素
    void DrawOneListElement(Rect rect, int index, bool selected, bool focused) {
        SerializedProperty element = itemOneList.GetArrayElementAtIndex(index);
        rect.y += space;
 
        SerializedProperty name = element.FindPropertyRelative("name");    //在相对于当前属性的相对路径中检索SerializedProperty
        EditorGUI.PropertyField(new Rect(rect.x, rect.y, Screen.width * .8f, EditorGUI.GetPropertyHeight(name)), name);
 
        SerializedProperty objs = element.FindPropertyRelative("objs");
        EditorGUI.PropertyField(new Rect(rect.x + 12, rect.y + EditorGUI.GetPropertyHeight(name), Screen.width * .8f, EditorGUI.GetPropertyHeight(objs, true)), objs, true);
    }
 
    void DrawTwoListElement(Rect rect, int index, bool selected, bool focused) {
        SerializedProperty element = itemTwoList.GetArrayElementAtIndex(index);
        rect.y += space;
 
        SerializedProperty trigger = element.FindPropertyRelative("trigger");
        float triggerHeight = EditorGUI.GetPropertyHeight(trigger);
        EditorGUI.PropertyField(new Rect(rect.x, rect.y, Screen.width * .8f, triggerHeight), trigger);
 
        //折叠
        SerializedProperty fold = element.FindPropertyRelative("fold");
        fold.boolValue = EditorGUI.Foldout(new Rect(rect.x + 12, rect.y + triggerHeight, Screen.width * .8f, foldHeight), fold.boolValue, "Amounts", true);
        if (fold.boolValue) {   //如果为true，则应渲染子对象
            SerializedProperty types = element.FindPropertyRelative("types");
            SerializedProperty amounts = element.FindPropertyRelative("amounts");
            types.arraySize = amounts.arraySize = reorderableOneList.count;     //设置types数组和amounts数组的容量
            for (int i = 0; i < types.arraySize; i++) {
                SerializedProperty typeElement = types.GetArrayElementAtIndex(i);
                SerializedProperty amountElement = amounts.GetArrayElementAtIndex(i);
 
                SerializedProperty oneListElement = itemOneList.GetArrayElementAtIndex(i);
                SerializedProperty name = oneListElement.FindPropertyRelative("name");  //取对应关系的oneListElement的name
                typeElement.stringValue = name.stringValue;
 
                EditorGUI.LabelField(new Rect(rect.x + 12, rect.y + triggerHeight + foldHeight + amountElementHeight * i + space * (i + 1), Screen.width * .3f, amountElementHeight), typeElement.stringValue);
                amountElement.floatValue = EditorGUI.Slider(new Rect(rect.x + 12 + Screen.width * .3f, rect.y + triggerHeight + foldHeight + amountElementHeight * i + space * (i + 1), Screen.width * .5f, amountElementHeight), amountElement.floatValue, 0, 1);
            }
        }
    }
 
    //设置元素高度
    float OneListElementHeight(int index) {
        SerializedProperty element = itemOneList.GetArrayElementAtIndex(index);
        float height = space;
 
        SerializedProperty name = element.FindPropertyRelative("name");
        SerializedProperty objs = element.FindPropertyRelative("objs");
        height += EditorGUI.GetPropertyHeight(name) + EditorGUI.GetPropertyHeight(objs, true);
        return height + space;
    }
 
    float TwoListElementHeight(int index) {
        SerializedProperty element = itemTwoList.GetArrayElementAtIndex(index);
        float height = space;
 
        SerializedProperty trigger = element.FindPropertyRelative("trigger");
        SerializedProperty amounts = element.FindPropertyRelative("amounts");
        SerializedProperty fold = element.FindPropertyRelative("fold");
        float triggerHeight = EditorGUI.GetPropertyHeight(trigger);
        //amount用的是EditorGUI.LabelField和EditorGUI.Slider，不是EditorGUI.PropertyField，所以无法使用EditorGUI.GetPropertyHeight得到高度，需要自己计算
        float amountHeight = (amountElementHeight + space) * amounts.arraySize;
 
        height += triggerHeight + foldHeight + (fold.boolValue ? amountHeight : 0f);    //如折叠，则不需要加上amountHeight
        return height + space;
    }
 
    public override void OnInspectorGUI() {
        serializedObject.Update();    //更新序列化对象的表示形式
        reorderableOneList.DoLayoutList();    //自动布局绘制列表
        reorderableTwoList.DoLayoutList();
        serializedObject.ApplyModifiedProperties();    //应用属性修改
    }
}
```

# 相关对象及属性说明

- [SerializedObject](https://docs.unity3d.com/ScriptReference/SerializedObject.html)：表示当前Inspector选中的一个或多个对象的序列化对象，是当前Inspector的可绘制对象。
- [EditorGUIUtility.standardVerticalSpacing](https://docs.unity3d.com/ScriptReference/EditorGUIUtility-standardVerticalSpacing.html)：默认情况下控件之间的垂直间距使用的高度
- [EditorGUI.GetPropertyHeight](https://docs.unity3d.com/ScriptReference/EditorGUI.GetPropertyHeight.html)：获取PropertyField控件所需的高度
- [SerializedObject.FindProperty](https://docs.unity3d.com/ScriptReference/SerializedObject.FindProperty.html)：通过名称查找序列化的属性
- [SerializedProperty.FindPropertyRelative](https://docs.unity3d.com/ScriptReference/SerializedProperty.FindPropertyRelative.html)：在相对于当前属性的相对路径中检索SerializedProperty
- [SerializedObject.Update](https://docs.unity3d.com/ScriptReference/SerializedObject.Update.html)：更新序列化对象的表示形式
- ReorderableList.DoLayoutList：自动布局绘制列表
- [SerializedObject.ApplyModifiedProperties](https://docs.unity3d.com/ScriptReference/SerializedObject.ApplyModifiedProperties.html)：应用属性修改

# 参考

- [「Unity3D」(10)自定义属性面板Inspector详解 - scottcgi](https://blog.csdn.net/tom_221x/article/details/79437561)
- [Unity: make your lists functional with ReorderableList - Valentin Simonov](https://va.lent.in/unity-make-your-lists-functional-with-reorderablelist/)
- [Unity ReorderableList 可重新排序的列表框使用 - 无幻](https://blog.csdn.net/akof1314/article/details/49642109)