---
title: 【译】现代JavaScript概念术语part1——可观察对象(observables)
date: 2020-05-19 20:30:43
tags: 
- javascript
categories:
- javascript
top_img:
---

## 可观察对象
可观察对象类似于数组，除了不是存储在内存中，而是随着时间异步获取到的(也称为流)。我们能订阅可观察对象并响应它们触发的事件。JavaScript可观察对象是通过观察者模式实现的，[Reactive Extensions](http://reactivex.io/)(通常称为Rx*)通过RxJS为JS提供了一个可观察的库。

为了说明可观察对象的概念，让我们看一个简单的例子：改变浏览器窗口的大小。这种情况下很容易理解可观察对象。改变浏览器窗口大小会在一段时间内触发事件流直到窗口拖动到满意的尺寸。我们来创建一个可观察对象，并且监听它以用来响应尺寸改变的事件流：
```javascript
// create window resize stream
// throttle resize events
const resize$ =
  Rx.Observable
    .fromEvent(window, 'resize')
    .throttleTime(350);

// subscribe to the resize$ observable
// log window width x height
const subscription =
  resize$.subscribe((event) => {
    let t = event.target;
    console.log(`${t.innerWidth}px x ${t.innerHeight}px`);
  });
```
上述示例代码显示，随着窗口大小的改变，我们可以控制可观察流并订阅更改以响应集合中的新值。这是一个热观查的例子。

### 热观察(**hot observable**)
用户界面事件例如按钮点击，鼠标移动等都是**热观察**。热观察总是会推送(***译者注：触发***)即使我们没有通过订阅去特别响应它们。上述窗口尺寸的例子十个热观察：观察者`resize$`总是会触发，无论`subscription`是否存在。

### 冷观察(**cold observable**)
冷观察只有当我们订阅它，才会推送。如果我们再次订阅，它将重新开始。
让我们创建一个从1到5的可观察的数字集合:
```javascript
// create source number stream
const source$ = Rx.Observable.range(1, 5);

// subscribe to source$ observable
const subscription = source$.subscribe(
  (value) => { console.log(`Next: ${value}`); }, // onNext
  (event) => { console.log(`Error: ${event}`); }, // onError
  () => { console.log('Completed!'); }  // onCompleted
);
```
我们用`subscribe()`订阅我们刚创建的`source$`。在订阅时，值按顺序发送给观察者。`onNext`回调函数记录值:`Next: 1`、`Next: 2`等，直到完成`Completed!`。除非我们订阅创建的冷观察对象`source$`，不然它将不会推送。

### 可观察对象的优势
可观察对象是流，我们可以观察任何的流：从尺寸改变事件到存在的数组再到API的响应。我们几乎可以从任何东西中去创建可观察对象。Promise是具有单个触发值的可观察对象，但是可观察值可以随着时间的推移返回许多值。
我们可以通过许多方式来操作可观察对象。RxJS使用了许多运算符方法。可观察对象通常使用直线上的点进行可视化，如RxMarbles站点所示。由于流由异步事件组成，因此很容易以线性方式对其进行概念化，并使用这种可视化来理解Rx*操作符。例如，下面的RxMarbles图像说明了filter算子:
![Interactive diagrams of Rx Observables](https://cdn.auth0.com/blog/jsglossary/rxmarbles.png)

想学习更多可观察对象相关，可参考下列资源：
[Reactive Extensions: Observable](http://reactivex.io/documentation/observable.html)
[Creating and Subscribing to Simple Observable Sequences](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/gettingstarted/creating.md)
[The introduction to Reactive Programming you've been missing: Request and Response](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754#request-and-response)
[Introducing the Observable](https://egghead.io/lessons/javascript-introducing-the-observable)
[RxMarbles](http://rxmarbles.com/)
[Rx Book - Observable](https://xgrommx.github.io/rx-book/content/observable/index.html)
[Introducing the Observable](https://egghead.io/lessons/rxjs-introducing-the-observable)

****
阅读原文：[https://auth0.com/blog/glossary-of-modern-javascript-concepts/](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#L-span-id--observables----span-Observables)
