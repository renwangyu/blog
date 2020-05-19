---
title: 【译】现代JavaScript概念术语part1——函数式编程(functional programming)
date: 2020-05-19 17:51:39
tags: 
- javascript
categories:
- javascript
top_img:
---

## 函数式编程
看到这里，我们已经了解了关于纯净性，状态，不可变性，声明式编程和高阶函数的内容。这些都是理解函数式编程范式至关重要的概念。

### 实践：用JavaScript来函数式编程
函数式编程由以下概念构成：
+ 主要功能由没有副作用的纯函数实现。
+ 数据是不可变的。
+ 功能函数是无状态的。
+ 外部容器(命令式)代码管理副作用并执行(声明式)核心代码。*

> *如果我们尝试写一个由纯函数(无副作用)组成的JavaScript web应用，它不能与环境产生交互，因此将用途不大。

让我们看个例子，假设我们有一个text的副本，并且我们想得到它的文字长度，我们还想招到长度大于5个字符的关键字。如果用函数式编程，我们的目标代码可能看上去像这样：
```javascript
const fpCopy = `Functional programming is powerful and enjoyable to write. It's very cool!`;

// 移除标点符号
const stripPunctuation = (str) =>
  str.replace(/[.,\/#!$%\^&\*;:{}=\-_`~()]/g, '');

// 根据空格分隔生成数组
const getArr = (str) =>
  str.split(' ');

// 对传递的数组中的项进行计数
const getWordCount = (arr) =>
  arr.length;

// 在传递的数组中查找长度超过5个字符的元素
// 小写
const getKeywords = (arr) =>
  arr
    .filter(item => item.length > 5)
    .map(item => item.toLowerCase());

// 处理copy以准备字符串、创建数组、计数单词和获取关键字
function processCopy(str, prepFn, arrFn, countFn, kwFn) {
  const copyArray = arrFn(prepFn(str));

  console.log(`Word count: ${countFn(copyArray)}`);
  console.log(`Keywords: ${kwFn(copyArray)}`);
}

processCopy(fpCopy, stripPunctuation, getArr, getWordCount, getKeywords);
// 结果: Word count: 11
// 结果: Keywords: functional,programming,powerful,enjoyable
```
它被分解为易于理解的、具有明确目的的声明式函数。如果我们单步执行并阅读注释，其实并不需要进一步解释代码。每一个核心功能都是模块化的，并且只依赖它的输入([纯净性](/2020/05/18/modern-javascript-concepts-1-purity/))。最后那个函数处理了核心功能并生成了输出集合。这个`processCopy()`函数，相当于一个非纯容器，用以执行核心功能并管理副作用。我们使用[高阶函数](/2020/05/19/modern-javascript-concepts-1-hoc)，接受其他函数作为参数，来维护函数式风格。

### 函数式编程的优势
不可变数据和无状态性意味着程序存在的状态不会被修改，相反的，会返回新的值。纯函数被用做核心功能，为了实现功能并处理必要的副作用，非纯函数应该命令式的调用纯函数。

想学习更多函数式编程相关，可参考下列资源：
[Introduction to Immutable.js and Functional Programming Concepts](https://auth0.com/blog/intro-to-immutable-js/)
[Functional Programming For The Rest of Us](http://www.defmacro.org/ramblings/fp.html)
[Functional Programming with JavaScript](http://stephen-young.me.uk/2013/01/20/functional-programming-with-javascript.html)
[Don't be Scared of Functional Programming](https://www.smashingmagazine.com/2014/07/dont-be-scared-of-functional-programming/)
[So You Want to be a Functional Programmer](https://medium.com/@cscalfani/so-you-want-to-be-a-functional-programmer-part-1-1f15e387e536#.q8a7nwjat)
[lodash - Functional Programming Guide](https://github.com/lodash/lodash/wiki/FP-Guide)
[What is the difference between functional and imperative programming languages?](https://stackoverflow.com/questions/17826380/what-is-difference-between-functional-and-imperative-programming-languages)
[Eloquent JavaScript, 1st Edition - Functional Programming](http://eloquentjavascript.net/1st_edition/chapter6.html)
[Functional Programming by Example](http://tobyho.com/2015/11/09/functional-programming-by-example/)
[Functional Programming in JavaScript - Video Series](https://www.youtube.com/watch?v=BMUiFMZr7vk&list=PL0zVEGEvSaeEd9hlmCXrk5yUyqUag-n84)
[Introduction to Functional JavaScript](https://medium.com/functional-javascript/introduction-to-functional-javascript-45a9dca6c64a#.2qjh0i04y)
[How to perform side effects in pure functional programming](https://stackoverflow.com/questions/18172947/how-to-perform-side-effects-in-pure-functional-programming)
[Preventing Side Effects in JavaScript](https://davidwalsh.name/preventing-sideeffects-javascript)

****
阅读原文：[https://auth0.com/blog/glossary-of-modern-javascript-concepts/](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#L-span-id--immutable-mutable----span-Immutability-and-Mutability)