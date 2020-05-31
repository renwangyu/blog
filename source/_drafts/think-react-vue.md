---
title: react和vue异同的个人理解
date: 2020-05-28 14:52:30
tags:
- react
- vue
- javascript
categories:
- 前端
top_img: https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcScuVHsDBC2w9TfXZ25ofZjXOXcmb42sJqjjuuueSs91e4YGfR6&s
---

而 useState 返回的 count 和 setCount 则会被保存在组件对应的 Fiber 节点上，每个 React 函数每次执行 Hook 的顺序必须是相同的，举例来说。 这个例子里的 useState 在初次执行的时候，由于执行了两次 useState，会在 Fiber 上保存一个 { value, setValue } -> { value2, setValue2 } 这样的链表结构。

[对比](https://juejin.im/post/5e9ce011f265da47b8450c11#heading-15)
[react hooks最佳实践](https://juejin.im/post/5ec7372cf265da76de5cd0c9)
[译,尤雨溪：Vue3的设计过程](https://juejin.im/post/5ecf58b9f265da76e97d39da?utm_source=gold_browser_extension)