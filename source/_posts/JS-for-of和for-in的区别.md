---
title: JS for of和for in的区别
comments: true
toc: true
excerpt: ' '
date: 2018-09-04 17:48:38
categories: Javascript
tags:
sticky: 0
recommend: 0
article:
    licenses:
---
# javascript总for of和for in的区别？

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： wujunchuan
原文标题： javascript总for of和for in的区别？
链接： https://segmentfault.com/q/1010000006658882
来源： SegmentFault 思否
转载篇幅： 全文
是否修改： 否
{% raw %}</div></article>{% endraw %}

先说结论：

1. 推荐在循环对象属性的时候，使用`for...in`,在遍历数组的时候的时候使用`for...of`。

2. `for...in`循环出的是`key`，`for...of`循环出的是`value`

3. 注意，`for...of`是ES6新引入的特性。修复了ES5引入的`for...in`的不足

4. **`for...of`不能循环普通的对象**，需要通过和`Object.keys()`搭配使用

假设我们要遍历一个数组的`valuelet aArray = ['a',123,{a:'1',b:'2'}]`

使用`for...in`循环：

``` javascript
for(let index in aArray){
    console.log(`${aArray[index]}`);
}
```

使用`for...of`循环：

``` javascript
for(var value of aArray){
    console.log(value);
}
```

咋一看好像好像只是写法不一样而已，那为什么说`for...of`修复了`for...in`的缺陷和不足。

假设我们往数组添加一个属性`name`:

`aArray.name = 'demo'`,再分别查看上面写的两个循环：

``` javascript
for(let index in aArray){
    console.log(`${aArray[index]}`); //Notice!!aArray.name也被循环出来了
}

for(var value of aArray){
    console.log(value);
}
```

所以说，作用于数组的`for-in`循环除了遍历数组元素以外,还会遍历自定义属性。

`for...of`循环不会循环对象的`key`，只会循环出数组的`value`，**因此`for...of`不能循环遍历普通对象**,对普通对象的属性遍历推荐使用`for...in`

---

# ECMAScript 6 入门 - Iterator 和 for...of 循环

{% raw %}<article class="message is-link"><div class="message-body">{% endraw %}
作者： 阮一峰
原文标题： ECMAScript 6 入门 - Iterator 和 for...of 循环
链接： https://es6.ruanyifeng.com/#docs/iterator
来源： ES6 入门教程
转载篇幅： 部分
是否修改： 否
{% raw %}</div></article>{% endraw %}

## for...of 循环

ES6 借鉴 C++、Java、C# 和 Python 语言，引入了`for...of`循环，作为遍历所有数据结构的统一的方法。

一个数据结构只要部署了`Symbol.iterator`属性，就被视为具有 `iterator` 接口，就可以用`for...of`循环遍历它的成员。也就是说，`for...of`循环内部调用的是数据结构的`Symbol.iterator`方法。

`for...of`循环可以使用的范围包括数组、`Set` 和 `Map` 结构、某些类似数组的对象（比如`arguments`对象、`DOM NodeList` 对象）、后文的 `Generator` 对象，以及字符串。

## 数组

``` javascript
const arr = ['red', 'green', 'blue'];
 
for(let v of arr) {
  console.log(v); // red green blue
}
```

`for...of循环`可以代替数组实例的`forEach`方法。

JavaScript 原有的`for...in`循环，只能获得对象的键名，不能直接获取键值。ES6 提供`for...of`循环，允许遍历获得键值。

``` javascript
var arr = ['a', 'b', 'c', 'd'];
 
for (let a in arr) {
  console.log(a); // 0 1 2 3
}
 
for (let a of arr) {
  console.log(a); // a b c d
}
```

