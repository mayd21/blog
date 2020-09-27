---
title: Vue3 Teleport
date: 2020-09-27 15:39:46
categories:
  - 前端
tags:
  - Vue
  - Vue3
---

Vue 鼓励我们通过将 UI 和相关行为封装到组件中来构建我们的 UI。我们可以将它们相互嵌套在一起，构建一棵树，从构成一个应用程序的 UI。但是有时组件的一部分模板只是在逻辑上属于该组件，而在技术实现上我们希望它渲染在其他的`DOM`节点下面，针对这种情况 Vue3 提供了`teleport`组件。

<!-- more -->

## Teleport

例如，我们需要一个模态框，该模态框这和该组件在逻辑上相关，但是我们希望他渲染在其他的节点上。目前是结构如下：

```html
<body>
  <div style="position: relative;">
    <h3>Tooltips with Vue 3 Teleport</h3>
    <div>
      <modal-button></modal-button>
    </div>
  </div>
</body>
```

这种结构模态框被嵌入到组件的内部，并且其`css`相互影响，而`teleport`提供一种能够将一段`HTML`渲染到指定的`DOM`节点下，同时不需要存储全局的状态或者拆分为两个组件。

```js
app.component("modal-button", {
  template: `
    <button @click="modalOpen = true">
        Open full screen modal! (With teleport!)
    </button>
    <!-- 将其渲染到 body 节点 -->
    <teleport to="body">
      <div v-if="modalOpen" class="modal">
        <div>
          I'm a teleported modal! 
          (My parent is "body")
          <button @click="modalOpen = false">
            Close
          </button>
        </div>
      </div>
    </teleport>
  `,
  data() {
    return {
      modalOpen: false,
    };
  },
});
```

## Teleport API

`<teleport>`接受两个`Props`：

- `to - string`：需要移动到的目标`DOM`。必须是合法的`query selector`或者`HTMLElement`

```html
<!-- ok -->
<teleport to="#some-id" />
<teleport to=".some-class" />
<teleport to="[data-teleport]" />

<!-- Wrong -->
<teleport to="h1" />
<teleport to="some-string" />
```

- `disabled - boolean`：`default false`。当为`true`时表示插槽内容将不会移动，但是会在原始书写的位置进行渲染。

```html
<teleport to="#popup" :disabled="displayVideoInline">
  <video src="./my-movie.mp4">
</teleport>
```

当`disabled`状态变化时，其内容会被移动，但是并不会销毁或者重新创建，将保持其原来的状态（例如 video 将保持其播放状态不变）。

## 和 Vue 组件一起使用

如果`<teleport>`包含一个 Vue 组件，那么它将作为`<teleport>`父组件的子组件存在，换句话说，`<teleport>`是透明的。

如下例子，`child-component`的父组件为`parent-component`。

```js
const app = Vue.createApp({
  template: `
    <h1>Root instance</h1>
    <parent-component />
  `,
});

app.component("parent-component", {
  template: `
    <h2>This is a parent component</h2>
    <teleport to="#endofbody">
      <child-component name="John" />
    </teleport>
  `,
});

app.component("child-component", {
  props: ["name"],
  template: `
    <div>Hello, {{ name }}</div>
  `,
});
```

## 在同一个目标上使用多个 Teleport

当需要在同一目标下渲染多个`<teleport>`时，其顺序为简单的向后追加。

```html
<teleport to="#modals">
  <div>A</div>
</teleport>
<teleport to="#modals">
  <div>B</div>
</teleport>

<!-- result-->
<div id="modals">
  <div>A</div>
  <div>B</div>
</div>
```
