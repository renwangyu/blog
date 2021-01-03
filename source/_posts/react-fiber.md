---
title: 简单读懂React Fiber
date: 2020-07-19 11:19:50
tags:
- react
- javascript
categories:
- 前端
keywords:
- react fiber
top_img: /img/post/react-fiber.png
cover: /img/post/react-fiber.png
---

### 前言
React在带给前端便捷开发体验的同时，也在不断的尝试新的技术体验，为开发者带来新的惊喜。16版本之后，例如`hooks`机制引入，仿佛打开了新世界的大门。除此之外，另一个机制`fiber`，虽然对开发者比较透明，不像hooks那样开发者能直接调用，但光从2年左右的研发期可以看出，可见FB开发团队对其的重视程度。理解fiber的概念，对于今后在react中一些生命周期的调用，或是渲染性能优化的理解，都是大有益处的。

### React Fiber的前因后果
首先来看下React Fiber的概念：
> React Fiber是一组用于react库的内部图形渲染算法，区别于曾经的stack渲染算法。React的实际使用语法并未改变，只有执行语法的方式发生了变化。——[《维基百科React Fiber》](https://en.wikipedia.org/wiki/React_Fiber)

从上可知，**React Fiber是对核心算法的一次重新实现**。对开发者而言无需任何变动，还是原来的味道，只是不一样的配方~~
那为什么耗费时间人力去做fiber的重构呢？这里就涉及到渲染更新的机制问题。

#### 16版本之前的更新
React在运行时存在3层结构：
+ DOM：真实DOM节点。
+ Instances：React维护的vDOM tree node(想象为连接上线两层的桥)。其实就是createELement生成的vDOM。
+ Elements：描述UI的React element(type，props)。

Instances是根据Elements创建的`实例`，对组件及DOM节点的`抽象表示`，**vDOM tree维护了组件状态及组件与DOM的关系**。
在首次渲染过程中构建了vDOM tree，后续更新时(setState)，根据diff算法获得新老vDOM tree的差异，并把差异应用(patch)到DOM树。

React开发团队称这种自顶而下、无法中断的更新流程为`stack reconciler`(相对而言，fiber机制称为`fiber reconciler`)：简单来说就是一种当开发者调用setState通知React更新的时候，React遍历所有节点，生成vDOM tree ，diff出差异，调用生命周期函数，最后更新DOM tree，这一系列操作都是一鼓作气、一气呵成的，都不带喘息的，更别说休息了~但这里就有个问题，我们都知道浏览器主线程中，js运算、layout布局，页面绘制等都是互斥的，这么一长串的更新连招，如果React的渲染更新里一个环节耗时严重，就会把页面绘制挡在门外，导致掉帧，甚至忽视用户的输入操作等，对用户来说就会出现页面卡顿的感觉。

#### Fiber机制的更新
破解同步更新方法其实就是把同步方法`碎片化(分片)`：**增量更新**
> 把渲染/更新过程（递归diff）拆分成一系列小任务，每次检查树上的一小部分，做完看是否还有时间继续下一个任务，有的话继续，没有的话把自己挂起，主线程不忙的时候再继续。

综上可见，Fiber机制有着如下特性：
+ **增量渲染**(把渲染更新拆分成小片，每次执行完询问是否继续下一小片，不阻塞dom渲染，不掉帧)。
+ 更新时**能暂停，终止，复用渲染任务**。
+ 不同的类型的更新有不同的`优先级`，大致如下：
  - synchronous：同步执行，和旧版的stack reconciler一样(要求尽量快，不管会不会阻塞UI线程)。
  - task：在next tick前执行。
  - animation：下一帧前执行(requestAnimationFrame调度)。
  - high：在不就得将来执行(requestIdleCallback调度)。
  - low：稍微延迟执行也可以(requestIdleCallback调度)。
  - offscreen：下一次render或scroll时才执行。指的是当前隐藏的、屏幕外的(看不见的)元素。
  高优先级的任务(用户input)可以打断低优先级的任务(diff)的执行，从而更快生效。
+ 并发方面新的基础能力。


### Fiber tree
要实现增量更新，必然依赖更多的上下文信息，之前的vDOM tree显然有些力不从心，所以诞生了`Fiber tree(带有Fiber上下文的vDOM tree)`~~更新过程就是根据输入数据以及现有的Fiber tree构造出新的Fiber tree`快照`(workInProgress tree)。
![vDOM Tree(图上)与Fiber Tree(图下)对比图](/img/post/fiber-tree.png)
相比较vDOM tree，其实`Fiber tree`是一种基于`单链表的树结构`。而`fiber`也是对应的存储每一个节点信息的`数据结构`，大概长这么样：
```javascript
{
  stateNode, // 状态节点
  child,  // 子节点
  return,  // 表示当前节点处理完毕后，应该向谁提交自己的成果（effect list）
  sibling,    // 兄弟节点
  ...
}
```
每一个fiber节点都知道该对谁汇报`return`，下属是谁`child`，同级是谁`sibling`。在此基础上，以前的运行时三层，就拓展为五层：
+ DOM：真实DOM节点。
+ effect：workInProgress tree上的每一个节点都有一个effect list，用来存放diff结果，**当前节点更新完毕会向上merge effect list(queue收集diff结果)**。
+ workInProgress tree：是reconcile过程中从Fiber tree建立的当前进度快照，用于`断点恢复`。
+ Fiber tree：带有Fiber上下文的vDOM tree(用来描述增量更新所需的上下文信息)。
+ Elements：描述UI的React element(type，props)。

这样的层级，让原本不能中断的vDOM的diff，因为workInProgress tree的存在，转换为可以中断恢复的fiber的diff。在生成workInProgress tree的过程中，完成diff的操作，下面在reconciler阶段一详细阐述。

### Fiber Reconciler
既然是渲染更新算法的重构，那首当其冲就是`reconciler层`(这里是生成vDOM，并diff出差异的关键)。fiber把reconciler分成了两个阶段：
![Fiber Reconciler的两个阶段](/img/post/fiber-2-phase.png)

#### Phase1：render/reconciliation阶段
这一阶段是**可中断的**，也就是说在更新任务进行中被另一个更高优先级更新任务中断，则低优先级更新任务会终止，所做的工作也会完全抛弃，等更高优先级任务结束，再重头开始做。
以`上一个Fiber tree`为基础，开始自上而下逐节点构造`workInProgress tree(构建中的新Fiber tree)`的过程。通过`requestIdleCallback`来调度执行，每完成一个任务后回来看看有没有插队的(更紧急的)，每完成一组任务，把时间控制权交还给主线程，直到下一次`requestIdleCallback`回调再继续构建workInProgress tree。其中每一个组件节点工作如下：
1. 如果`当前节点`不需要更新，直接把`子节点`clone过来，跳到步骤5。反之，若需要更新则打个tag。
2. 更新`当前节点`的状态(props、state、context等)。
3. 调用shouldComponentUpdate，false的话，跳到步骤5。
4. 调用render获取新的子节点vDOM，并为其创建fiber(创建过程会尽量复用现有fiber，子节点增删也发生在这里)。
5. 如果没有产生child fiber，该工作单元结束，把effect list归并到return，并把当前节点的sibling作为下一个工作单元；否则把child作为下一个工作单元。
6. 查看是否如果没有剩余可用时间了，等到下一次主线程空闲时才开始下一个工作单元；否则，立即开始做。
7. 如果没有下一个工作单元了(回到了workInProgress tree的根节点)，Phase1结束，进入pending Commit状态。
**构建workInProgress tree的过程就是diff的过程**(React Fiber会找出需要更新哪些DOM)，工作循环结束时，**workInProgress tree的`根节点身上的effect list`就是收集到的所有side effect**(因为每做完一个都向上归并)。

#### Phase2：commit阶段
这一阶段是**不可中断的**，是对DOM的修改提交。
1. 处理effect list(包括3种处理：更新DOM树、调用组件生命周期函数以及更新ref等内部状态)。
2. 处理结束，所有更新commit到DOM树。

> 这个阶段应该避免大计算量的工作。

### 再谈workInProgress tree
之前提到workInProgress tree是reconcile过程中从Fiber tree建立的当前进度快照，构造的构成就是diff的过程。当构造完毕得到的就是新的Fiber tree，然后喜新厌旧(把current指针指向workInProgress tree，丢掉旧的Fiber tree)就好了。这样一来就完成了阶段一的工作，可以开始阶段二的一气呵成了~~之后再有新的更新，循环往复。
![workInProgress tree](/img/post/workIn-progress.png)

### React Fiber中的生命周期函数
由于Fiber Reconciler的两个阶段，相应的生命周期在代码中的调用也起了一点微妙的变化。
1. Phase1：render/reconciliation阶段的生命周期函数
  + componentWillMount
  + componentWillReceiveProps
  + shouldComponentUpdate
  + componentWillUpdate

这一阶段的fiber任务会暂停重来，所以这几个生命周期函数也有可能被**重复调用**。
> 像请求接口等操作放在willXXX内就有可能被反复请求。在以前并不会。

2. Phase2：commit阶段的生命周期函数
  + componentDidMount
  + componentDidUpdate
  + componentWillUnmount

由于在这一阶段是不可中断的，那么这几个生命周期函数就只会在各自的life阶段调用一次，不会被重复调用。

### 参考文章
[React Fiber是什么](https://zhuanlan.zhihu.com/p/26027085)
[完全理解React Fiber](http://www.ayqy.net/blog/dive-into-react-fiber/#articleHeader8)