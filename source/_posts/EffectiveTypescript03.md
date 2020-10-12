---
title: Effective TypeScript学习笔记第三章
date: 2020-09-29 18:20:10
categories:
  - 前端
tags:
  - TypeScript
---

本章主要介绍类型推断存在的一些问题和如何修复它们，便于我们理解 TypeScript 是如何推断类型的。

<!-- more -->

# 使用推断类型来避免代码的杂乱

## 当 TypeScript 可以进行类型推断时尽量不写类型注解

对于刚从 JavaScript 转向 TypeScript 的用户来说，第一件事就是将所有的代码都加上类型注解，但是有时类型注解是不需要的，并且会使得代码风格变得糟糕。

```ts
// 避免这样写
let x: number = 1;
// 采用类型推断
let x: = 1; // Type is number
```

TypeScript 同样可以推断复杂对象：

```ts
const person: {
  name: string;
  born: {
    where: string;
    when: string;
  };
} = {
  name: "Jack",
  born: {
    where: "UK",
    when: "Nov. 26",
  },
};
```

可直接写为：

```ts
const person = {
  name: "Jack",
  born: {
    where: "UK",
    when: "Nov. 26",
  },
};
```

适用于对象的同样适用于数组，TypeScript 可以根据函数的参数和操作推断出其返回值的类型：

```ts
function square(nums: number[]) {
  return nums.map((x) => x * x);
}
const squares = square([1, 2, 3, 4]); // Type is number[]
```

TypeScript 通常会推断出比你期待的类型更将精确：

```ts
const axis1: string = "x"; // Type is string
const axis2 = "y"; // Type is "y"
```

## 使用类型推断更便于重构

```ts
interface Product {
  id: number;
  name: string;
  price: number;
}
// 对函数内的声明都进行类型注解
function logProduct(product: Product) {
  const id: number = product.id;
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

当由于一些原因需要将 ID 的类型修改为`string`时，上述的`logProduct`函数内部的 `id` 类型注解就会报错，取消函数内的类型注解，便可以避免修改：

```ts
function logProduct(product: Product) {
  const { id, name, price } = product;
  console.log(id, name, price);
}
```

## 函数参数的类型注解

TypeScript 一般情况下变量的类型确定是在变量对一次声明的时候，无法在使用参数时去推断其类型，因此函数的参数无法在使用时进行类型推断。理想的方式是对函数的签名进行类型声明，但是函数内部的本地变量采用类型推断。

当函数作为一个有类型声明的库中的回调函数时，其参数的类型是可以被推断的，例如当我们使用一个带有类型声明的`HTTP`库时，其回调函数中`request`和`response`的类型声明是不需要的：

```ts
// Don't do this:
app.get("/health", (request: express.Request, response: express.Response) => {
  response.send("OK");
});

// Do this:
app.get("/health", (request, response) => {
  response.send("OK");
});
```

## 即使可以推断类型仍需要声明的情况

- 当定义一个对象字面量的类型是，可以对对象的属性进行检查
- 期望在定义的时候检查错误，而不是在使用的时候才检查
- 为函数声明返回类型

  ```ts
  // 类型推断为 Promise
  function getQuote(ticker: string) {
    return fetch(`https://quotes.example.com/?q=${ticker}`).then((response) =>
      response.json()
    );
  }
  // 决定增加缓存
  // 类型推断为 Promise | number
  const cache: { [ticker: string]: number } = {};
  function getQuote(ticker: string) {
    if (ticker in cache) {
      return cache[ticker];
    }
    return fetch(`https://quotes.example.com/?q=${ticker}`)
      .then((response) => response.json())
      .then((quote) => {
        cache[ticker] = quote;
        return quote;
      });
  }
  // 当再次调用时会报错
  getQuote("MSFT").then(considerBuying);

  // 为函数添加返回类型声明可以防止实现的错误变现在代码中
  // 并且可以更加清楚的函数
  const cache: { [ticker: string]: number } = {};
  function getQuote(ticker: string): Promise<number> {
    if (ticker in cache) {
      return cache[ticker];
      // ~~~~~~~~~~~~~ Type 'number' is not assignable to 'Promise<number>'
    }
    // ...
  }
  ```

## 如果想使用一个已命名的类型时，添加类型声明

如下，你可能会选择类型推断而不写函数返回值类型声明：

```ts
interface Vector2D {
  x: number;
  y: number;
}
// 类型推断为： { x: number; y: number; }
// 虽然该类型和 Vector2D兼容，但是对于使用者来说更期待是的是 Vector2D，而非 { x: number; y: number; }
function add(a: Vector2D, b: Vector2D) {
  return { x: a.x + b.x, y: a.y + b.y };
}
```
