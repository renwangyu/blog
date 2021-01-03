---
title: 【译】现代JavaScript概念术语part1——响应式编程(reactive programming)
date: 2020-05-20 10:42:07
tags: 
- javascript
categories:
- javascript
top_img: /img/php-code.jpg
cover: /img/php-code.jpg
---

## 响应式编程
响应式编程关注于随着时间推移，以声明方式(做什么，而不是怎么做)去响应传入事件。
响应式编程经常与响应式扩展相关联，是一种对可观察流的异步编程API。响应式扩展(简称Rx*)为各种语言提供了三方库，也包括JavaScript(RxJS)。

### 实践：JavaScript中的响应式编程
这有一个带有可观察对象响应式编程的例子，有一个供用户输入六位确认码输入框，我们希望尝试打印出最新的有效码，HTML代码可能看上去是这样的：
```html
<!-- HTML -->
<input id="confirmation-code" type="text">
<p>
  <strong>Valid code attempt:</strong>
  <code id="attempted-code"></code>
</p>
```
我们用RxJS创建输入事件的流来实现我们想要的功能，如下：

```javascript
// JS
const confCodeInput = document.getElementById('confirmation-code');
const attemptedCode = document.getElementById('attempted-code');

const confCodes$ =
  Rx.Observable
    .fromEvent(confCodeInput, 'input')
    .map(e => e.target.value)
    .filter(code => code.length === 6);

const subscription = confCodes$.subscribe(
  (value) => attemptedCode.innerText = value,
  (event) => { console.warn(`Error: ${event}`); },
  () => { console.info('Completed!'); }
);
```
代码示例[JSFiddle: Reactive Programming with JavaScript](https://jsfiddle.net/kmaida/v1ozuwgu/)。我们从`confCodeInput`元素观察事件，然后用`map`方法从每一个输入事件获取`value`。然后我们过滤`filter`任何不是六位字符的结果，让它们不会出现在返回的流中。最后，我们订阅`subscribe`可观察的`confCodes$`并且打印出最新有效的确认码。请注意，这是以声明的方式对事件进行响应的：这是响应式编程的关键。

### 响应式编程的优势
响应式编程范式囊括了观察和响应异步数据流中的事件。RxJS在[Angular](https://medium.com/google-developer-experts/angular-introduction-to-reactive-extensions-rxjs-a86a7430a61f#.41aap1i8a)中的应用，最为JavaScript响应式编程的一种解决方案，就获得了极大的流行。

想学习更多响应式编程相关，可参考下列资源：
[The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
[Introduction to Rx](http://introtorx.com/)
[The Reactive Manifesto](http://www.reactivemanifesto.org/)
[Understanding Reactive Programming and RxJS](https://auth0.com/blog/understanding-reactive-programming-and-rxjs/)
[Reactive Programming](http://paulstovell.com/blog/reactive-programming)
[Modernization of Reactivity](https://davidwalsh.name/modernization-reactivity)
[Reactive-Extensions RxJS API Core](https://github.com/Reactive-Extensions/RxJS/tree/master/doc/api/core)

****
阅读原文：[https://auth0.com/blog/glossary-of-modern-javascript-concepts/](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#L-span-id--reactive-programming----span-Reactive-Programming)