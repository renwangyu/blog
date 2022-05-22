---
title: React 18源码分析：新的挂载方式
date: 2022-05-09 19:50:47
tags:
- react
- javascript
categories:
- 前端
- keywords:
- react
top_img: https://keyholesoftware.com/wp-content/uploads/React-Hooks-862x431.png
cover: https://keyholesoftware.com/wp-content/uploads/React-Hooks-862x431.png
---

### 前沿
React从18版本起，不推荐再使用老版本的`ReactDOM.render`方法，建议用两个新方法，`createRoot`和root对象的`render`来替代：
```javascript
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```
就让我么我们通过源码调试，一步步看看其中发生了什么。
> hydrateRoot使用在ssr的，原理基本同createRoot。

### 简介
我们先看[createRoot](https://reactjs.org/docs/react-dom-client.html#createroot)方法，该方法至多接受两个参数，返回一个root对象，该root对象有两个方法：
+ render：用来把React元素渲染成DOM
+ unmount：卸载调用
需要注意的点：
+ `createRoot()`会控制传入的容器节点，render调用的时，会替换掉其中任何现有的DOM元素。
+ `createRoot()`不会修改容器节点，只修改容器的子节点。可以将组件插入到现有的DOM节点而不覆盖现有的子节点。
+ `createRoot()`不支持ssr，服务端渲染要用`hydrateRoot()`。

### createRoot源码分析
首先进入`react-dom/client.js`的createRoot，其实就是调用`react-dom`的createRoot，其中参数c就是我们传入的root元素：
```javascript
// client.js
var m = require('react-dom');

exports.createRoot = function(c, o) {
  i.usingClientEntryPoint = true;
  try {
    return m.createRoot(c, o);
  } finally {
    i.usingClientEntryPoint = false;
  }
};
// react-dom.js
function createRoot$1(container, options) {
  {
    if (!Internals.usingClientEntryPoint && !false) {
      error('You are importing createRoot from "react-dom" which is not supported. ' + 'You should instead import it from "react-dom/client".');
    }
  }
  return createRoot(container, options);
}
```
> 这里会提示，项目中需要从client.js引入，而不是直接从react-dom引入，这个就和client.js中的usingClientEntryPoint设置true对应上了。完成createRoot后会再设置为false。

#### createRoot
然后正式进入createRoot，其中开头有一些校验的逻辑和options的初始化，这里就不详细介绍
```javascript
// react-dom.js
function createRoot(container, options) {
  // 省略校验和options赋值，作为createContainer的参数
  var root = createContainer(container, ConcurrentRoot, null, isStrictMode, concurrentUpdatesByDefaultOverride, identifierPrefix, onRecoverableError);
  markContainerAsRoot(root.current, container);
  var rootContainerElement = container.nodeType === COMMENT_NODE ? container.parentNode : container;
  listenToAllSupportedEvents(rootContainerElement);
  return new ReactDOMRoot(root);
}
```

#### 创建FiberRoot对象
这是创建root对象的重要步骤，其中代码非常简单，就是创建FiberRoot结点：
```javascript
// react-dom.js
function createContainer(containerInfo, tag, hydrationCallbacks, isStrictMode, concurrentUpdatesByDefaultOverride, identifierPrefix, onRecoverableError, transitionCallbacks) {
  var hydrate = false;
  var initialChildren = null;
  return createFiberRoot(containerInfo, tag, hydrate, initialChildren, hydrationCallbacks, isStrictMode, concurrentUpdatesByDefaultOverride, identifierPrefix, onRecoverableError);
}
```
在进入之前，先看下FiberRootNode的类：
```javascript
function FiberRootNode(containerInfo, tag, hydrate, identifierPrefix, onRecoverableError) {
  this.tag = tag;
  this.containerInfo = containerInfo;
  this.pendingChildren = null;
  this.current = null;
  this.pingCache = null;
  this.finishedWork = null;
  this.timeoutHandle = noTimeout;
  this.context = null;
  this.pendingContext = null;
  this.callbackNode = null;
  this.callbackPriority = NoLane;
  this.eventTimes = createLaneMap(NoLanes);
  this.expirationTimes = createLaneMap(NoTimestamp);
  this.pendingLanes = NoLanes;
  this.suspendedLanes = NoLanes;
  this.pingedLanes = NoLanes;
  this.expiredLanes = NoLanes;
  this.mutableReadLanes = NoLanes;
  this.finishedLanes = NoLanes;
  this.entangledLanes = NoLanes;
  this.entanglements = createLaneMap(NoLanes);
  this.identifierPrefix = identifierPrefix;
  this.onRecoverableError = onRecoverableError;
  this.mutableSourceEagerHydrationData = null;
  this.effectDuration = 0;
  this.passiveEffectDuration = 0;
  this.memoizedUpdaters = new Set();
  var pendingUpdatersLaneMap = this.pendingUpdatersLaneMap = [];
  for (var _i = 0; _i < TotalLanes; _i++) {
    pendingUpdatersLaneMap.push(new Set());
  }
  switch (tag) {
    case ConcurrentRoot:
      this._debugRootType = hydrate ? 'hydrateRoot()' : 'createRoot()';
      break;

    case LegacyRoot:
      this._debugRootType = hydrate ? 'hydrate()' : 'render()';
      break;
  }
}
```
> 之后运行此处实例化的过程中，会初始化pendingUpdatersLaneMap，其中TotalLanes在源码中是31。

初始化root后(new了一个FiberRootNode)，通过`createHostRootFiber`创建一个新的FiberNode对象，然后挂在root的current上，并且把这个对象的stateNode反指向root，两者互相关联。
```javascript
function createFiberRoot(containerInfo, tag, hydrate, initialChildren, hydrationCallbacks, isStrictMode,concurrentUpdatesByDefaultOverride, identifierPrefix, onRecoverableError, transitionCallbacks) {
  var root = new FiberRootNode(containerInfo, tag, hydrate, identifierPrefix, onRecoverableError);
  var uninitializedFiber = createHostRootFiber(tag, isStrictMode);
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;
  {
    var _initialState = {
      element: initialChildren,
      isDehydrated: hydrate,
      cache: null,
      // not enabled yet
      transitions: null,
      pendingSuspenseBoundaries: null
    };
    uninitializedFiber.memoizedState = _initialState;
  }
  initializeUpdateQueue(uninitializedFiber);
  return root;
}
```
再看`initializeUpdateQueue`这个方法，其作用就是初始化了root.current对象的updateQueue：
```javascript
function initializeUpdateQueue(fiber) {
  var queue = {
    baseState: fiber.memoizedState,
    firstBaseUpdate: null,
    lastBaseUpdate: null,
    shared: {
      pending: null,
      interleaved: null,
      lanes: NoLanes
    },
    effects: null
  };
  fiber.updateQueue = queue;
}
```
#### markContainerAsRoot
通过上面的步骤，FiberRoot对象已经创建完，会在回到`createRoot`函数内，调用`markContainerAsRoot`方法把刚创建FiberRootNode对象的current(一个FiberNode对象)，用一个随机值做key，放到了当前容器dom上。

#### listenToAllSupportedEvents
最后一步，把支持的事件挂载到root(FiberRootNode对象)所在的dom元素上。
> 每次react运行都会用一个随机值作为key，表示当前的dom是否已经绑定事件。
+ allNativeEvents是一个有81个事件名的Set，数量太多不一一陈述
+ nonDelegatedEvents是一最不需要进行事件委托的事件名，共30个，基本是关于ajax和media的事件，数量太多也不一一陈述

```javascript
var listeningMarker = '_reactListening' + Math.random().toString(36).slice(2);
function listenToAllSupportedEvents(rootContainerElement) {
  if (!rootContainerElement[listeningMarker]) {
    rootContainerElement[listeningMarker] = true;
    allNativeEvents.forEach(function (domEventName) {
      // We handle selectionchange separately because it
      // doesn't bubble and needs to be on the document.
      if (domEventName !== 'selectionchange') {
        if (!nonDelegatedEvents.has(domEventName)) {
          listenToNativeEvent(domEventName, false, rootContainerElement);
        }
        listenToNativeEvent(domEventName, true, rootContainerElement);
      }
    });
    var ownerDocument = rootContainerElement.nodeType === DOCUMENT_NODE ? rootContainerElement : rootContainerElement.ownerDocument;
    if (ownerDocument !== null) {
      // The selectionchange event also needs deduplication
      // but it is attached to the document.
      if (!ownerDocument[listeningMarker]) {
        ownerDocument[listeningMarker] = true;
        listenToNativeEvent('selectionchange', false, ownerDocument);
      }
    }
  }
}
```
从源码可以看出，其中会涉及到的操作有：
+ 如果是委托事件，则会在捕获和冒泡阶段都进行监听；反之则只在捕获阶段进行监听。
+ 根据不同的事件设置优先级
+ 对于`touchstart`，`touchmove`和`wheel`事件，会进行passive为true的模拟优化（具体原因[戳此issue](https://github.com/facebook/react/issues/19651)）
+ 最后，除了`selectionchange`事件是直接绑在document上的，其余都会直接通过`listenToNativeEvent`绑定到root所在的dom上。

#### ReactDOMRoot
最后一步，把刚创建的`FiberRootNode`对象挂在`ReactDOMRoot`对象（新new的）的`_internalRoot`属性上，并返回。
```javascript
function ReactDOMRoot(internalRoot) {
  this._internalRoot = internalRoot;
}
```
至此，这个createRoot阶段就结束了，可以总结下，做了：
+ 创建FiberRootNode对象
+ 创建第一个FiberNode对象，并挂在FiberRootNode对象的current上，其自身的stateNode指向该FiberRootNode对象
+ 初始化事件监听
+ 创建ReactDOMRoot对象并返回