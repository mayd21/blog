---
title: Vue3 Custom Element
date: 2020-09-30 10:52:04
categories:
  - 前端
tags:
  - Vue
  - Vue3
---

Vue3 中对自定义元素操作进行了更新

<!-- more -->

# 自定义元素互操作

## 更新内容

- **BREAKING:** 现在自定义元素白名单已在模板编译期间执行，应通过编译选项配置而不是运行时配置
- **BREAKING:** 特殊的`is`属性的使用只限于`<component>`标签
- **NEW:** 新的`v-is`指令来支持 2.x 中`is`被用于原生元素，以绕过原生 HTML 解析限制的使用案例

## 自主定制元素

如果要添加在 Vue 之外的自定义元素（例如：使用 Web Components API），我们需要“指导”Vue 去把它当做自定义元素对待，例如我们自定义了如下元素：

```html
<plastic-button></plastic-button>
```

### 2.x 语法

```js
// This will make Vue ignore custom element defined outside of Vue
// (e.g., using the Web Components APIs)

Vue.config.ignoredElements = ["plastic-button"];
```

### 3.x 语法

在 3.0 中是在模板编译期间执行此检查

- 使用构建步骤：通过模板编译的`isCustomElement`选项。如果使用`vue-loader`可以通过`vue-loader`的`compilerOptions`选项：
  ```js
  // in webpack config
  rules: [
    {
      test: /\.vue$/,
      use: "vue-loader",
      options: {
        compilerOptions: {
          isCustomElement: (tag) => tag === "plastic-button",
        },
      },
    },
    // ...
  ];
  ```
- 使用动态模板编译，通过`app.config.isCustomElement`：
  ```js
  const app = Vue.createApp({});
  app.config.isCustomElement = (tag) => tag === "plastic-button";
  ```
  **Note:** 运行时的配置只会影响运行时的模板编译，不会影响预编译模板

## 自定义内置元素

自定义元素规范提供了一种通过`is`属性将自定义元素用作内置元素的方法：

```html
<button is="plastic-button">Click Me!</button>
```

Vue 通过`is`属性来模拟浏览器的原生的作用，在 2.x 中被解释为渲染一个名为`plastic-button`的组件，这就阻碍了上述的原生元素的用法。

在 3.x 中 Vue 对`is`属性的特殊处理仅限于`<template>`标签：

- 当在`<template>`标签中使用时，和 2.x 相同
- 当在普通的组件中使用时，将作为一个普通的属性：
  ```html
  <foo is="bar" />
  ```
  - 2.x：渲染为`bar`组件
  - 3.x：渲染`foo`组件并传入一个`is`属性
- 当在普通的元素上使用时，它将`is`选项传递给`createElement`，并作为原生属性呈现。这支持自定义内置元素的使用。
  ```html
  <button is="plastic-button">Click Me!</button>
  ```
  - 2.x：渲染`plastic-button`组件
  - 3.x：通过调用如下方法渲染原生`button`元素：
    ```js
    document.createElement("button", { is: "plastic-button" });
    ```

## In-DOM 模板解析的方法 `v-is`

**Note:** 本部分只影响将 Vue 模板直接写 HTML 中的情况，当我们使用 In-DOM 模板时，模板需要遵守原生 HTML 解析的规则。有些 HTML 元素，诸如 `<ul>、<ol>、<table>` 和 `<select>`，对于哪些元素可以出现在其内部是有严格限制的。而有些元素，诸如 `<li>、<tr>` 和 `<option>`，只能出现在其它某些特定的元素内部。

```html
<!-- 2.x -->
<table>
  <tr is="blog-post-row"></tr>
</table>

<!-- 3.x -->
<table>
  <tr v-is="'blog-post-row'"></tr>
</table>
```

**Note:** `v-is`的功能就像 2.x 中`:is`的绑定，因此其值需要是一个 JavaScript 字面量，而不是一个字符串。
