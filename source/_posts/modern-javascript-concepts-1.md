---
title: 【译】现代JavaScript概念术语part1
date: 2020-05-18 00:10:24
tags: 
- javascript
categories:
- javascript
# top_img: https://cdn.auth0.com/blog/js-fatigue/JSLogo.png
---

## 引子
机缘巧合，好友推荐了一篇好文，读来神清气爽。翻译记录梳理成多篇(原文较长)，为今后温故知新。原文需翻墙，喜欢原文的可以[阅读原文](https://auth0.com/blog/glossary-of-modern-javascript-concepts/)，ok，废话少说，让我们开始吧~
****

在现代JS概念术语系列的第一部分，我们将理解函数式编程(**functional programming**)、响应式编程(**reactive programming**)和函数响应式变编程(**functional reactive programming**)。为了达到这个目的，我们将要学习以下关于纯净性(**purity**)，状态性(**statefulness**
)和无状态性(**statelessness**)，不变性(**immutability**)和可变性(**mutability**)，命令式(**imperative**)和声明式(**declarative**)编程，高阶函数(**higher-order function**)，可观察性(**ovservables**)，FP(**functional programming**)、RP(**reactive programming**)和FRP(**functional reactive programming**)范式

## 介绍
现代JavaScript在最近几年经历了大规模的扩展且并未表露出一丝放缓的迹象，无数的概念在JS博客和文档中出现，对大部分前端开发者而言仍旧十分陌生。在本系列文章中，我们将学到目前前端开发领域里，中级和高级的概念，并且探索下如何在现代JavaScript中应用它们。

## 概念
在这篇文章种中，我们将列举对理解函数式编程、响应式编程、函数响应式编程以及它们在JavaScript的应用中至关重要的概念。
你可以直接进入相应的概念，或者顺序阅读并学习它们
+ [纯净性：纯函数(**pure function**)，非纯函数(**impure function**)，副作用(**side effect**)](/2020/05/18/modern-javascript-concepts-1-purity/)
+ [状态：状态性(**stateful**)和无状态性(**stateless**)](/2020/05/18/modern-javascript-concepts-1-state/)
+ [不变性(**immutability**)和可变性(**mutability**)](/2020/05/19/modern-javascript-concepts-1-immutability/)
+ [命令式(**imperative**)和声明式(**declarative**)编程](/2020/05/19/modern-javascript-concepts-1-imperative-declarative/)
+ [高阶函数(**higher-order function**)](/2020/05/19/modern-javascript-concepts-1-hoc)
+ [函数式编程(**functional programming**)](/2020/05/19/modern-javascript-concepts-1-functional-programming)
+ [可观察对象(**observables**)](/2020/05/19/modern-javascript-concepts-1-observables/)
+ [响应式编程(**reactive programming**)](/2020/05/20/modern-javascript-concepts-1-reactive-programming)
+ [函数响应式编程(**functional reactive programming**)](/2020/05/20/modern-javascript-concepts-1-functional-reactive-programming)

## 总结(***译者注：请确保已看完上述概念***)
最后，我们将引用[《Eloquent JavaScript》](https://eloquentjavascript.net/1st_edition/)第一版中的另一段精彩语录：
> "一个学生一动不动地坐在电脑前已经好几个小时了，忧郁的皱着眉。他试图用一个漂亮的解决方案去解决一个困难的问题，但始终没找到正确的方法。Fu-Tzu敲了下他的后脑并大声说「写点什么！」。这个学生开始写了个缺乏优雅的方案。当他写完后，他突然悟出了一个漂亮的解决方案。"
——Marijn Haverbeke, [Eloquent JavaScript, 1st Edition, Chapter 6](http://eloquentjavascript.net/1st_edition/chapter6.html)

针对理解[函数式编程](/2020/05/19/modern-javascript-concepts-1-functional-programming)
+ [可观察对象(**observables**)](/2020/05/19/modern-javascript-concepts-1-observables/)，[响应式编程](/2020/05/20/modern-javascript-concepts-1-reactive-programming)和[函数响应式编程](/2020/05/20/modern-javascript-concepts-1-functional-reactive-programming)的必要概念可能都很难掌握，更别说精通了。利用泛型基础来编写代码只是第一步，即使刚开始并不完全可靠。实践会照亮前行的路，也会暴露潜在的问题。
