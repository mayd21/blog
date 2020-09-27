---
title: Effective TypeScript学习笔记第二章
date: 2020-09-21 16:43:19
categories:
  - 前端
tags:
  - TypeScript
---

《Effective TypeScript》学习笔记第二章 TypeScript's Type System，了解类型系统，如何使用？如何做抉择？和需要避免的部分

<!-- more -->

# Type System

## 将类型视为一些值的集合

当`TypeScript`进行类型检查时，可以认为是判断该值是不是在类型所代表的集合之中，例如`number`类型就是所有数字值的集合。

最小的集合为空集合，即对应`never`类型。

第二小的集合为单一值集合，对应字面量类型：

```ts
type A = "A";
```

将`interface`视为对其类型域中值的描述

## 区分`Type Space or Value Space`

### 类型声明空间

`TypeScript`中一个什么可以存在以下一个或两个空间中：

- Type Space
- Value Space
  相同的名称可以引用不同的空间中的值：

```ts
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });

function calculateVolume(shape: unkown) {
  if (shape instanceof Cylinder) {
    console.log(shape.radius);
    //          ~~~~~~~~~~~~ Property 'radius' does not exist on type '{}'
  }
}
```

`instanceof`是 JavaScript 运行时的操作，所以无法获取 Type Space 中的`Cylinder`

TypeScript 所提供的声明只存在于 Type Space 中，仅在 TypeScript 提供的类型语法中有效，但是有些声明即可以在 Type Space 也可以在 Value Space，例如`class`和`enum`。

### 操作与`keywords`含义的区别

`typeof`

- 在`type`上下文中返回`TypeScript`的类型
- 在`value`上下文中返回`JavaScript`运行时的类型字符串

```ts
type T1 = typeof p; // Type is Person
const v1 = typeof p; // Value is "object"
```

除此之外还有`this; |; &; extends; const`等

## 采用类型声明而非类型断言

`TypeScript`有两种方式为一个值指定类型：

```ts
interface Person {
  name: string;
}

const alice: Person = { name: "Alice" };
const bob = { name: "Bob" } as Person;
```

类型声明会验证值是否符合接口，类型断言会忽视错误

```ts
const alice: Person = {};
//  ~~~~~~~~ Property 'name' is missing in type '{}'
//           but required in type 'Person'
const bob = {} as Person; // No Error
```

箭头函数中类型应用

```ts
const people = ["alice", "bob", "jab"].map((name) => ({ name }));
// 期望得到 Person[] 类型

// 断言方式 谁出现上述的问题
const people = ["alice", "bob", "jab"].map((name) => ({ name } as Person));

// 声明方式
const people = ["alice", "bob", "jab"].map((name) => {
  const person: Person = { name };
  return person;
});
const people = ["alice", "bob", "jab"].map(
  (name): Person => ({
    name;
  })
);
```

## 避免使用对象包装类型（String, Number, Booblean, Symbol, BigInt）

我么都知道，JavaScript 中字符串可以直接使用`String`对象上的方法，是因为 JavaScript 对字符串进行了对象包装：

```js
> 'primitive'.charAt(3)
"m"
```

在 TypeScript 中同样有原始类型和包装对象类型：

- string and String
- number and Number
- boolean and Boolean
- symbol and Symbol
- bigint and BigInt

采用对象包装类型看上去没什么问题：

```ts
function getStringLen(foo: String) {
  return foo.length;
}

getStringLen("hello"); // OK
getStringLen(new String("hello")); // OK
```

但是它和原始的`string`类型不兼容：

```ts
function isGreeting(phrase: String) {
  return ["hello", "good day"].includes(phrase);
  // ~~~~~~~
  // Argument of type 'String' is not assignable to parameter
  // of type 'string'.
  // 'string' is a primitive, but 'String' is a wrapper object;
  // prefer using 'string' when possible
}
```

由此可见`string`是可以赋值给`String`，反之则不可以。

## 认识到额外属性检查的局限性

```ts
interface Room {
  numDoors: number;
  ceilHeightFt: number;
}

const r: Room = {
  numDoors: 1,
  ceilHeightFt: 10,
  elepant: "persent", // Error
};
const obj = {
  numDoors: 1,
  ceilHeightFt: 10,
  elepant: "persent",
};
// Room 属于 obj 的子级
const r: Room = obj; // OK
r.elepent; // Error
```

## type VS interface

### 相似点

- 都可以使用 index 索引
- 都可以定义函数
- 都支持泛型
- `interface`可以`extends type`；`type`也可以`& interface`
- 都可以被`class implement`

### 不同点

- `type`有联合类型`|`， `interface`没有
- `type`比`interface`能力更强，`type`不仅可以使用联合类型，还可以使用更高级例如映射类型和条件类型
- `interface`可以声明合并

```ts
interface IState {
  name: string;
  capital: string;
}
interface IState {
  population: number;
}
const wyoming: IState = {
  name: "Wyoming",
  capital: "Cheyenne",
  population: 5000, // OK
};
```

## 使用类型操作和声明避免违反 DRY

- 采用类型声明避免同类型函数声明重复
- 采用`extends`继承公共类型
- 活用`Partial<T> Pick<T, K>`等`Utility Types`
