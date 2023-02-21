# ES 标准

一直以来经常听说 ES6 ES7 等相关的 JavaScript 标准，但是对于其标准的具体包含内容和版本的更新流程以及每个版本的具体新增的特性并没有认真的了解过。因此本次打算首先从 ECMAScript 标准入手，了解 ECMA 标准规范具体包含的内容和表转化过程。并对各个 ECMAScript 所涉及的新的 JavaScript 特性进行一个介绍。

## JavaScript 与 ECMAScript

我们都知道 JavaScript 语言的核心是 ECMAScript，它是由 [ECMA TC39](https://tc39.es/) 委员会标准化的语言。ECMA 包含对个规范，指定 JavaScript 语言规范的只是其中一个标准，相关标准包括：

- ECMA-262 ECMAScript

- ECMA-402 国际化 API

- ECMA-404 JSON

- ECMA-414 ECMAScript 规范套件

### 标准化流程

ECMAScript 版本每年都有 ECMA 大会批准并作为标准发布，其所有的进展都在 [Ecma TC39 · GitHub](https://github.com/tc39) 上公开。

在 ES 第六版即 ES6 之前通常由其主要版本号代替，例如 ES6、ES5 等，在 ES6 之后规范的发布就一年份进行命名，如 ES2017、ES2018 等。这即是目前对于 ES 版本表述的既有按照年份的表示也有按照版本代号的形式，ES6 即为 ES2015。

### 哪些内容会被纳入 ECMAScript 中？

- 语法（解析规则、关键词、流程控制、对象初始化，等等）

- 错误处理机制（[`throw`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/throw)、[`try...catch`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)，以及创建用户自定义[错误](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)类型的能力）

- 类型（布尔值、数字、字符串、函数、对象，等等）

- 基于原型的继承机制

- 内置对象和函数（[`JSON`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON)、[`Math`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math)、[`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)  方法、[`parseInt`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt)、[`decodeURI`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/decodeURI)，等等）

- [严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)

- [模块系统](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules)

- 基本内存模型
