# JS 事件循环一次说清

## 从单线程说起

我们都知道 JavaScript 是单线程语言，那为什么 JavaScript 不支持多线程呢，这是因为 JavaScript 本身设计之初就是按照单线程设计的。JavaScript 设计之初的目的是在浏览器端进行运行，用于处理页面操作的也就是操作DOM，因此他的用途就决定了的他是单线程的语言，因为多线程会带来线程间同步的问题。



## 宏任务与微任务

JavaScript 是单线程语言，这也就意味着所有的任务都需要排队，一个一个按照顺序执行。但是当遇到很慢的 IO 设备时，例如网络请求等，后续的任务就需要等待网络请求任务完成才能继续执行，这就会导致 CPU 处于空闲的挂起状态，因此为了不阻塞后续的任务，任务又分为同步任务和异步任务两种。同步任务在主线程进行依次执行，而一步任务会进入*任务队列* 中，任务队列中的任务不会阻塞主线程的执行，只有当任务队列内的异步任务可以执行时才会加入到主线程进行执行。



同步任务和异步任务的执行机制简单如下：

1. 同步任务会直接在主线程中依次执行

2. 异步任务会加入到任务队列中

3. 当主线程中同步任务执行然后会读取任务队列执行任务队列内的任务

4. 重复上面的循环



常见宏任务：

- script

- setTimeout, setInterval 等定时器

- 事件触发的回调函数，如 DOM 事件

常见微任务：

- Promise

- MutationObserver



## 事件循环

主线程同步任务执行完，读取任务队列执行异步任务，不停的循环这个过程就是*事件循环* 。在 Node.js 环境中和浏览器环境中，事件循环执行的顺序有一些差别：

- Node.js 独有的 ```process.nextTick```  微任务总是最先执行，之后才会执行其他的微任务

- 在 Node.js 11.x 之前每次都是执行完所有的宏任务后才会执行各个宏任务的微任务。而在 11.x 之后，则为当一个宏任务执行完后则会立即执行其相关的微任务，这样的改动和目前浏览器的执行机制保持一致。



## 实际案例

讲了这么多实际理解起来还是比较抽象，我们来看几个例子：

### 例子一

```js
console.log("1");

setTimeout(function () {
  console.log("2");
}, 0);

new Promise((resolve) => {
  console.log("3");
  resolve("4");
})
  .then((res) => {
    console.log(res);
  })
  .then(() => {
    console.log("5");
  });

Promise.resolve()
  .then(() => {
    console.log("6");
  })
  .then(() => {
    console.log("7");
  });

console.log("8");

// 执行结果：1 3 8 4 6 5 7 2 
```

解析：

- 首先在第一次宏任务中会先打印出 1 2 8，因为```Promise``` 的构造函数属于是同步任务，因此会先打印出来。

- 第一次的宏任务后会执行微任务，其中包括第一个的 ```Promise``` 的第一个 ```then``` 4 和第二个```Pormise``` 的第一个 ```then``` 6，因为```then``` 函数本身会放到下一次微任务中执行，因此第二次执行的 ```then``` 会放到下次微任务中执行而延后。

- 紧接着第一次微任务，会继续执行后续的微任务，即 5 和 7。

- 最后执行由于 ```setTimeout``` 回调产生的第二次宏任务，打印出 2。

### 

### 例子二

```js
console.log(1);

setTimeout(() => {
  console.log(2);
  process.nextTick(() => {
    console.log(3);
  });
  new Promise((resolve) => {
    console.log(4);
    resolve();
  }).then(() => {
    console.log(5);
  });
});

new Promise((resolve) => {
  console.log(7);
  resolve();
}).then(() => {
  console.log(8);
});

process.nextTick(() => {
  console.log(6);
});

setTimeout(() => {
  console.log(9);
  process.nextTick(() => {
    console.log(10);
  });
  new Promise((resolve) => {
    console.log(11);
    resolve();
  }).then(() => {
    console.log(12);
  });
});

// 执行结果：1 7 6 8 2 4 3 5 9 11 10 12
```

解析：

- 首先第一次宏任务会先打印出 1 7。

- 紧接着进入微任务，```process.nextTick``` 是Node.js中特有的，他会在微任务中优先执行，因此，当前微任务输出的顺序是 6 8，而非 8 6。

- 之后进入第二次宏任务，为 ```setTimeout``` 的回调函数，第一个宏任务优先输出 2 4。宏任务执行完后会立即执行其对应的微任务，输出 3 5。

- 之后为第二个宏任务，其逻辑与上一个宏任务一致，先输出 9 11，后再执行其相关的微任务，输出 10 12。
