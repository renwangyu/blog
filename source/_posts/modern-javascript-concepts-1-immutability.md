---
title: 【译】现代JavaScript概念术语part1——不变性(immutability)和可变性(mutability)
date: 2020-05-19 11:19:17
tags: 
- javascript
categories:
- javascript
top_img:
---

## 不变性和可变性
相比其他编程语言，不变性和可变性的概念在JavaScript中略显模糊，然而当阅读JS函数式编程时，你仍然会感受到关于不变性的概念。了解它们为何具有经典意义以及在JavaScript中如何引用和实现是极其重要的。定义非常简单：

### 不变性(**immutable**)
如果一个对象具有不变性，那它的值在创建后不能被修改

### 可变性(**mutable**)
如果一个对象具有可变性，那它的值在创建后可以被修改

设计：JavaScript中的不变性和可变性
在JavaScript里，字符串(string)和数值(number)常量被设计成不可变性。这对我们考虑如何使用它们是非常容易理解的：
```javascript
var str = 'Hello!';
var anotherStr = str.substring(2);
// 结果: str = 'Hello!' (未改变))
// 结果: anotherStr = 'llo!' (新字符串)
```
在我们的`Hello!`使用`.substing()`方法时不会对原始字符串做修改，相反，它创建了一个新字符串。我们能给`str`变量重新赋值，但一旦我们创建了`Hello!`字符串，它将一直是`Hello!`。

数值变量同样也是不可变性的，下面的总是会得到同样的结果：
```javascript
var three = 1 + 2;
// 结果: three = 3
```
在任何情况下`1 + 2`都只等于`3`。

以上表明不变性确实存在于JavaScript的设计中，然而，JS开发人员往往更关注语言中允许改变的场景。举个例子，对象和数组在设计上是可变性的，考虑下面的例子：
```javascript
var arr = [1, 2, 3];
arr.push(4);
// 结果: arr = [1, 2, 3, 4]

var obj = { greeting: 'Hello' };
obj.name = 'Jon';
// 结果: obj = { greeting: 'Hello', name: 'Jon' }
```
通过以上例子，原始的对象是可变的，新的对象并没有返回。
想学习更多关于可变性在其他语言的应用，点击[Mutable vs Immutable Objects](https://www.interviewcake.com/concept/java/mutable)。

### 实践：JavaScript里的不可变性
函数式编程在JavaScript中已经获得了很大的发展势头，但在设计上，JS是一门非常多变的多范式语言。函数式编程增强了不可变性。当一名开发者试图修改一个不可变对象时，其他的函数式语言会抛出错误，那么我们用函数式或函数响应式编程的时候，该如何调整JS先天带来的缺失呢？
当我们谈起在JS中采用函数式编程的时候，时常会说起一个词"不可变性"，但这需要开发人员写代码的时候，时刻保持
这个意识。例如，Redux一来一个简单，不可变的状态树，然而，JavaScript本市是能够改变状态对象的。为了实现不可变的状态树，我们需要在状态改变时每次返回一个新的状态对象。
JavaScript对象能用方法`Object.freeze(obj)`使对象冻结，让它们成为不可变性，注意这是浅层冻结，意味着被冻结的对象内的属性仍然使能被修改的。为了更深层次的保证不可变性，类似Mozilla的`deepFreeze()`方法和npm上的`deep-freeze`都能递归地冻结对象。冻结操作最常用于调试，而不是JS应用中。调试中当改变发生时将提醒开发者，以纠正或避免在正式编译后不带有`Object.freeze`(使核心代码变得混乱)的改动发生。
我们也可以使用三方库来支持JS中地不可变性。[Mori](http://swannodette.github.io/mori/)提供基于闭包地持久化数据结构，Facebook的[Immutable.js](https://immutable-js.github.io/immutable-js/)为JS提供不可变的类型集合，譬如[Underscore.js](http://underscorejs.org/)和[lodash](https://lodash.com/)的工具库提供了方法和模块，进一步促进了不可变性的函数式编程的风格。

### 不可变性和可变性的优势
总得来说，JavaScript是一门非常可变的语言，有些JS编码的风格依赖与生俱来的可变性。然而，当编写函数式JS，实现不可变性，需要开发者的自我意识，当你无意改动某些点的时候，JS并不会原生抛出错误提示。虽然调试和三方库能起到帮助，但在JS中使用不可变性，需要练习和技巧。
不可变性确实具有优点，它使代码更容易推演，它也提供了持久化(**persistency**)，一种保持老版本的数据结构和只复制改变部分的能力。
不可变性的不足在于无法高效的实现许多算法和操作。

想学习更多不可变性和可变性相关，可参考下列资源：
[Immutability in JavaScript](https://www.sitepoint.com/immutability-javascript/)
[Immutable Objects with Object Freeze](http://adripofjavascript.com/blog/drips/immutable-objects-with-object-freeze.html)
[Mutable vs Immutable Objects](https://www.interviewcake.com/concept/java/mutable)
[Using Immutable Data Stuctures in JavaScript](http://jlongster.com/Using-Immutable-Data-Structures-in-JavaScript)
[Getting Started with Redux](https://egghead.io/courses/getting-started-with-redux)

****
阅读原文：[https://auth0.com/blog/glossary-of-modern-javascript-concepts/](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#L-span-id--immutable-mutable----span-Immutability-and-Mutability)