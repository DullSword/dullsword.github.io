---
title: JS Math.random()在指定的范围内生成随机数
date: 2018-07-07 15:53:37
comments: true
toc: true
categories: Javascript
tags: 随机数
excerpt: ' '
sticky: 0
recommend: 0
---
{% raw %}<article class="message is-info"><div class="message-body">{% endraw %}
最小值n，最大值m
{% raw %}</div></article>{% endraw %}

``` javascript
Math.random()*(m-n)+n;
```

由于<strong>Math.random()</strong>函数返回一个浮点,  伪随机数在范围<strong>[0，1)</strong>，所以Math.random()*(m-n)+n返回的结果为<strong>[n，m)</strong>。

---

# n，m为**整数**，需要<strong>[n，m)</strong>

``` javascript
Math.floor( Math.random() * (m-n) )+n;
```

---

# n，m为**整数**，需要<strong>[n，m]</strong>

``` javascript
Math.floor( Math.random()*(m-n+1) )+n;
```

{% raw %}<article class="message is-danger"><div class="message-body">{% endraw %}
以下内容已失效，请使用上述Math.floor( Math.random()*(m-n+1) )+n☝️
{% raw %}</div></article>{% endraw %}

~~Math.random()*(m-n+1)的取值为[0,m-n+1)，再加n就是[n,m+1)，但这样就可能取到(m,m+1)的数，所以向下取整。~~

~~Math.round( Math.random()*(m-n) )+n;~~

~~round的舍入其实并不公平，小数部分恰巧等于0.5时会舍入到相邻的在正无穷方向上的整数~~

~~( Math.random()*(m-n) ).toFixed(num)+n;~~

~~原理银行家舍入法，toFixed参数num规定小数的位数，是 0 ~ 20 之间的值，包括 0 和 20，如果省略了该参数，将用 0 代替。~~

~~toFixed（银行家舍入法）及其缺陷和解决方法~~

~~https://dullsword.github.io/2018/11/12/JS-toFixed%EF%BC%88%E9%93%B6%E8%A1%8C%E5%AE%B6%E8%88%8D%E5%85%A5%E6%B3%95%EF%BC%89%E5%8F%8A%E5%85%B6%E7%BC%BA%E9%99%B7%E5%92%8C%E8%A7%A3%E5%86%B3%E6%96%B9%E6%B3%95/~~

---

# n，m为**整数**，需要<strong>(n，m]</strong>

``` javascript
Math.ceil( Math.random()*(m-n) )+n;
```

---
