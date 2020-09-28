---
title: Vue3 Global API
date: 2020-09-28 11:39:28
categories:
  - 前端
tags:
  - Vue
  - Vue3
---

Vue2.x 有许多的全局 API 和配置会改变 Vue 的全局行为，这些方法很方便，但是也带了很多的问题。从技术上来说，Vue2.x 并没有`app`的概念，我们所定义的`app`这是通过`new Vue()`创建的 Vue 实例，每个实例都会共享全局配置，这样则会带来一系列的问题。

<!-- more -->

## Vue2.x Global API

- 全局配置会很容易的导致意外污染其他测试实例。用户需要小心的保存原始的全局配置的副本，并在下一个测试时将其还原。但是有些 API 例如`Vue.use Vue.mixin`没有办法重置它们的影响，因此涉及到插件的测试就非常棘手。
- 全局配置使得很难在同一个页面上的多个`app`之间共享具有不同全局配置的 Vue 副本。

  ```js
  // this affects both root instances
  Vue.mixin({
    /* ... */
  });

  const app1 = new Vue({ el: "#app-1" });
  const app2 = new Vue({ el: "#app-2" });
  ```

## 新的 Global API：`createApp`

Vue3 中一个新的概念，调用`createApp`返回一个`app`实例。

```js
import { createApp } from "vue";
const app = createApp({});
```

`app`实例上暴露了当前全局 API 的子集，任何会改变 Vue 行为的全局 API 都会被移到`app`实例下：

| 2.x Global API               | 3.x Instance API (app)       |
| ---------------------------- | ---------------------------- |
| `Vue.config`                 | `app.config`                 |
| `Vue.config.productionTip`   | removed                      |
| `Vue.config.ignoredElements` | `app.config.isCustomElement` |
| `Vue.component`              | `app.component`              |
| `Vue.directive`              | `app.directive`              |
| `Vue.mixin`                  | `app.mixin`                  |
| `Vue.use`                    | `app.use`                    |

其他的 API 不会全局的改变 Vue 的行为，目前都被命名导出，可采用导入的方式使用：

- `Vue.nextTick`
- `Vue.observable` replaced by `Vue.reactive`
- `Vue.version`
- `Vue.compile` only in full builds
- `Vue.set` only in compat builds
- `Vue.delete` only in compat builds

```js
import { nextTick } from "vue";
nextTick(() => {
  // do someting
});
```

## 插件作者须知

通常情况下我们使用`Vue.use`来安装插件，例如在浏览器环境中安装`vue-router`插件：

```js
var inBrowser = typeof window !== "undefined";
/* … */
if (inBrowser && window.Vue) {
  window.Vue.use(VueRouter);
}
```

在 Vue3 中全局的`use`API 将不可用，调用`Vue.use`将会触发警告，相对的，最终用户需要明确的指出在`app`实例上使用插件：

```js
const app = createApp(MyApp);
app.use(VueRouter);
```

## Mounting App Instance

通过`createApp(/* option */)`初始化实例之后，可以通过`app.mount(domTarget)`来挂载 Vue 的根实例：

```js
import { createApp } from "vue";
import MyApp from "./MyApp.vue";

const app = createApp(MyApp);
app.mount("#app");
```

创建组件和指令等方式也将改写为如下形式：

```js
const app = createApp(MyApp);

app.component("button-counter", {
  data: () => ({
    count: 0,
  }),
  template: '<button @click="count++">Clicked {{ count }} times.</button>',
});

app.directive("focus", {
  mounted: (el) => el.focus(),
});

// 这样每个通过 app.mount() 挂载的 app 实例都有其自己的组件树，
// 并且有 button-counter 组件和 "focus" 指令，
// 从而不会污染全局环境
app.mount("#app");
```

## Provide / Inject

和 Vue2.x 中`provide`类似，Vue3`app`实例也可以将依赖注入到实例内部的组件中：

```js
// in the entry
app.provide("guide", "Vue 3 Guide");

// in a child component
export default {
  inject: {
    book: {
      from: "guide",
    },
  },
  template: `<div>{{ book }}</div>`,
};
```

## 在`app`间共享配置

```js
import { createApp } from "vue";
import Foo from "./Foo.vue";
import Bar from "./Bar.vue";

const createMyApp = (options) => {
  const app = createApp(options);
  app.directive("focus" /* ... */);

  return app;
};

createMyApp(Foo).mount("#foo");
createMyApp(Bar).mount("#bar");
```
