---
title: 【译】现代JavaScript概念术语part1——纯净性(Purity)
date: 2020-05-18 19:28:33
tags: 
- javascript
categories:
- javascript
top_img:
---

## 纯净性：纯函数(**pure function**)，非纯函数(**impure function**)和副作用(**side effect**)
### 纯函数
**纯函数**的返回值只由传入值(函数参数)决定，且不带副作用。当给定想通的参数，则返回结果总是相同的。看下面的这个例子：
```javascript
function half(x) {
  return x / 2;
}
```
`half(x)`函数接受一个数值`x`并且返回`x`数值的一半。如果我们传入参数`8`，函数总是会返回`4`。当被调用时，纯函数可以它返回的结果所替代。换句话说，我们能在代码中的任何地方把`half(8)`替换`4`，将不会改变最终输出，这就是所谓的引用透明性(**referential transparency**)。
纯函数只关心传给它们的东西，举个例子，纯函数并不能引用来自父作用域的变量，除非它们做为明确的参数传入函数，即使这样，纯函数也无法修改父作用域。
```javascript
// 能改变的变量
let someNum = 8;

// 这不是纯函数！
function impureHalf() {
  return someNum / 2;
}
```
总结：
+ 纯函数必须带参数
+ 同样的输入(参数)总是生成同样的输出(结果)
+ 纯函数只依赖自身状态并且不改变外部状态(注：`console.log`会改变全局状态)(***译者注：alert同理***)
+ 纯函数不会产生副作用
+ 纯函数不能调用非纯函数

### 非纯函数
**非纯函数**会改变自身范围之外的状态，所有产生**副作用**的函数都是非纯函数。
思考下面的例子：
```javascript
// 非纯函数，产生一个副作用
function showAlert() {
  alert('This is a side effect!');
}

// 非纯函数，改变外部状态
var globalVal = 1;
function incrementGlobalVal(x) {
  globalVal += x;
}

// 看似纯函数的非纯函数
// 但相同的输入，得到不同的结果
function getRandomRange(min, max) {
  return Math.random() * (max - min) + min;
}
```

### JavaScript里的副作用
当一个函数或表达式改变了自身上下文之外的状态，这样的结果就是**副作用**。副作用的例子包括调用一个API，修改DOM，唤起一个alert对话框，写入数据库等等。如果一个函数产生副作用，它就被认为是非纯函数。引发副作用的函数是难以预测的，且当它们导致外部作用域改变的时候将变得难以调试。

### 纯净性的优势
大量高质量的代码，是由逻辑性程序性的调用纯函数的非纯函数组成的，但这仍然对调试和不可变性是有益的。引用透明性也带来了缓存化：缓存函数调用的结果，当同样的输入可复用缓存结果。不过确定函数是否真的是纯函数依然是十分有挑战性的。

想学习更多纯净性相关，可参考下列资源：
+ [Pure versus impure functions](https://ultimatecourses.com/blog/pure-versus-impure-functions)
+ [Master the JavaScript Interview: What is a Pure Function?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976#.kt48h2bfa)
+ [Functional Programming: Pure Functions](https://www.sitepoint.com/functional-programming-pure-functions/)

****
阅读原文：[https://auth0.com/blog/glossary-of-modern-javascript-concepts/](https://auth0.com/blog/glossary-of-modern-javascript-concepts/)