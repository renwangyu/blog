---
title: Eventloop和Render的相关总结
date: 2020-05-28 14:52:30
tags:
- javascript
categories:
- 前端
keywords:
- 事件队列
- 浏览器渲染
top_img: https://pic4.zhimg.com/v2-0b35a3df0b2e2712839ce551062e6d7f_1200x500.jpg
cover: https://pic4.zhimg.com/v2-0b35a3df0b2e2712839ce551062e6d7f_1200x500.jpg
---

### 前言
前两天针对react hooks的理解中，涉及了render的时序。这让我想到event loop，因此还是来回顾总结下event loop以及render的相关知识吧。

### 事件循环
在进入正题前，我们先看看几个点：
+ 首先，javascript是**单线程**的，这是由于需要操作DOM决定的，多线程操作了谁也不服谁。
+ 因为单线程，所以在主线程上执行过程中就要排队执行，就是**执行栈**`execution context stack`。
+ 执行栈里有些任务很快，有些会很慢(比如ajax请求，IO操作等)，为了避免阻塞，就把任务分为两种：**同步任务**和**异步任务**。
+ **同步任务**`synchronous`：在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。
+ 重点看下**异步任务**`asynchronous`：执行完成后不进入主线程、而进入**任务队列**`task queue`的任务，只有当主线程执行完所有同步任务，再去任务队列中捞异步任务来执行栈执行。
+ **回调函数**`callback`：就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。
+ 主线程不断重复上面执行捞取的步骤。

重复不断？嗯，这个循环的过程就是**事件循环**`event loop`。
![事件循环](/img/post/event-loop.jpeg)

### 任务队列分类
首先明确，只要在任务队列里的，都是异步任务了，ok，继续往下：
+ **宏任务**`macrotask`(也叫tasks)：依次进入**宏任务队列**`macro task queue`：
  + setTimeout
  + setInterval
  + setImmediate (node独有)
  + requestAnimationFrame (浏览器独有)
  + I/O (比如用户点击交互什么的)
  + UI rendering (浏览器独有)
  + requestIdleCallback (浏览器独有)
+ **微任务**`microtask`(也叫jobs)：依次进入**微任务队列**`micro task queue`:
  + Promise (then，catch，finally)
  + Object.observe
  + MutationObserver
  + process.nextTick (Node独有)
  + async，await (Promise语法糖)

### 真刀真枪看程序在浏览器的运行
弄清楚上述概念，那么我们视野放宽点，来一场web浏览器运行渲染之旅了。
打开浏览器当我们输入url按回车，经过dns解析，https的n次握手，资源加载巴拉巴拉。。。好了，页面出来了，步入正题：
+ 主线程`执行栈`顺序执行完代码，从`任务队列`获取一个`宏任务`。
+ 唔？`任务队列`里有`微任务`？执行并清空它们~ **如果在执行过程中又来了新的微任务，一起执行(有插队的感觉~)**。
+ 异步(宏任务和微任务)也都执行完了，该进入渲染阶段了吧？等等，这里浏览器有个**渲染时机**`rendering opportunity`的概念：
  + 浏览器尽可能`保持不丢帧`，页面性能无法维持60fps(每 16.66ms 渲染一次)的话，那么浏览器就会选择30fps的更新速率。
  + 如果浏览器上下文不可见，那么页面会降低到4fps左右甚至更低。
  + 浏览器判断更新渲染`是否会带来视觉上的改变`。
  + `requestAnimationFrame`是否有回调。
  浏览器还是很聪明的，会综合根据上面这些因素判断是否需要渲染，从而是否进入渲染阶段。不然就结束了，event loop继续循环~
+ 恭喜，进入**渲染阶段**
  + 对于将要渲染的文档，如果窗口发生变化，并监听了resize，执行该回调方法。
  + 对于将要渲染的文档，如果发生了滚动，并监听了scroll，执行该回调方法。
  + 执行`requestAnimationFrame`的回调。
  + 执行`IntersectionObserver`的回调。
+ 上面的细节都搞定，**重新渲染**。
+ 执行栈和任务队列都为空，进行`Idle`空闲周期算法，判断是否执行`requestIdleCallback`的回调。

### requestAnimationFrame
简称rAF，动画神器，不会丢帧。两个特点：
+ 在**重新渲染**前调用，保证渲染前的修改都能呈现在界面，不失帧。
+ **很可能在宏任务之后不调用**。这就是上面说的浏览器很智能，说不定就不渲染了~
总之做动画就应该用这个，setTimeout都是不靠谱的。

### requestIdleCallback
简称rIC，让我们把一些计算量较大但是又没那么紧急的任务放到空闲时间去执行。不要去影响浏览器中优先级较高的任务，比如动画绘制、用户输入等等。日常开发用的比较少，简单看看其运行：
+ 每个idle回调都是比较小的切片，要求我们去读取`rIC`提供给的`deadline`里的时间，去动态的安排我们切分的小任务。比如根据截止时间，保存已做的，跳出，到下一个idle时机继续。
  ![渲染有序进行](/img/post/rIC-1.png)

+ 有时候可能在未来几帧的时期，浏览器都是空闲的(执行栈和任务队列为空)，并没有发生影响视图的操作，也就不需要绘制页面，此时会有一个`50ms`的**deadline**，因为浏览器要提前应对用户可能的突发交互操作，防止用户产生卡顿的感觉。[详见此文](https://developer.mozilla.org/zh-CN/docs/Web/API/Background_Tasks_API)
![渲染长期空闲](/img/post/rIC-2.png)

### 结尾
到此，基本上一个事件循环，任务队列和渲染的流程以及相关的重要api介绍的七七八八了。也算是自己的一个理解和总结~