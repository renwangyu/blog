---
title: react运行过程随记
date: 2021-11-07 12:48:42
tags:
- react
categories:
- 前端
keywords:
- react
top_img: /img/post/react-running.png
cover: /img/post/react-running.png
---

+ ReactDOM.render
  - 判断container容器是否合法
  - 容器是否标记为root

+ legacyRenderSubtreeIntoContainer
  - topLevelUpdateWarnings: 判断是否是合法的container，否则会抛出warning
  - warnOnInvalidCallback$1: 判断render的回调函数是否合法
  - 检查有没有root，首次渲染是undefined，需要通过legacyCreateRootFromDOMContainer创建
  - new ReactDOMBlockingRoot对象，内部的_internalRoot是createRootImpl的返回值，该函数内部调用createFiberRoot，创建new FiberRootNode作为返回值，并且把首次的current指向uninitializedFiber，相同的，把uninitializedFiber的stateNode设为新建的FiberRootNode。再初始化update队列。
  ```javascript
  function createFiberRoot(containerInfo, tag, hydrate, hydrationCallbacks) {
    var root = new FiberRootNode(containerInfo, tag, hydrate);
    var uninitializedFiber = createHostRootFiber(tag);
    root.current = uninitializedFiber;
    uninitializedFiber.stateNode = root;
    initializeUpdateQueue(uninitializedFiber);
    return root;
  }
  ```
  - root创建完后，用markContainerAsRoot把container标记为root，对应上面的ReactDOM.render中的判断。
  - 继续调用listenToAllSupportedEvents方法在container监听所有的方法。
  - 最终返回的结果会挂在container._reactRootContainer上。（container._reactRootContainer应该就是RootFiber，其属性_internalRoot则就是FiberRoot）
  ```javascript
  // Initial mount
  root = container._reactRootContainer = legacyCreateRootFromDOMContainer(container, forceHydrate);
  fiberRoot = root._internalRoot;
  ```
  - 然后包装下传入的callback，今后处理。
  - 再用unbatchedUpdates来调用updateContainer。这里面处理update，用createUpdate配合优先级创建，并把callback等参数塞入创建的update，然后scheduleUpdateOnFiber开启调度。
  - 在scheduleUpdateOnFiber中开启performSyncWorkOnRoot，传入FiberRootNode，开始重要的reconcile。

+ performSyncWorkOnRoot
  - 先用flushPassiveEffects函数来把之前的effects给一次性调用了。
  - 调用renderRootSync，感觉是从入口进入了。开头判断了workInProgressRoot是否一致，这里有点不清楚原因。
  ```javascript
  var prevDispatcher = pushDispatcher();
  // If the root or lanes have changed, throw out the existing stack
  // and prepare a fresh one. Otherwise we'll continue where we left off.

  if (workInProgressRoot !== root || workInProgressRootRenderLanes !== lanes) {
    prepareFreshStack(root, lanes);
    startWorkOnPendingInteractions(root, lanes);
  }
  ```
  - 进入workLoopSync，好戏开场。
  ```javascript
  do {
    try {
      workLoopSync();
      break;
    } catch (thrownValue) {
      handleError(root, thrownValue);
    }
  } while (true);
  ```
  - 用workInProgress来循环调用performUnitOfWork，直到构造完所有节点。
  ```javascript
  function workLoopSync() {
    // Already timed out, so perform work without checking if we need to yield.
    while (workInProgress !== null) {
      performUnitOfWork(workInProgress);
    }
  }
  ```
  - performUnitOfWork中开始fiber的『递』操作：beginWork（深度优先遍历，会为遍历到的每个Fiber节点生成他的所有子Fiber并返回第一个子Fiber，这个子Fiber将赋值给workInProgress）。在其中会根据fiber的tag来做不同的处理。随后会调用reconcileChildren来继续深入子节点。
  - reconcileChildren中继续调用reconcileChildFibers。
  - 调用createFiberFromElement，当中的createFiberFromTypeAndProps会调用createFiber
  ```javascript
  function createFiberFromElement(element, mode, lanes) {
    var owner = null;
    {
      owner = element._owner;
    }
    var type = element.type;
    var key = element.key;
    var pendingProps = element.props;
    var fiber = createFiberFromTypeAndProps(type, key, pendingProps, owner, mode, lanes);
    {
      fiber._debugSource = element._source;
      fiber._debugOwner = element._owner;
    }
    return fiber;
  }
  ```
  ```javascript
  function createFiberFromTypeAndProps(type/*React$ElementType*/, key, pendingProps, owner, mode, lanes) {
    // ...
    var fiber = createFiber(fiberTag, pendingProps, key, mode);
    fiber.elementType = type;
    fiber.type = resolvedType;
    fiber.lanes = lanes;
    {
      fiber._debugOwner = owner;
    }
    return fiber;
  }
  ```
  - 当遇到叶子结点，也就是没有child的fiber时，就会进入『归』操作：completeUnitOfWork。处理完会接着处理该叶子节点的sibling fiber。如果没有sibing fiber，那么就会处理return fiber。
  - 循环往复上面的『递』和『归』逻辑。