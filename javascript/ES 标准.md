# ES 标准

一直以来经常听说 ES6 ES7 等相关的 JavaScript 标准，但是对于其标准的具体包含内容和版本的更新流程以及每个版本的具体新增的特性并没有认真的了解过。因此本次打算首先从 ECMAScript 标准入手，了解 ECMA 标准规范具体包含的内容和标准转化过程。并对各个 ECMAScript 所涉及的新的 JavaScript 特性进行一个介绍。

## JavaScript 与 ECMAScript

我们都知道 JavaScript 语言的核心是 ECMAScript，它是由 [ECMA TC39](https://tc39.es/) 委员会标准化的语言。ECMA(European Computer Manufacturers Association) 包含多个规范，而 ECMAScript ECMA-262 规范，JavaScript 则是符合这一规范的程序语言，但是 JavaScript 还提供了超出该规范的一些特性。ECMA TC39 相关标准包括：

- [ECMA-262 ECMAScript](https://tc39.es/ecma262/)

- [ECMA-402 国际化 API](https://tc39.es/ecma402/)

- [ECMA-404 JSON](https://www.ecma-international.org/publications-and-standards/standards/ecma-404/)

- [ECMA-414 ECMAScript 规范套件](https://www.ecma-international.org/publications-and-standards/standards/ecma-414/)

### 标准化流程

ECMAScript 版本每年都由 ECMA 大会批准并作为标准发布，其所有的进展都在 [Ecma TC39 · GitHub](https://github.com/tc39) 上公开。

在 ES 第六版即 ES6 之前通常由其主要版本号代替，例如 ES6、ES5 等，在 ES6 之后规范的发布就以年份进行命名，如 ES2017、ES2018 等。这即是目前对于 ES 版本表述的既有按照年份的表示也有按照版本代号的形式，ES6 即为 ES2015。

### 哪些内容会被纳入 ECMAScript 中？

- 语法（解析规则、关键词、流程控制、对象初始化，等等）

- 错误处理机制（[`throw`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/throw)、[`try...catch`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/try...catch)，以及创建用户自定义[错误](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)类型的能力）

- 类型（布尔值、数字、字符串、函数、对象，等等）

- 基于原型的继承机制

- 内置对象和函数（[`JSON`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON)、[`Math`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math)、[`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array)  方法、[`parseInt`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt)、[`decodeURI`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/decodeURI)，等等）

- [严格模式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Strict_mode)

- [模块系统](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules)

- 基本内存模型

## TC39 提案

### 提案流程

- Stage 0 (Strawperson)：草案阶段，没有作为真实提案的任何想法

- Stage 1 (Proposal)：补充理由，描述解决方案、用法的说明范例等。在该阶段就会有 polyfill 或者 demo

- Stage 2 (Draft)：用规范的方式来精确的描述语法和语义，进入到该阶段就很可能成为规范

- Stage 3 (Candidate)：已完成规范的内容，但是需要大量的使用和反馈才能进入下一阶段

- Stage 4 (Finished)：已经完成的提案，正式加 ECMAScript 标准中

提案和标准的完成还需要浏览器或者 Node.js 等运行环境实现，但是有些提案还没有到 Stage 4 就已经可以在浏览器中使用，或者有对应的 polyfill 可以使用。

### 如何查看提案？

各个提案目前在哪个阶段？是否被拒绝或者已经完成，都可以在 [tc39/proposals: Tracking ECMAScript Proposals](https://github.com/tc39/proposals)  中查看，其中提供了处在各个阶段中的提案详情。

### ECMAScript 历次规范哪里看？

ECMA-262 的最新规范和历史规范可以在 [ECMA-262 - Ecma International](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/) 查看，其中包含了最新草案及历史规范文档。

## 查看浏览器和 Node.js 的兼容情况

不同的浏览器对标准的支持程度各不相同，如何查看某个特性是否得到了支持，在哪个版本得到了支持？

- [ECMAScript 6 compatibility table](https://kangax.github.io/compat-table/es6/)：ECMA 标准兼容性表格，内部列举了不同版本标准在浏览器、Node.js等环境下的兼容情况

- [Node.js ES2015/ES6, ES2016 and ES2017 support](https://node.green/)：若你只想关注 Node.js 中的兼容情况，可以查看该表格

- [Can I use](https://caniuse.com/)：想要便捷的查询某个 API 在浏览器中的兼容情况？该网站支持直接搜索某个 API 在各个浏览器中的兼容信息。不仅针对 JavaScript，同时可以查询浏览器相关的其他特性的兼容情况，如 CSS、HTML、SVG 等。
