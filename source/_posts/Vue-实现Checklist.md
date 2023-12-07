---
title: Vue 实现Checklist
comments: true
toc: true
excerpt: ' '
date: 2019-05-31 16:44:19
categories: Vue
tags:
sticky: 0
recommend: 0
---
``` html
<div>
  <div v-for="item in options" :key="item.value">
    <input type="checkbox" :id="item.value" :name="item.value" :value="item.value" v-model="selTime">
    <label :for='item.value'>{{item.label}}</label>
  </div>
</div>
```

选择项会添加到`selTime`数组里