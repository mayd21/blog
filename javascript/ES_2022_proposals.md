# ES 2022 新特性都有什么？🤔

依据 [tc39/proposals](https://github.com/tc39/proposals/blob/main/finished-proposals.md) ES2022 新增了 8 个新特性，我们一起来看一下具体是哪些新特性。

## Class 新特性

历尽千辛，私有 Class 的提案终于通过了，在此之前我们只能从编码规范上约束私有成员，然而其实际上还是可以访问的。使用过 TypeScript 的应该知道，TypeScript 提供了 `private` 修饰符，但是其也不是真正的“私有”。本次提案通过了在 Class 下定义私有方法和属性已经私有的静态方法和属性四类。

### 实例属性的新写法

在 ES6 的 Class 要声明类的实例属性必须在 `constructor` 内，并且必须进行初始化，而 ES2022 为其带来了新的写法

> ES6 属性声明方式

```javascript
class MyClass {
  constructor() {
    // 声明属性并初始化
    this.field = 'field';
  }

  check() {
    // output => field
    console.log(this.field);
  }
}
```

> ES2022 属性声明方式

```javascript
class MyClass {
  // 直接在 constructor 外声明，并且可以不进行初始化
  field

  check() {
    // output => undefined
    console.log(this.field);
  }
}
```

这种写法的好处是首先属性声明时可以不进行初始化，其次所有实例自身的属性都定义在类的头部，类的定义更加整洁。

> ***注意*** 新写法定义的属性是实例自身的属性，而不是定义在实例对象的原型上

### 私有实例方法和属性

通过 `#` 前缀来声明私有的属性和方法，包括静态私有成员方法和属性的声明。

```javascript
class MyClass {
  // 通过 # 定义私有属性和私有方法
  #privateField = 'private field';
  #privateMethod() {
    console.log(this.#privateField);
  }

  // 静态私有属性和方法的定义
  static #PRIVATE_STATIC_FIELD = 'PRIVATE STATIC FIELD';
  static #privateStaticMethod() {
    console.log(this.#PRIVATE_STATIC_FIELD);
  }

  checkPrivate() {
    this.#privateMethod();
    MyClass.#privateStaticMethod();
  }
}

const myClass = new MyClass();
// output => private field PRIVATE STATIC FIELD
myClass.checkPrivate();
// SyntaxError: Private field '#privateMethod' must be declared in an enclosing class
myClass.#privateMethod();
```

### 静态初始化块

ES2022 给 Class `static` 块提供了一种机制，可以在类定义时执行额外的静态初始化过程。

#### 语法示例

```javascript
class MyClass {
  static {
    // statements
  }
}
```

#### 为什么需要？

- 目前的静态字段和静态私有字段是可以在类的定义阶段进行初始化的，但是当我们需要在初始化时执行一些语句时，我们就只能将这一部分的初始化逻辑放到类的定义之外。
  
  ```javascript
  // 没有静态块
  class MyClass {
    // x 为默写原始数据，需要经过处理
    static x = {};
    static y;
    static z;
  }
  
  try {
    // 处理 x 后给 y 和 z 进行初始化
    const obj = doSomethingWith(MyClass.x);
    MyClass.y = obj.y;
    MyClass.z = obj.z;
  } catch {
    // 异常是的初始化逻辑
    MyClass.y = ...;
    MyClass.z = ...;
  }
  ```
  
  ```javascript
  // 采用静态块
  class MyClass {
      static x = {};
      static y;
      static z;
      // 初始化逻辑可以直接写在类的定义
      static {
          try {
              // 处理 x 后给 y 和 z 进行初始化
              const obj = doSomethingWith(MyClass.x);
              MyClass.y = obj.y;
              MyClass.z = obj.z;
          } catch {
              // 异常是的初始化逻辑
              MyClass.y = ...;
              MyClass.z = ...;
          }
      }
  }
  ```

- 当我们需要将某个类的私有属性的访问权限共享给在同一范围内声明的其他类和函数时，可以通过静态块赋予其访问私有属性的特权。
  
  ```javascript
  let operator
  class MyClass {
    #x = 'x';
    static {
      operator = {
        getX(obj) { return obj.#x },
        setX(obj, value) { obj.#x = value }
      }
    }
  }
  
  const myClass = new MyClass()
  getX(myClass); // x
  setX(myClass, 'new x'); // set myClass.#x = 'new x'
  ```

### 私有属性检查

当我们尝试去访问一个私有属性时会抛出一个异常，这样本身没有什么问题。但是我们经常需要检查一个对象上是否有某个私有属性，从而针对检查的结果做一些处理。

- 使用`try/catch`来实现，可以，但没必要
  
  ```javascript
  class MyClass {
    #field;
  
    static isMyClass(obj) {
      try {
        obj.#field;
        return true;
      } catch {
        return false;
      }
    }
  }
  ```

- 提供一种简单的方案返回一个布尔值来表明一个私有属性是否存在，使用关键字`in`
  
  ```javascript
  class MyClass {
    #field;
    #method() {}
  
    static isMyClass(obj) {
      return #field in obj && #method in obj;
    }
  }
  ```

## RegExp Match Indices

在正则表达式中添加标志`/d`，当使用`exec()`方法后会产生匹配对象，记录每组捕获的开始和结束。

```javascript
const re1 = /a+(?<Z>z)?/d;

const s1 = "xaaaz";
const m1 = re1.exec(s1);
// indices 记录了匹配的起始索引
s1.slice(...m1.indices[0]) === "aaaz";
/**
* m1 除了之前已有的结果外，会有一个新的属性 indices 来记录每组的开始和结束索引
* 同时，具名 group 中也会记录当前 group 的起始索引
* m1 =>  [ 'aaaz', 'z', index: 1, input: 'xaaaz', 
*        indices: [[1,5], [4,5],
*        groups: {Z: [4,5]}
*    ]
*/
```

## 顶层 await

### 现在的 await 存在哪些问题？

一直以来`await`都必须在`async`中才能使用，但是当一个`module`中依赖异步获取的资源时，我们`import`该模块时是立即执行的，这就导致我们获取不到异步依赖的资源。看一下下面的例子：

```javascript
// 依赖异步资源
// awaiting.js
import {  process } from './some-modules.js';
let output;
async function main() {
  const dynamic = await import(computedModuleSpecifier);
  const data = await fetch(url);
  // output 依赖异步数据
  output = process(dynamic.default, data);
}
// 执行处理函数
// 当然此处 main 方法也可以改造为自执行函数
main();
export { output };
```

这种方式如果在加载模块后立即访问，是会出输出 `undefined` 的，因为异步任务并未执行完成，并未完成对`output`的赋值。

```javascript
// usage.js
// 使用 awaiting.js
import { output } from './awaiting.js';
export function outputPlusValue(value) {
  return output + value;
}
// NaN, 此处 output = undefined
console.log(outputPlusValue(100));
// 若 1s 后异步任务处理完，才可能符合预期
setTimeout(() => console.log(outputPlusValue(100), 1000);
```

### 有什么解决方案？

有什么解决方案呢？是有的，我们可以导出一个`Promise`来保证`output`的初始化完成。

```javascript
// awaiting.js
import { process } from "./some-module.js";
let output;
export default (async () => {
  const dynamic = await import(computedModuleSpecifier);
  const data = await fetch(url);
  output = process(dynamic.default, data);
})();
export { output };
```

然后当我们需要使用时，就需要按照下面的方式进行使用：

```javascript
// usage.js
import promise, { output } from "./awaiting.js";
export function outputPlusValue(value) { return output + value }

promise.then(() => {
  // 在异步的结束后访问 output
  console.log(outputPlusValue(100));
  setTimeout(() => console.log(outputPlusValue(100), 1000);
});
```

然而，这种方式也会存在很多问题：

- 每个人都必须按照这种特定的协议来加载模块，必须要以这种正确的方式`await`一个`Promise`

- 万一你忘记严格按照这种方式，那么就不能保证读取模块时异步任务是否执行完毕

- 若是一个深层引用的模块，那么这种`Promise`就要层层穿透到每一步

### 有什么方法避免忘记 await

为了尽量避免在使用时忘记`await`依赖的`Promise`执行完完毕，可以将依赖异步的导出内容统一放到`Promise`的`resolve`中

```javascript
// awaiting.js
import { process } from "./some-module.mjs";
export default (async () => {
  const dynamic = await import(computedModuleSpecifier);
  const data = await fetch(url);
  const output = process(dynamic.default, data);
  // 将 output 以 resolve 的形式通过 promise 返回
  return { output };
})();

// usage.mjs
import promise from "./awaiting.mjs";
// 引用时则只能从 promise.then 中引用
export default promise.then(({output}) => {
  function outputPlusValue(value) { return output + value }

  console.log(outputPlusValue(100));
  setTimeout(() => console.log(outputPlusValue(100), 1000);
  return { outputPlusValue };
});
```

虽然这种方式能够解决一部分问题，但是并不是理想的方式。这就需要所有异步的模块都必须要放在`.then()`的回调内来实现动态导入，这种方式是有悖于 ES Module 的设计模式的。

### 解决方案：顶层 await

顶层 `await`是指`await`可以不再依赖`async`函数而直接使用，例如上面例子使用顶层`await`即可简化如下：

```javascript
// awaiting.js
import { process } from "./some-module.js";
const dynamic = import(computedModuleSpecifier);
const data = fetch(url);
// 直接使用 await 等待异步数据
export const output = process((await dynamic).default, await data);

// usage.js
import { output } from "./awaiting.js";
// 引入时无需关心是否是动态导入
export function outputPlusValue(value) { return output + value }

// 预期一致
console.log(outputPlusValue(100));
setTimeout(() => console.log(outputPlusValue(100), 1000);
```

#### 有哪些应用场景？

- 动态模块加载
  
  ```javascript
  const strings = await import(`/i18n/${navigator.language}`);
  ```

- 资源初始化
  
  ```javascript
  const connection = await dbConnector()
  ```

- 依赖回退
  
  ```javascript
  let jQuery;
  try {
    jQuery = await import('https://cdn-a.com/jQuery');
  } catch {
    jQuery = await import('https://cdn-b.com/jQuery');
  }
  ```

#### await 如何阻断代码执行的？

- 一个模块需要等待它依赖的所有模块全部执行完毕才会执行
  
  ```javascript
  import { a } from './a.js';
  import { b } from './b.js';
  import { c } from './c.js';
  // 需等待 a b c 全部执行完毕才会执行下面的语句
  // 若包含异步任务，需等待异步任务执行完
  console.log(a, b, c);
  ```

- 顶层`await`不会阻止相邻模块的执行
  
  ```javascript
  // x.js
  console.log("X1");
  await new Promise(r => setTimeout(r, 1000));
  console.log("X2");
  
  // y.js
  console.log("Y");
  
  // z.js
  import "./x.mjs";
  import "./y.mjs";
  
  // 输出结果为 X1 Y X2
  ```

## 新增 .at() 方法

为所有基本可索引类(`Array`、`String`、`TypedArray`)新增一个`.at()`方法。可以通过指定的索引读取数据，并且支持复数索引来从末端开始读取数据。

```javascript
[1,2,3,4,5].at(3)  // returns 4

[1,2,3,4,5].at(-2)   // returns 4
```

## 新增 Object.hasOwn() 方法

在此之前我们可以使用`Object.prototype.hasOwnProperty()`来判断对象是否拥有某个属性。但是不建议直接使用对象原型链上的`hasOwnProperty()`来判断，因为`Object.prototype`有时是不可访问或者是被重新定义了的。

- `Object.prototype`不可访问
  
  ```javascript
  // 通过 Object.create(null) 创建的对象并不是从 Object.prototype 继承
  const obj = Object.create(null)
  // Error
  obj.hasOwnProperty('foo')
  ```

- `hasOwnPrototype`被重定义
  
  ```js
  let obj = {
    hasOwnProperty() {
      // statements
    }
  }
  
  // 不符合预期
  obj.hasOwnProperty('foo')
  ```

因此，通常情况下我们不建议直接使用原型链上的方法。

```javascript
const obj = { foo: 'f' }
// 不推荐方式
obj.hasOwnProperty('foo')

// 推荐方式
let hasOwnProperty = Object.prototype.hasOwnProperty
if (hasOwnProperty.call(obj, 'foo')) {
  console.log('has property foo')
}
```

`Object.hasOwn(object, property)`其结果和`Object.prototype.hasOwnProperty.call(object, property)`是一致的，但是它适用于所有的对象类型，使用更加方便。

```js
let object = { foo: false }
Object.hasOwn(object, "foo") // true

let object2 = Object.create({ foo: true })
Object.hasOwn(object2, "foo") // false

let object3 = Object.create(null)
Object.hasOwn(object3, "foo") // false
```

## Error Cause

`Error`支持指定其错误的原因，在构建`Error`时，支持传入`cause`属性，这样对一些较深层次的嵌套结构时，可以更好地串联错误发生的原因。

```js
async function doJob() {
  const rawResource = await fetch('//domain/resource-a')
    .catch(err => {
      throw new Error('Download raw resource failed', { cause: err });
    });
  const jobResult = doComputationalHeavyJob(rawResource);
  await fetch('//domain/upload', { method: 'POST', body: jobResult })
    .catch(err => {
      throw new Error('Upload job result failed', { cause: err });
    });
}

try {
  await doJob();
} catch (e) {
  console.log(e);
  console.log('Caused by', e.cause);
}
// Error: Upload job result failed
// Caused by TypeError: Failed to fetch
```
