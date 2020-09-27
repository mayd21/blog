---
title: Effective TypeScript学习笔记第一章
date: 2020-09-18 16:02:17
categories:
  - 前端
tags:
  - TypeScript
---

《Effective TypeScript》学习笔记第一章 Getting to know TypeScript，了解 TypeScript 的基础和应用 TypeScript 原则

<!-- more -->

# TypeScript 基础

## 代码的生成独立于类型

### 不能在`Runtime`检查`TypeScript`类型

```ts
interface Square {
  width: number;
}
interface Rectangle extend Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    // ~~~~~~~~~~~~ 'Rectangle' only refers a type,
    //              but is being used as a value here
    return shape.width * shape.height;
    // ~~~~~~~~~~~~ Property 'height' does not exist
    //              on type 'Shape'
  } else {
    return shape.width * shape.width;
  }
}
```

`instanceof`检查是在运行时进行，而`Rectangle`是`TypeScript`的类型，它不会影响运行时的行为。`TypeScript`的类型定义和声明会在编译为`JavaScript`后全部移除。

通过检查相关属性确定类型

```ts
function calculateArea(shape: Shape) {
  if ("height" in shape) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

`class`可以作为类型，因此采用`class`作为类型可以使用`instanceof`进行判断

```ts
class Square {
  constructor(public width: number) {}
}
class Rectangle {
  constructor(public width: number, height: number) {}
}
type Shape = Square | Rectangle;
function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

### 类型操作无法影响运行时的值

例如当将一个可能为`string`的值转化为`number`时：

```ts
function asNumber(value: number | string): number {
  return value as number;
}
```

编译后的`JS`并未改变`value`的值：

```js
function asNumber(value) {
  return value;
}
```

正确写法：

```ts
function asNumber(value: number | string): number {
  return typeof value === "string" ? Number(value) : value;
}
```

## 避免使用`any`类型

- `any`导致类型不安全
- `any`会打破类型约定，导致接口类型定义的约束失效
- 代码补全和提示等语言服务能力丢失
- 重构代码是会隐藏 bug，例如接口的改变导致无法及时发现引用处的错误
- `any`会隐藏你的类型设计

{% post_link EffectiveTypescript02 Effective TypeScript学习笔记第二章 %}
