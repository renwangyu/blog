---
title: 【译】现代JavaScript概念术语part1——高阶函数(higher-order function)
date: 2020-05-19 17:29:30
tags: 
- javascript
categories:
- javascript
top_img:
---

## 高阶函数
高阶函数式这样的一个函数：
+ 接受另一个函数做为参数
+ 返回一个函数做为结果

在JavaScript世界里，函数是一等公民。它们能被当成值一样保存和传递：我们能把函数赋值给一个变量，或传递一个函数到另一个函数里。
```javascript
const double = function(x) {
  return x * 2;
}
const timesTwo = double;

timesTwo(4); // 结果: 返回 8
```
函数做为参数传递的典型例子就是回调函数，回调函数可以是匿名函数，也可以是命名函数：
```javascript
const myBtn = document.getElementById('myButton');

// 匿名函数
myBtn.addEventListener('click', function(e) { console.log(`Click event: ${e}`); });

// 命名函数
function btnHandler(e) {
  console.log(`Click event: ${e}`);
}
myBtn.addEventListener('click', btnHandler);
```
我们也能把函数做为参数传递给另一个我们创建的函数，然后执行函数参数：
```javascript
function sayHi() {
  alert('Hi!');
}
function greet(greeting) {
  greeting();
}
greet(sayHi); // "Hi!"
```
>注：如上实例，当传递一个命名函数做为参数，我们不需要用括号`()`。这就像我们当一个对象一样传递函数。括号会让函数执行，且把执行后的结果当做参数，替换函数本身。

高阶函数能返回另一个函数：
```javascript
function whenMeetingJohn() {
  return function() {
    alert('Hi!');
  }
}
var atLunchToday = whenMeetingJohn();

atLunchToday(); // "Hi!"
```

### 高阶函数的优势
JavaScript函数作为一等公民的特性，促进了函数式编程。


想学习更多高阶函数相关，可参考下列资源：
[Functions are first class objects in JavaScript](http://helephant.com/2008/08/19/functions-are-first-class-objects-in-javascript/)
[Higher-Order Functions in JavaScript](https://www.sitepoint.com/higher-order-functions-javascript/)
[Higher-order functions - Part 1 of Functional Programming in JavaScript](https://www.youtube.com/watch?v=BMUiFMZr7vk)
[Eloquent JavaScript - Higher-order Functions](http://eloquentjavascript.net/05_higher_order.html)
[Higher Order Functions](https://medium.com/functional-javascript/higher-order-functions-78084829fff4#.dwg58papp)

****
阅读原文：[https://auth0.com/blog/glossary-of-modern-javascript-concepts/](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#L-span-id--higher-order-functions----span-Higher-order-Functions)