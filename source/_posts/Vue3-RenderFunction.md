---
title: Vue3 Render Function
date: 2020-09-29 11:51:32
categories:
  - 前端
tags:
  - Vue
  - Vue3
---

Vue3 中针对渲染函数的更新。

<!-- more -->

# Render Function API

## 更新内容

此更新项对使用`<template>`无影响

- `h`现在已经采用全局导入的形式，而不是作为 render 函数的参数
- 渲染函数的参数已经和状态组件以及函数组件保持一致
- VNodes 的`props`结构更加扁平

## 渲染函数参数

### 2.x 语法

```js
// Vue 2 Render Function Example
export default {
  render(h) {
    return h("div");
  },
};
```

### 3.x 语法

```js
// Vue 3 Render Function Example
import { h } from "vue";

export default {
  render() {
    return h("div");
  },
};
```

## VNode 参数格式

### 2.x 语法

在 2.x 中`domProps`包含一个嵌套的列表：

```js
// 2.x
{
  class: ['button', 'is-outlined'],
  style: { color: '#34495E' },
  attrs: { id: 'submit' },
  domProps: { innerHTML: '' },
  on: { click: submitForm },
  key: 'submit-button'
}
```

### 3.x 语法

在 3.x 中`props`结构更加扁平

```js
// 3.x Syntax
{
  class: ['button', 'is-outlined'],
  style: { color: '#34495E' },
  id: 'submit',
  innerHTML: '',
  onClick: submitForm,
  key: 'submit-button'
}
```

## 注册组件

### 2.x 语法

当组件注册完成之后，渲染函数可以通过组件的名称的字符串作为第一个参数进行渲染

```js
// 2.x
Vue.component('button-counter', {
  data() {
    return {
      count: 0
    }
  }
  template: `
    <button @click="count++">
      Clicked {{ count }} times.
    </button>
  `
})

export default {
  render(h) {
    return h('button-counter')
  }
}
```

### 3.x 语法

在 3.x 中，VNode 是无上下文的（context-free），因此无法再使用字符串 ID 隐式的查找已经注册的组件。相反我们需要导入`resolveComponent`方法：

_`resolveComponent`只可在`render`和`setup`函数中使用_

```js
// 3.x
import { h, resolveComponent } from "vue";

export default {
  setup() {
    const ButtonCounter = resolveComponent("button-counter");
    return () => h(ButtonCounter);
  },
};
```

# 插槽统一

## 更新内容

- `this.$slots`现在向外暴露为函数
- **BREAKING:** 移除了`this.scopedSlots`

## 2.x 语法

2.x 在内容节点上定义`slot`的数据属性：

```js
// 2.x Syntax
h(LayoutComponent, [
  h('div', { slot: 'header' }, this.header),
  h('div', { slot: 'content' }, this.content)
]
```

采用如下形式引用作用域插槽

```js
// 2.x Syntax
this.$scopedSlots.header;
```

## 3.x 语法

`slot`被定义为当前节点的子对象：

```js
// 3.x Syntax
h(
  LayoutComponent,
  {},
  {
    header: () => h("div", this.header),
    content: () => h("div", this.content),
  }
);
```

引用方式：

```js
// 3.x Syntax
this.$slots.header();
```
