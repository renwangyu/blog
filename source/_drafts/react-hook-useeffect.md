---
title: 关于useEffect和useLayoutEffect的完全理解
date: 2022-05-08 18:27:47
tags:
- react
- javascript
categories:
- 前端
- keywords:
- react hooks
top_img: https://keyholesoftware.com/wp-content/uploads/React-Hooks-862x431.png
cover: https://keyholesoftware.com/wp-content/uploads/React-Hooks-862x431.png
---

### 前言
在使用React开发中，越来越多的同学已经习惯并喜欢上了函数式的编程，但我们在实际业务开发中，不可能所有的组件都是纯函数，所以不可避免会存在请求数据、订阅事件、手动操作DOM这些副作用（side effect），因此我们在函数组件中会使用useEffect和useLayoutEffect来管理副作用。本文通过调试React 18的源码，来彻底看看React hooks中关于effect的那些概念和流程。

### 初始化
在React 18中，创建container的方法已经从原来的ReactDOM.render，替换成如下操作：
```javascript
const root = createRoot(container);
root.render(<App />)
```
FiberRootNode
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
  {
    this.mutableSourceEagerHydrationData = null;
  }
  {
    this.effectDuration = 0;
    this.passiveEffectDuration = 0;
  }
  {
    this.memoizedUpdaters = new Set();
    var pendingUpdatersLaneMap = this.pendingUpdatersLaneMap = [];

    for (var _i = 0; _i < TotalLanes; _i++) {
      pendingUpdatersLaneMap.push(new Set());
    }
  }
  {
    switch (tag) {
      case ConcurrentRoot:
        this._debugRootType = hydrate ? 'hydrateRoot()' : 'createRoot()';
        break;
      case LegacyRoot:
        this._debugRootType = hydrate ? 'hydrate()' : 'render()';
        break;
    }
  }
}
```

ReactDOMRoot