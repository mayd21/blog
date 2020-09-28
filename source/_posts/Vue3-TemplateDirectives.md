---
title: Vue3 Template Directives
date: 2020-09-28 14:42:03
categories:
  - 前端
tags:
  - Vue
  - Vue3
---

Vue3 中对 Vue2.x 中的模板指令有一些破坏性的更新，本文对其中破坏性的更新进行介绍。

<!-- more -->

# `v-model`

## 更新内容

- **BREAKING:** 当在自定义组件上使用时，`v-model`的属性和时间的默认名称改变：
  - prop: `value` -> `modelValue`；
  - event: `input` -> `update:modelValue`；
- **BREAKING:** `v-bind`的`.sync`修饰符和组件的`model`选项被移除，由`v-model`的参数代替。
- **NEW:** 多`v-model`可绑定在同一个组件内
- **NEW:** 可创建自定义的`v-model`修饰符

## 2.x 语法

在 Vue2.0`v-model`只能使用`value`作为参数，如果需要使用不同的参数用于不同的目的可以使用`v-bind.sync`代替。在 Vue2.2 中就提供了为`v-model`自定义参数和时间的能力，但是仍然限制了在组件中只能使用一个`v-model`。Vue3 中为这两种双向绑定方式提供了统一标准，减少困惑，并且更加灵活。

### `v-model`

```html
<ChildComponent v-model="pageTitle" />

<!-- would be shorthand for: -->
<ChildComponent :value="pageTitle" @input="pageTitle = $event" />
```

自定义`v-model`参数和事件

```html
<!-- ParentComponent.vue -->
<ChildComponent v-model="pageTitle" />
```

```js
// ChildComponent.vue

export default {
  model: {
    prop: "title",
    event: "change",
  },
  props: {
    // 使用 `value` 用于其他目的
    value: String,
    // 使用 `title` 作为 prop 取代 `value`
    title: {
      type: String,
      default: "Default title",
    },
  },
};
```

### 使用`v-bind.sync`

有些情况我们需要除了`v-model`的双向绑定，为此 Vue 推荐以`update:myPropName`模式触发事件的方式，例如`ChildComponent`提供了一个`title`参数，我们可以用以下方法表达对其赋新值的意图：

```js
this.$emit("update:title", newTitle);
```

然后父组件可以监听那个事件并根据需要更新一个本地的数据 property。例如：

```html
<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

为了方便起见，Vue 为这种模式提供了一个缩写：

```html
<ChildComponent :title.sync="pageTitle" />
```

## 3.x 语法

在 3.x 中`v-model`相当于采用一个`modelValue`参数和`update:modelValue`触发事件：

```html
<ChildComponent v-model="pageTitle" />

<!-- would be shorthand for: -->
<ChildComponent
  :modelValue="pageTitle"
  @update:modelValue="pageTitle = $event"
/>
```

### `v-model`参数

可以直接将参数传递给`v-model`:

```html
<ChildComponent v-model:title="pageTitle" />

<!-- would be shorthand for: -->
<ChildComponent :title="pageTitle" @update:title="pageTitle = $event" />
```

同样可以代替`.sync`修饰符实现多个`v-model`:

```html
<ChildComponent v-model:title="pageTitle" v-model:content="pageContent" />

<!-- would be shorthand for: -->
<ChildComponent
  :title="pageTitle"
  @update:title="pageTitle = $event"
  :content="pageContent"
  @update:content="pageContent = $event"
/>
```

### `v-model`修饰符

3.x 支持自定义`v-model`修饰符：

```html
<div id="app">
  <my-component v-model.capitalize="myText"></my-component>
  {{ myText }}
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      myText: "",
    };
  },
});

app.component("my-component", {
  props: {
    modelValue: String,
    modelModifiers: {
      default: () => ({}),
    },
  },
  methods: {
    emitValue(e) {
      let value = e.target.value;
      if (this.modelModifiers.capitalize) {
        value = value.charAt(0).toUpperCase() + value.slice(1);
      }
      this.$emit("update:modelValue", value);
    },
  },
  template: `<input
    type="text"
    :value="modelValue"
    @input="emitValue">`,
});

