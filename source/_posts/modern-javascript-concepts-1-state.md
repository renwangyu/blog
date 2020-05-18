---
title: 【译】现代javascript概念术语part1——状态(state)
date: 2020-05-18 21:16:49
tags: 
- javascript
categories:
- javascript
top_img:
---

[阅读原文](https://auth0.com/blog/glossary-of-modern-javascript-concepts/)

## 状态
**状态**是指程序在某个时间点可以访问和操作的信息，包括存储在存储器(操作系统、输入/输出、数据库等)的数据。举个例子，在任何给定时刻，一个应用程序的变量内容就是典型的应用程序的状态。

### 状态性
**状态性**的程序，应用或组件将有关当前状态的数据存储在内存中，它们能改变状态，也能访问改变历史。下面是个有状态的例子：
```javascript
// 状态性
var number = 1;
function increment() {
  return number++;
}
increment(); // global variable modified: number = 2
```

### 无状态性
**无状态性**函数或组件执行任务时就像第一次运行它们一样，每次都是，这意味着它们不引用或使用之前执行所产生的信息。无状态性使引用透明性产生了可能，这样的函数仅依赖参数且不需要访问自身作用域之外的数据。***纯函数是无状态性的***，来看下面的例子：
```javascript
// 无状态性
var number = 1;
function increment(n) {
  return n + 1;
}
increment(number); // 全局变量 不会 修改：返回 2
```
无状态性程序确实仍然在管理状态，然而，它们返回当前的状态，并不改变之前的状态。**这是个函数式编程的原则**。

### 状态的优点
状态管理对任何复杂应用的欧式至关重要的。状态性函数或组件会改变状态并记录历史，但加重了调试和debug的复杂性。无状态性函数仅依赖参数去生成结果。一个无状态性的程序总是返回新的状态而不是修改已有的状态。

想学习更多状态相关，可参考下列资源：
[State](https://en.wikipedia.org/wiki/State_(computer_science))
[Advantages of stateless programming](https://stackoverflow.com/questions/844536/advantages-of-stateless-programming)
[Stateful and stateless components, the missing manual](https://toddmotto.com/stateful-stateless-components)
[Redux: predictable state container for JavaScript apps](https://redux.js.org/)
