---
title: Vue3 Components Changes
date: 2020-09-29 10:19:42
categories:
  - 前端
tags:
  - Vue
  - Vue3
---

Vue3 中对组件进行一些更新，其中包括对函数组件的更新和对异步组件的支持。

<!-- more -->

# 函数组件

## 更新内容

- 在 2.x 中函数组件所带来的的性能优势在 3.x 中已经微不足道了，因此在 3.x 中推荐直接使用状态组件
- 函数组件目前只允许通过普通函数进行创建，该函数接受`props`和`context`（如：`slot attrs emit`）参数
- **BREAKING:** 单文件组件（SFC）`<template>`上的`functional`属性被移除
- **BREAKING:** 通过函数创建组件时其选项`{ functional: true }`被移除

在 Vue2 中，函数组件主要有以下两个用途：

- 函数组件有这更佳的性能
- 函数组件可以返回多个根节点

而目前 Vue3 函数组件的性能优势已经微不足道，而且状态组件同样可以返回多个根节点。

## 2.x 语法

以一个动态标题组件为例：

```js
// Vue 2 Functional Component Example
export default {
  functional: true,
  props: ["level"],
  render(h, { props, data, children }) {
    return h(`h${props.level}`, data, children);
  },
};
```

```js
// Vue 2 Functional Component Example with <template>
<template functional>
  <component
    :is="`h${props.level}`"
    v-bind="attrs"
    v-on="listeners"
  />
</template>

<script>
export default {
  props: ['level']
}
</script>
```

## 3.x 语法

```js
import { h } from "vue";

const DynamicHeading = (props, context) => {
  return h(`h${props.level}`, context.attrs, context.slots);
};

DynamicHeading.props = ["level"];

export default DynamicHeading;
```

```js
<template>
  <component
    v-bind:is="`h${props.level}`"
    v-bind="$attrs"
  />
</template>

<script>
export default {
  props: ['level']
}
</script>
```

**Differences:**

- `<template>`上的`functional`属性移除
- `listeners`现在作为`$attrs`的一部分传递，并且可以移除

# 异步组件

## 更新内容

- 新的方法`defineAsyncComponent`可以显式的定义一个异步组件
- `component`选项被重命名为`loader`
- Loader 函数不在是固有的接受`resolve`和`reject`参数，但必须返回一个 Promise

## 2.x 现有形式

异步组件通过返回一个`Promise`来实现：

```js
const asyncPage = () => import("./NextPage.vue");
```

或者通过高级的格式对象：

```js
const asyncPage = {
  component: () => import("./NextPage.vue"),
  delay: 200,
  timeout: 3000,
  error: ErrorComponent,
  loading: LoadingComponent,
};
```

## 3.x 语法

```js
import { defineAsyncComponent } from "vue";
import ErrorComponent from "./components/ErrorComponent.vue";
import LoadingComponent from "./components/LoadingComponent.vue";

// Async component without options
const asyncPage = defineAsyncComponent(() => import("./NextPage.vue"));

// Async component with options
const asyncPageWithOptions = defineAsyncComponent({
  loader: () => import("./NextPage.vue"),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent,
});
```

另外 loader 函数不是必须接受`resolve`和`reject`参数，但是必须返回一个`Promise`:

```js
// 2.x version
const oldAsyncComponent = (resolve, reject) => {
  /* ... */
};

// 3.x version
const asyncComponent = defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      /* ... */
    })
);
```
