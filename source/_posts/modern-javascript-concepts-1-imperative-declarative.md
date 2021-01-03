---
title: 【译】现代JavaScript概念术语part1——命令式(imperative)和声明式(declarative)编程
date: 2020-05-19 16:10:56
tags: 
- javascript
categories:
- javascript
top_img: /img/php-code.jpg
cover: /img/php-code.jpg
---

## 命令式编程和声明式编程
某些语言被设计成命令式(C,PHP)，而某些被设计成声明式(SQL,HTML)，JavaScript(其余如Java和C#)则能支持这两种编程范式。
大部分熟悉基本JavaScript的开发者惯用命令式编码：用指令告诉计算机怎么达到预期的结果。如果你写过`for`循环，那么你就写过命令式JS。
声明式编码告诉计算机什么是你想要的结果，而不是怎么做，计算机关心的是没有来自开发者明确指示的情况下如何获得最终结果。如果你用过`Array.map`，那么你就写过声明式JS。

### 命令式编程
命令式编程描述了让程序逻辑化工作的明确的声明和修改的语句。
设想有一个函数，能使一个整数数组中的每个整数增加。命令式JavaScript的例子可能是这样的：
```javascript
function incrementArray(arr) {
  let resultArr = [];
  for (let i = 0; i < arr.length; i++) {
    resultArr.push(arr[i] + 1);
  }
  return resultArr;
}
```
这个函数精确地展示了这个功能的工作逻辑：我们迭代了数组并精确的增加了每一个数值，再把它推到一个新的数组里，然后我们返回了结果。这是一步步描述函数的逻辑。

### 声明式编程
声明式编程描述了程序的逻辑目的(不包括怎么做。
一个非常明确的声明式编程的例子就是SQL。我们能查询数据库表(people)里最后名字是smith的人，就像：
```sql
SELECT * FROM People WHERE LastName = 'Smith'
```
这个例子非常容易地表达了我们想做什么，这里没有描述怎么才能找到，计算机自己知道怎么弄。

考虑上面我们命令式实现的`incrementArray()`函数，现在让我们用声明式实现下：
```javascript
function incrementArray(arr) {
  return arr.map(item => item + 1);
}
```
我们表达了我们想做的，而不是怎么做。`Array.map()`方法会返回一个对老数组里每一个元素执行回调函数后得到的新元素的数组。这个方法不会修改已经存在的值，也不显示如何创建新数组的逻辑。
> 注：JavaScript里的`map`，`reduce`和`filter`都是声明式数组方法。工具库例如`lodash`提供除`map`，`reduce`和`filter`等额外的方法，如`takeWhile`，`uniq`，`zip`等。

### 命令式和声明式编程的优势
作为一门语言，JavaScript能使用命令式和声明式两种编程范式。大部分我们看到的和编写的都是命令式的，然而随着函数式编程的崛起，声明式将变得越来越普遍。
声明式编程在代码简洁和可读性上具有明显的优势，但同时它会显得有点魔幻(***译者注：魔术代码，魔术变量***)。许多JavaScript初学者在深入学习声明式编程之前，都可以从命令式JS的编写经验中获益。

想学习更多命令式和声明式编程相关，可参考下列资源：
[Imperative vs Declarative Programming](https://tylermcginnis.com/imperative-vs-declarative-programming/)
[What's the Difference Between Imperative, Procedural, and Structured Programming?](http://softwareengineering.stackexchange.com/questions/117092/whats-the-difference-between-imperative-procedural-and-structured-programming)
[Imperative and (Functional) Declarative JS In Practice](http://www.redotheweb.com/2015/09/18/declarative-imperative-js.html)
[JavaScript's Map, Reduce, and Filter](https://danmartensen.svbtle.com/javascripts-map-reduce-and-filter)

****
阅读原文：[https://auth0.com/blog/glossary-of-modern-javascript-concepts/](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#imperative-declarative)