app.mount("#app");
```

当`v-model`传入参数时，所产生的参数名称为`arg + Modifiers`

```html
<my-component v-model:foo.capitalize="bar"></my-component>
```

```js
app.component("my-component", {
  props: ["foo", "fooModifiers"],
  template: `
    <input type="text" 
      :value="foo"
      @input="$emit('update:foo', $event.target.value)">
  `,
  created() {
    console.log(this.fooModifiers); // { capitalize: true }
  },
});
```

## 迁移策略

- 将`.sync`替换为`v-model`:

  ```html
  <ChildComponent :title.sync="pageTitle" />

  <!-- to be replaced with -->
  <ChildComponent v-model:title="pageTitle" />
  ```

- 没有参数的`v-model`，要将参数和事件名称分别更改为`modelValue`和`update:modelValue`:

  ```html
  <ChildComponent v-model="pageTitle" />
  ```

  ```js
  // ChildComponent.vue

  export default {
    props: {
      modelValue: String, // previously was `value: String`
    },
    methods: {
      changePageTitle(title) {
        this.$emit("update:modelValue", title); // previously was `this.$emit('input', title)`
      },
    },
  };
  ```

# `key`属性

## 更新内容

- **NEW:** `key`在`v-if / v-else / v-else-if`分支上不再是必须的，Vue 会自动创建一个唯一的`key`。
  - **BREAKING:** 如果手动添加`key`，必须是唯一的，无法在故意的使用相同的`key`去强制重用分支。
- **BREAKING:** `<template v-for>`的`key`可以放置在`<template>`标签上，之前只能放置在子标签上。

## 条件分支

Vue3 推荐移除`key`

```html
<!-- Vue 2.x -->
<div v-if="condition" key="a">Yes</div>
<div v-else key="a">No</div>

<!-- Vue 3.x (recommended solution: remove keys) -->
<div v-if="condition">Yes</div>
<div v-else>No</div>

<!-- Vue 3.x (alternate solution: make sure the keys are always unique) -->
<div v-if="condition" key="a">Yes</div>
<div v-else key="b">No</div>
```

## `<template v-for>`

```html
<!-- Vue 2.x -->
<template v-for="item in list">
  <div :key="item.id">...</div>
  <span :key="item.id">...</span>
</template>

<!-- Vue 3.x -->
<template v-for="item in list" :key="item.id">
  <div>...</div>
  <span>...</span>
</template>
```

# `v-if`和`v-for`优先级

## 更新内容

- **BREAKING:** 如果在同一个元素上使用`v-if v-for`，`v-if`的优先级将高于`v-for`。

## 迁移策略

- 不推荐同时使用`v-if v-for`，会造成语法歧义
- 通过计算属性过滤掉不需渲染的元素

# `v-bind`合并行为

## 更新内容

- **BREAKING:** `v-bind`绑定的顺序会影响渲染的结果。

我们可以通过`v-bind="object"`绑定一个全是 attribute 的对象，但是当对象内存在和单独绑定的属性重复时，就会带来合并的问题。

## 3.x 语法

在 2.x 中单独绑定的属性总是会覆盖对象内的属性：

```html
<!-- template -->
<div id="red" v-bind="{ id: 'blue' }"></div>
<!-- result -->
<div id="red"></div>
```

而 3.x 中和绑定时顺序相关，后面的会覆盖前面的：

```html
<!-- template -->
<div id="red" v-bind="{ id: 'blue' }"></div>
<!-- result -->
<div id="blue"></div>

<!-- template -->
<div v-bind="{ id: 'blue' }" id="red"></div>
<!-- result -->
<div id="red"></div>
```

## 迁移策略

如果现有的项目依赖单个`v-bind`的覆盖能力，则要保证单个属性的`v-bind`在绑定对象之后。

# `v-for` Array Refs

在 Vue2 中，在`v-for`中使用`ref`时`$refs`会使用一个数组来存储，这样在存在`v-for`嵌套的情况下就会变得模棱不清且效率低下。

Vue3 中类似的情况就不会自动创建一个`$refs`数组，而是给`ref`绑定一个函数，使其更加的灵活。

```html
<div v-for="item in list" :ref="setItemRef"></div>
```

Options API

```js
export default {
  data() {
    return {
      itemRefs: [],
    };
  },
  methods: {
    setItemRef(el) {
      this.itemRefs.push(el);
    },
  },
  beforeUpdate() {
    this.itemRefs = [];
  },
  updated() {
    console.log(this.itemRefs);
  },
};
```

Composition API

```js
import { ref, onBeforeUpdate, onUpdated } from "vue";

export default {
  setup() {
    let itemRefs = [];
    const setItemRef = (el) => {
      itemRefs.push(el);
    };
    onBeforeUpdate(() => {
      itemRefs = [];
    });
    onUpdated(() => {
      console.log(itemRefs);
    });
    return {
      itemRefs,
      setItemRef,
    };
  },
};
```

**Note:**

- `itemRefs`可不必为数组，可以是对象或其他形式
- 如果需要，可是对`itemRefs`进行监听或者变成响应式