上面代码表明，`for...in`循环读取键名，`for...of`循环读取键值。如果要通过`for...of`循环，获取数组的索引，可以借助数组实例的`entries`方法和`keys`方法（参见[《数组的扩展》](https://es6.ruanyifeng.com/#docs/array)一章）。

`for...of`循环调用遍历器接口，数组的遍历器接口只返回具有数字索引的属性。这一点跟`for...in`循环也不一样。

``` javascript
let arr = [3, 5, 7];
arr.foo = 'hello';
 
for (let i in arr) {
  console.log(i); // "0", "1", "2", "foo"
}
 
for (let i of arr) {
  console.log(i); //  "3", "5", "7"
}
```

上面代码中， **`for...of`循环不会返回数组arr的`foo`属性**。

## 类似数组的对象

类似数组的对象包括好几类。下面是 **`for...of`循环用于字符串**、`DOM NodeList` 对象、`arguments`对象的例子。

``` javascript
// 字符串
let str = "hello";
 
for (let s of str) {
  console.log(s); // h e l l o
}
 
// DOM NodeList对象
let paras = document.querySelectorAll("p");
 
for (let p of paras) {
  p.classList.add("test");
}
 
// arguments对象
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');
// 'a'
// 'b'
```

对于字符串来说，`for...of`循环还有一个特点，就是会正确识别 32 位 `UTF-16` 字符。

``` javascript
for (let x of 'a\uD83D\uDC0A') {
  console.log(x);
}
// 'a'
// '\uD83D\uDC0A'
```

并不是所有类似数组的对象都具有 `Iterator` 接口，一个简便的解决方法，就是使用`Array.from`方法将其转为数组。

``` javascript
let arrayLike = { length: 2, 0: 'a', 1: 'b' };
 
// 报错
for (let x of arrayLike) {
  console.log(x);
}
 
// 正确
for (let x of Array.from(arrayLike)) {
  console.log(x);
}
```

## 对象

**对于普通的对象，`for...of`结构不能直接使用，会报错**，必须部署了 `Iterator` 接口后才能使用。但是，这样情况下，`for...in`循环依然可以用来遍历键名。

``` javascript
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};
 
for (let e in es6) {
  console.log(e);
}
// edition
// committee
// standard
 
for (let e of es6) {
  console.log(e);
}
// TypeError: es6[Symbol.iterator] is not a function
```

上面代码表示，对于普通的对象，`for...in`循环可以遍历键名，`for...of`循环会报错。

一种解决方法是，使用`Object.keys`方法将对象的键名生成一个数组，然后遍历这个数组。

``` javascript
for (var key of Object.keys(someObject)) {
  console.log(key + ': ' + someObject[key]);
}
```

另一个方法是使用 `Generator` 函数将对象重新包装一下。

``` javascript
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}
 
for (let [key, value] of entries(obj)) {
  console.log(key, '->', value);
}
// a -> 1
// b -> 2
// c -> 3
```

## 与其他遍历语法的比较

以数组为例，JavaScript 提供多种遍历语法。最原始的写法就是`for`循环。

``` javascript
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
```

这种写法比较麻烦，因此数组提供内置的`forEach`方法。

``` javascript
myArray.forEach(function (value) {
  console.log(value);
});
```

这种写法的问题在于，**无法中途跳出`forEach`循环，`break`命令或`return`命令都不能奏效**。

`for...in`循环可以遍历数组的键名。

``` javascript
for (var index in myArray) {
  console.log(myArray[index]);
}
```

`for...in`循环有几个缺点。

- **数组的键名是数字，但是`for...in`循环是以字符串作为键名“0”、“1”、“2”等等**。
- **`for...in`循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键**。
- **某些情况下，`for...in`循环会以任意顺序遍历键名**。

总之，`for...in`循环主要是为遍历对象而设计的，不适用于遍历数组。

`for...of`循环相比上面几种做法，有一些显著的优点。

``` javascript
for (let value of myArray) {
  console.log(value);
}
```

- **有着同`for...in`一样的简洁语法，但是没有`for...in`那些缺点**。
- **不同于`forEach`方法，它可以与`break`、`continue`和`return`配合使用**。
- **提供了遍历所有数据结构的统一操作接口**。

下面是一个使用 `break` 语句，跳出`for...of`循环的例子。

``` javascript
for (var n of fibonacci) {
  if (n > 1000)
    break;
  console.log(n);
}
```

上面的例子，会输出斐波纳契数列小于等于 1000 的项。如果当前项大于 1000，就会使用`break`语句跳出`for...of`循环。