---
title: React的哲学【译】
date: 2021-10-17 15:37:10
tags:
- react
categories:
- 前端
keywords:
- react
top_img:
cover:
---

Statically analyze your code with ESLint. Enable the rule-of-hooks and exhaustive-deps rule to catch React-specific errors.


## 最起码的认知程度
### 要知道无论什么时候电脑总是比你聪明
+ 严格地用`ESLint`去分析你的代码，开启`rule-of-hooks`和` exhaustive-deps`这两条规则去捕获React特有的错误。
+ 开启`strict`模式，都2021了。
You can try "The latest ref pattern" to keep your callbacks always up-to-date without unnecessary rerenders.
+ 真实地对待依赖(dependencies)。在使用`useMemo`，`useCallback`和`useEffect`时，修复`exhaustive-deps`抛出的warnings或errors。你可以尝试用[The latest ref pattern](https://epicreact.dev/the-latest-ref-pattern-in-react/)的方法，在避免不必要渲染的同时，始终保持你的回调处于最新状态。
> 注：ref.current不应该出现在任何dependencies中，会导致莫名奇妙的bug。ref也不应该出现在任何dependencies中，因为它是个不变的object，不会引起变化。

+ 当在用`map`渲染组件的时候，总是要加上属性[key](https://epicreact.dev/why-react-needs-a-key-prop/)