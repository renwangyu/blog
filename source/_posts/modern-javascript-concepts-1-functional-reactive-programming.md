---
title: 【译】现代JavaScript概念术语part1——函数响应式编程(functional reactive programming)
date: 2020-05-20 11:24:21
tags: 
- javascript
categories:
- javascript
top_img:
---

## 函数响应式编程
简单地说，函数响应式编程可以概括为随着时间的推移以声明的方式响应事件或行为(***译者注：监听事件，纯函数处理***)。更深入理解FRP，我们先来看一下FRP的公式，然后再研究它与JavaScript的关系。

### 什么是函数响应式编程？
FPR公式制定者的([Conal Elliot](https://twitter.com/conal))给出了一个完整的定义，即函数响应式编程是"外延的，时间连续的"。Elliot强调他更愿意把这个编程范式描述为外延的连续时间编程，而不是“函数响应式编程”。
函数响应式编程，如同它基本原始的定义，有两个根本属性：
+ **外延**：意为每个函数或类型都是精确的，简单的，与实现无关的("函数式"引用如此)。
+ **时间连续**：[变量在很短的时间内有一个特定的值，任意两点之间有无数个其他的点](https://en.wikipedia.org/wiki/Discrete_time_and_continuous_time#Continuous_time)，提供转换灵活性、效率、模块化和准确性(“响应式”引用如此)。
简而言之，[函数响应式编程就是随时间变化的声明式编程](https://www.quora.com/What-is-Functional-Reactive-Programming)

为了理解连续时间/时间的连续性，考虑使用矢量图形进行类比。矢量图具有无限的分辨率，不同于位图(离线分辨率)，矢量图可以无限缩放，从不会像素化或模糊化，位图在放大或缩小时则会出现这种情况。
> "FRP表达式描述了值随时间的整个变化过程，将这些变化直接表示为一等值"
  ——Conal Elliot

函数响应式编程应当具有如下特性：
+ 动态：能随时间或输入变化做出响应
+ 时变：响应行为时连续的，响应值时离散的
+ 高效：当改变发生，减少必要的处理
+ 历史关注：纯函数将状态从上一个时间点映射到下一个时间点，状态更改关注本地元素，而不是全局程序状态

Conal Elliot[关于FRP本质和起源的幻灯片](http://conal.net/talks/essence-and-origins-of-frp-lambdajam-2015.pdf)。源于天然的功能性，纯净性和惰性，编程语言Haskell实现了真正的FRP。Evan Czaplicki，Elm的作者，在[《Controlling Time and Space: Understanding the Many Formulations of FRP》](https://www.google.com/sorry/index?continue=https://www.youtube.com/watch%3Fv%3DAgu6jipKfYw&q=EgTLzY00GJ6Tk_YFIhkA8aeDS6W2g6ykVIRQERXVNMxYJaLrD1jbMgFy)一文中做了很好的概述。

事实上，我们简要聊一下Evan Czapliki的[Elm](https://auth0.com/blog/creating-your-first-elm-app-part-1/)。Elm是一个用于构建web应用的函数式类型化语言，它会被编译成JavaScript，CSS和HTML。Elm的架构受Redux在JS应用中状态容器的启发，Elm通常被认为是真正的函数响应式编程语言。但在版本0.17，它实现了订阅，替代了使语言更容易学习和使用而使用信号，这么做，使Elm告别了FRP。

### 实践：函数响应式编程和JavaScript
传统的FRP定义很难被理解，尤其是哪些没有例如Haskell或Elm语言使用经验的开发者。然而，这个术语开始在前端生态中出现的愈加频繁，所以让我们来了解下它在JavaScript中的应用吧。

为了整合你已经了解的JS中的FRP，对Rx*，Bacon.js，Angular和其他不符合Conal Elliot对FRP的两个基本定义的理解时非常重要的。Elliot说Rx*和Bacon.js并不是FRP，相反的，他们是"[受FRP启发的组合事件系统](https://stackoverflow.com/questions/5875929/specification-for-a-functional-reactive-programming-language#comment36554089_5878525)"。

函数响应式编程，与JavaScript的实现尤为相关，指在创建和响应流的时候采用函数式风格编程。这与Elliot最初的公式(特别排除了流作为组件)相差甚远，但仍受到传统FRP的启发。

理解JavaScript天生的与用户、界面、DOM(后端)的交互性也是很重要的，副作用和命令式代码在这个过程中是常见的，即使在采用功能性或功能性反应性方法时也是如此。如果没有命令式或非纯代码，带有UI的JS web应用程序就没有多大用处，因为它无法与环境进行交互。

我们来开一个例子来说明受FRP启发的JavaScript的基本原则。这个例子采用RxJS并打印出鼠标十秒内的移动：
```javascript
// create a time observable that adds an item every 1 second
// map so resulting stream contains event values
const time$ =
  Rx.Observable
    .timer(0, 1000)
    .timeInterval()
    .map(e => e.value);

// create a mouse movement observable
// throttle to every 350ms
// map so resulting stream pushes objects with x and y coordinates
const move$ =
  Rx.Observable
    .fromEvent(document, 'mousemove')
    .throttleTime(350)
    .map(e => { return {x: e.clientX, y: e.clientY} });

// merge time + mouse movement streams
// complete after 10 seconds
const source$ =
  Rx.Observable
    .merge(time$, move$)
    .takeUntil(Rx.Observable.timer(10000));

// subscribe to merged source$ observable
// if value is a number, createTimeset()
// if value is a coordinates object, addPoint()
const subscription =
  source$.subscribe(
    // onNext
    (x) => {
      if (typeof x === 'number') {
        createTimeset(x);
      } else {
        addPoint(x);
      }
    },
    // onError
    (err) => { console.warn('Error:', err); },
    // onCompleted
    () => { console.info('Completed'); }
  );

// add element to DOM to list out points touched in a particular second
function createTimeset(n) {
  const elem = document.createElement('div');
  const num = n + 1;
  elem.id = 't' + num;
  elem.innerHTML = `<strong>${num}</strong>: `;
  document.body.appendChild(elem);
}

// add points touched to latest time in stream
function addPoint(pointObj) {
  // add point to last appended element
  const numberElem = document.getElementsByTagName('body')[0].lastChild;
  numberElem.innerHTML += ` (${pointObj.x}, ${pointObj.y}) `;
}
```
你可以在这运行这段代码[JSFiddle: FRP-inspired JavaScript](https://jsfiddle.net/kmaida/3v8yw02s/)。运行fiddle，在屏幕结果区域移动你的鼠标计数至10秒，你就能看到鼠标坐标与计数器一起出现，表示在每个1秒的时间间隔内鼠标的位置。

我们一步步简要地讨论下这个实现。

首先，我们创建了一个可观察对象叫`time$`，这是个每`1000ms`(每秒)增加个值的定时器，通过`map`定时器事件获取它的`value`，推送到结果流中。

其次，我们从鼠标移动事件`document.mousemove`又创建了个可观察对象`move$`。鼠标移动是持续的，在这一过程任何点，都有无数的点在其中。我们使用节流，让结果流更容易管理。然后我们就能`map`事件来返回一个带有x和y属性的对象以代表鼠标的坐标。

接着我们想要合并`time$`和`move$`的流，这是个组合操作。通过这种方法我们可以绘制每个时间间隔内发生的鼠标移动。我们把可观察结果叫做`Source$`。我们也能限制`source$`让它十秒后完成(`10000ms`)。

现在我们已经有了时间和移动的组合流，就可以创建`subscription`去订阅`source$`，就能响应它了。在`onNext`回调里，检查值是否是一个`number`，如果是，我们就调用`createTimeset()`函数，如果是一个坐标对象，我们就调用`addPoint()`函数。在`onError`和`onCompletes`回调里，我们仅简单的记录些信息。

再来看下`createTimeset(n)`函数，我们在每个时间间隔创建了个新的`div`元素，记录标识，添加到DOM。

在`addPoint(pointObj)`函数里，我们在最近的`div`(***译者注：最后append的div***)打印出最新的坐标。这将把每组坐标与其对应的时间间隔联系起来。我们现在可以读取鼠标移动的位置了。

> 注：这些函数都是非纯的：它们没有返回值，并且都会产生副作用，副作用就是对DOM的操作。就像早期提到的，我们用JavaScript写的的app，会频繁地与函数外的作用域进行互动。

### 函数响应式编程的优势
FRP的编码行为即使用纯函数响应事件来映射从上一个时间到到下一个时间点的状态。JavaScript中的FRP并为遵循Conal Elliot提出的FRP的两条基本原则，但对原始概念的抽象是有价值的。JavaScript重度依赖副作用和命令式编程，但我们仍能从FRP的概念中获得收益，来改进我们的JS。

最后，思考下引用第一版[《loquent JavaScript》](http://eloquentjavascript.net/1st_edition/)的话(第二版在[这里](https://eloquentjavascript.net/))：
> "Fu-Tzu写了一个充满全局状态和不明确快捷键的小程序，读完后，一个学生问到：你警告我们不要用这些技术，但我发现这些都在你的程序里，为什么会这样？
Fu-Tzu说：若果房子没着火，那就没有必要取水桶。这不是在鼓励随意的编码，而是在警告人们不要神经质地遵守经验法则。"

想学习更多函数响应式编程相关，可参考下列资源：
[Functional Reactive Programming for Beginners](https://www.google.com/sorry/index?continue=https://www.youtube.com/watch%3Fv%3DvLmaZxegahk&q=EgTLzY00GMSok_YFIhkA8aeDS1DVPt8VyNX9fAptF6zS2ive8QRlMgFy)
[What is Functional Reactive Programming?](https://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming/1030631#1030631)
[Haskell - Functional Reactive Programming](https://wiki.haskell.org/Functional_Reactive_Programming)
[Composing Reactive Animations](http://conal.net/fran/tutorial.htm)
[Specification for a functional reactive programming language](https://stackoverflow.com/questions/5875929/specification-for-a-functional-reactive-programming-language#5878525)
[A more elegant specification for FRP](https://github.com/conal/talk-2015-more-elegant-frp)
[Functional Reactive Programming for Beginners](https://www.google.com/sorry/index?continue=https://www.youtube.com/watch%3Fv%3DvLmaZxegahk&q=EgTLzY00GKepk_YFIhkA8aeDSxqvUewRrAv0BzLxpvTfMPPU1ZBDMgFy)
[Elm - A Farewell to FRP](http://elm-lang.org/blog/farewell-to-frp)
[Early inspirations and new directions in functional reactive programming](http://conal.net/blog/posts/early-inspirations-and-new-directions-in-functional-reactive-programming)
[Breaking Down FRP](https://blog.janestreet.com/breaking-down-frp/)
[Rx* is not FRP](https://twitter.com/ReactiveX/status/483625917491970048)

****
阅读原文：[https://auth0.com/blog/glossary-of-modern-javascript-concepts/](https://auth0.com/blog/glossary-of-modern-javascript-concepts/#L-span-id--functional-reactive-programming----span-Functional-Reactive-Programming)