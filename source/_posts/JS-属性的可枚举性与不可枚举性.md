---
title: JS 属性的可枚举性与不可枚举性
comments: true
toc: true
excerpt: ' '
date: 2019-02-19 10:31:35
categories: Javascript
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： ShiRouMi
原文标题： 属性的可枚举性与不可枚举性
链接： https://segmentfault.com/a/1190000014745723
来源： SegmentFault 思否
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

我们到MDN上搜索[属性的可枚举性和所有权](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)

可枚举属性是指内部可枚举标志（`enumerable`）设置为`true`的属性，自然不可枚举属性即是`enumerable`为`false`

我们看下JavaScript基本类型和基本类型包裹对象

`基本类型`是指非对象且无方法的数据。

`JavaScript`有6种基本类型：`string,number,boolean,null,undefined,symbol`

除了`null和undefined`外，所有基本类型值都有包裹这个`基本类型值的等价对象`:
`String` `Boolean` `Number` `Symbol`

`JavaScript`有7种不同类型的值

- 六种 `原型` 数据类型

    - Boolean

    - String

    - Number

    - null

    - undefined

    - Symbol

- `Object`对象

还有一个概念就是`字面量`,有以下字面量

- 数组字面量(Array)

- 布尔字面量 (Boolean)

- 浮点数字面量 (Float)

- 整数 (Int)

- 对象字面量 (Object)

- RegExp

- 字符串字面量 (String)

好了好了扯远了，为什么说这些呢，抛出问题，以上属性是否可枚举呢?

下面来看一个例子：

``` javascript
function Person() {
    this.name = 'fyflying'
}
Person.prototype = {
    hobby: 'coding'
}
var person = new Person()
Object.defineProperty(person, 'sex', {
    value: 'female'
})
for (var item in person) {
    console.log(item + ':' + person[item])
}
/**
name:fyflying
hobby:coding
**/
Object.keys(person)
// ["name"]
JSON.stringify(person)
//"{"name":"fyflying"}"
```

可以看到除了`sex`属性其他都遍历到了,而且使用`for..in`与`Object.keys`还是有区别的，区别就在于使用`for...in`还会遍历出**对象从原型链上继承来的可枚举属性**

通过`Object.defineProperty`定义的属性，该标志值默认是`false`, 所以不可枚举，通过`for..in` 和 `Object.keys()`都遍历不到。

**判断可枚举属性与不可枚举属性方法**：
`propertyIsEnumerable()`: 方法返回一个布尔值，表示指定的**自身属性**是否可枚举。(不包括原型链继承的属性）

区别以下方法：

使用 `for...in`迭代，遍历出**自身以及原型链上的可枚举的属性**，通过`hasOwnProperty`进行筛选能遍历出**自身可枚举属性**
而使用`Object.keys`直接遍历出的**自身可枚举属性组成的数组**
使用`getOwnPropertyNames`可以访问**自身可枚举属性与不可枚举属性**

另外，还有大漠的文章参考，对于以上几个方法介绍的很详细。[对象属性的枚举](https://www.w3cplus.com/javascript/how-do-i-enumerate-the-properties-of-a-javascript-object.html)