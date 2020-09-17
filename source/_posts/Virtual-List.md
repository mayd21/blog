---
title: 虚拟列表设计
date: 2020-09-16 18:18:45
tags:
  - web
---

## 方案设计

实现虚拟列表，是因为在列表数据量过大的情况下会渲染大量的 DOM 节点，造成渲染压力的增大，带来较差的浏览体验。虚拟列表可以控制 DOM 节点的属数量，在实现无限滚动的前提下降低渲染的压力，提升浏览体验。

<!-- more -->

设计方案主要包括一下四个方面：

- 计算出可视范围内需要展示的数据
- 从 list 中截取对应的数据进行渲染
- 计算上下留白的高度使用空 div 节点填充，实现与实际列表一样的滚动展现
- 监听滚动事件，在滚动到底部后加载新的数据

基于上述的设计，如下图所示为虚拟列表滚动时 DOM 的状态示意图，依据图示需要计算以下一些变量：

```ts
const reserveTop = 3; // 顶部部预留个数
const reserveButtom = 3; // 底部预留个数
const itemHeight = 300; // 每个item大致的高度
let totalItemCount; // 目前总item数量
let scrollTop; // 滚动容器超过可是区域的顶部高度
const clientHeight; // 可是区域高度
// visibleCount为可视区域内能够展示的数据个数
const visibleCount = Math.ceil(clientHeight / itemHeight);
// topHeight为顶部留白的高度
let topHeight = scrollHeight - itemHeight * reserveTop;
// start为需要展示的数据在list中的起点坐标
let start = Math.floor(topHeight / itemheight);
// buttomHeight为底部留白的高度
let buttomHeight = (totalItemCount - end - 1) * itemHeight;
// end为需要展示的数据在list中的终点坐标
let end = start + visibleCount + reserveButtom;
```

![virtual-list.png](https://i.loli.net/2020/09/16/MnXThF6fQpwtzYm.png)

## 实现过程

### 计算逻辑

依据上述的设计方案，需要在滚动的过程中不断计算相关的数据对列表中展示的数据以及上下留白的高度进行更新，滚动事件内部执行逻辑如下图所示：
![on-scroll-virtual-list.png](https://i.loli.net/2020/09/16/IpUgCFLh4mnuB7O.png)

- 通过判断是否滚到到底部，然后调取服务分页加载数据实现无限滚动
- 通过判断前后`start`  与`end`是否变化避免页面的抖动，原因是因为即使`start`  与`end`没有变化，但是`topHeight`  与`buttomHeight`的计算会不同，更新上下留白的高度会出发`onScroll`滚动事件造成无限循环。

### 代码实现

核心实现代码如下：

```ts
/**
 * 更新虚拟列表
 * @method updateVisibleList
 */
updateVisibleList() {
  const dom = this.listDom;
  if (!dom) return;
  // 获取滚动的当前状态
  const scrollTop = dom.scrollTop || 0;
  const clientH = dom.clientHeight || 0;
  // topHeight为列表上面的虚拟高度
  const topHeight = scrollTop - itemHeight * reserveTop;
  // 就算虚拟列表展示的起始和终止事件对应的坐标
  let start = Math.floor(topHeight / itemHeight);
  start = start < 0 ? 0 : start;
  const visibleCount = Math.ceil(clientH / itemHeight);
  const end = start + visibleCount + reserveBottom;

  // start end 未变化则不更新，避免抖动
  const { start: oldStart, end: oldEnd } = this.state;
  if (start === oldStart && end === oldEnd) return;
  // bottomHeight为列表以下的虚拟高度
  const bottomHeight = (this.state.currentCount - 1 - end) * itemHeight;
  this.setState({ start, end, topHeight, bottomHeight });
}
/**
 * 滚动处理函数
 * @method handleOnScroll
 */
handleOnScroll() {
  const dom = this.listDom;
  if (!dom) return;
  const scrollH = dom.scrollHeight || 0;
  const scrollT = dom.scrollTop || 0;
  const clientH = dom.clientHeight || 0;
  this.updateVisibleList();
  // 触及底部 加载新数据
  if (clientH + scrollT >= scrollH) {
    queryParam.offset = this.state.currentCount;
    this.getList(queryParam, false);
  }
}
```

### 列表渲染方式

```tsx
render() {
  const list = events.slice(start, end);
  return (
    <div>
      <div style={{ height: start === 0 ? 0 : topHeight }} />
      {list.map(event => (
        <Item key={event.id} event={event} />
      ))}
      <div style={{ height: bottomHeight }} />
    </div>
  );
}
```
