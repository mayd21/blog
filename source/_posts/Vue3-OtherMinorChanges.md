---
title: Vue3 Other Minor Changes
date: 2020-09-30 14:39:29
categories:
  - 前端
tags:
  - Vue
  - Vue3
---

本文包含 Vue3 中一些主要的零碎更新内容。

<!-- more -->

## 声明周期

- 生命周期选项`destoryed`被更名为`unmounted`
- 生命周期选项`beforeDestory`被更名为`beforeUnmount`

## Props 默认值

`Props`的`default`工场函数无法在获取`this`上下文

- 组件的原始`props`将作为参数传递给`default`函数
- `inject`API 可以在`default`函数内部使用

  ```js
  import { inject } from "vue";

  export default {
    props: {
      theme: {
        default(props) {
          // `props` 是组件的原始 props,
          // 可以通过 inject 访问被注入的属性
          return inject("theme", "default-theme");
        },
      },
    },
  };
  ```

## 自定义指令声明周期

自定义指令声明周期更改为与组件生命周期一致：

- API 名称修改为与组件生命周期一致
- 自定义指令将由子组件通过`v-bind="$attrs"`控制

### 3.x 语法

3.x 中自定义指令的 API 的生命周期更加紧密：

- `bind` -> `beforeMount`
- `inserted` -> `mounted`
- `beforeUpdate`: **新增！** 在元素更新自己之前调用，类似组件的声明周期
- `update` -> **移除！** 与`updated`相似之处太多，因此移除，使用`updated`代替
- `componentUpdated` -> `updated`
- `beforeUnmount` **新增!** 与声明周期的 hook 类似，将在卸载元素之前立即执行
- `unbind` -> `unmounted`

```js
const MyDirective = {
  beforeMount(el, binding, vnode, prevVnode) {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeUnmount() {}, // new
  unmounted() {},
};

const app = Vue.createApp({});

app.directive("highlight", {
  beforeMount(el, binding, vnode) {
    el.style.background = binding.value;
  },
});
```

### 实现细节

Vue3 中允许组件返回多个 DOM 节点，因此就会遇到具有多个根节点的自定义指令的问题。因此，自定义指令现在成为了虚拟 Node 数据的一部分，当在组件上使用自定义指令时，hook 会作为无关的`prop`传递给组件，最终进入`this.$attrs`中。

这也意味着可以在模板中像这样直接 hook 一个元素的生命周期，当自定义指令涉及的内容太多时，可以很方便。

```html
<div @vnodeMounted="myHook" />
```

这与属性的穿透行为是一致的，所以当一个子组件在内部元素上使用 `v-bind="$attrs"`时，它也会应用在它身上的任何自定义指令。

## Data Option

- **BREAKING:** 组件选项`data`不再接受 JavaScript `object`，必须是一个函数
- **BREAKING:** 当从`mixin`或者`extends`中合并`data`时采用浅合并而非深合并
