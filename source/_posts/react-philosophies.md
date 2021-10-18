---
title: React代码哲学【译】
date: 2021-10-17 15:37:10
tags:
- react
categories:
- 前端
keywords:
- react
top_img: /img/post/react-philosophies.png
cover: /img/post/react-philosophies.png
---

## 前言
最近偶然发现一篇关于React代码经验的文章，感觉不错。节选了部分并精简其内容，翻译记录，有则改之无则加勉。
原文链接：[react-philosophies](https://github.com/mithi/react-philosophies)

## 🧘 基本认知
### 要知道无论什么时候电脑总是比你聪明
+ 严格地用`ESLint`去分析你的代码，开启`rule-of-hooks`和` exhaustive-deps`这两条规则去捕获React特有的错误。
+ 开启`strict`模式，都2021了。
You can try "The latest ref pattern" to keep your callbacks always up-to-date without unnecessary rerenders.
+ 真实地对待依赖(dependencies)。在使用`useMemo`，`useCallback`和`useEffect`时，修复`exhaustive-deps`抛出的warnings或errors。你可以尝试用[The latest ref pattern](https://epicreact.dev/the-latest-ref-pattern-in-react/)的方法，在避免不必要渲染的同时，始终保持你的回调处于最新状态。
> 译者注：ref.current不应该出现在任何dependencies中，会导致莫名奇妙的bug。ref也不应该出现在任何dependencies中，因为它是个不变的object，不会引起变化。

+ 当在用`map`渲染组件的时候，总是要加上属性[key](https://epicreact.dev/why-react-needs-a-key-prop/)。
+ 总是在最顶层位置调用hooks。不要在循环，条件或内嵌函数里调用。
+ 要理解这个warning："Can't perform state update on unmounted component"(不要在未挂在的组件上执行状态更新)。[facebook/react/pull/22114](https://github.com/facebook/react/pull/22114)
> 译者注：主要是提醒开发者注意在unmount的时候注销一些行为，以免造成内存泄露。

+ 通过在应用的不同层级增加[错误边界](https://reactjs.org/docs/error-boundaries.html)来防止页面白屏。你也可以通过它们来发送错误到监控服务，例如[Sentry](https://sentry.io/welcome/)。[在React中解析错误](https://www.brandondail.com/posts/fault-tolerance-react)
+ 不随便在控制台中显示error和warning。
+ 随时记得[tree-shaking](https://webpack.js.org/guides/tree-shaking/)。
+ 无需多想，无论何时，都要始终用Prettier或其他工具格式化你的代码。
+ Typescript将提供更便捷的体验。
+ 强烈推荐[Code Climate](https://codeclimate.com/quality/)或类似开源项目，我发现，自动检测代码看上去确实促使我减少我所开发应用所带来的技术债!
+ NextJS是个神器的框架。

### 代码是不可避免的恶魔
#### 添加额外的依赖前，先想想
不用说，你添加了越多的依赖，越多的代码就被带到了浏览器。问问自己，你真的用到了让某个库变得很棒的特性？
🙈 你是否真的需要？看些你可能不需要的依赖/代码的例子：
+ 你是否真的需要`Redux`？可能是需要的，但要记得，React已经是一个[状态管理的库](https://kentcdodds.com/blog/application-state-management-with-react)。
+ 你是否真的需要`Apollo client`？[Apollo client](https://www.apollographql.com/docs/react/)有很多很棒的特性，比如手动规范化。然而，它会显著增加你的打包大小。如果你的应用想使用的特性不单单只能依赖Apollo client，尝试考虑用更小的库，比如`react-query`或`SWR`（或根本不要去用）。
+ 你是否真的需要`Axios`？Axios是个很棒的库，让我们能更简单的使用本地fetch特性。但如果使用Axios的理由仅仅是它有看上去更好的API，那就该考虑是否能使用fetch的封装（比如`redaxios`）。要确认你的应用是否真正用到了Axios的最佳特性。
+ 你是否需要`Lodash`/`underscoreJS`？[你也许不需要Lodash或underscoreJS](https://github.com/you-dont-need/You-Dont-Need-Lodash-Underscore)。
+ 你是否需要`MomentJS`？[你也许不需要MomentJS](https://github.com/you-dont-need/You-Dont-Need-Momentjs)。
+ 你可能不需要用`Context`来做主题（light/dark模式），考虑用`css variables`替代。
+ 你可能不需要用`javascript`。CSS很强大。[你不需要使用javascript](https://github.com/you-dont-need/You-Dont-Need-JavaScript)。

#### 使用多方技术来减少代码而不仅仅是靠`React`
+ 简化[复杂的条件](https://github.com/sapegin/washingcode-book/blob/master/manuscript/Avoid_conditions.md)并尽早退出。
+ 如果没有明显的性能差异，用传统的循环，替换链式的高阶函数（`map`，`filter`，`find`，`findIndex`，`some`等）。

#### 不要自做聪明
"我的软件将来会发生什么？哦是的，也许这个和那个。让我们实现所有这些东西，因为我们正在做这个部分。这样它就不会过时。"
**你不需要它！**只有在你真正需要的时候去实现，永远不要你觉得你需要。

### 让它比你发现它的时候更好
检测代码，并在需要时对它们进行处理。
如果你意识到有些地方是错的，立马修复它。但如果不容易修或一时半会没有时间，至少增加一条能定义问题的简明注释（`FIXME`或`TODO`），确认大家都知道这个问题。这意味着你是这么做的，当其他人遇到类似的问题时，也应该这么做。
🙈 看些很容易发现问题的例子：
+ ❌ Method或Function定义为大量参数的。
+ ❌ 很难懂的Boolean逻辑。
+ ❌ 单文件代码行数过多的。
+ ❌ 语法相同的重复代码（但格式可能不同）。
+ ❌ Function或Method很难理解。
+ ❌ Class或Component定义了很多方法。
+ ❌ 当个方法或函数中代码行数过多的。
+ ❌ 返回大量状态的函数或方法。
+ ❌ 不同的重复代码但有相同的结构（比如变量名可能不同）。

请记住，代码异味并不一定意味着应该更改代码。代码异味只是告诉你，你也许能用更好的方法来实现相同的功能。

### 你可以做的更好
💁‍♀️ **TIP：牢记，你不用把你的`state`作为依赖，因为你可以传入状态回调函数去替代。**
你不用把`setState`（来自`useState`）和`dispatch`（来自`useReducer`）放到你的hooks依赖数组（比如`useEffect`和`useCallback`）。ESLint不会报错，因为React保证了他们的稳定性。
🙈 举例：
```javascript
❌ 不好
const decrement = useCallback(() => setCount(count - 1), [setCount, count])
const decrement = useCallback(() => setCount(count - 1), [count])
✅ 好
const decrement = useCallback(() => setCount(count => (count - 1)), [])
```
💁‍♀️ **TIP：如果你的`useMemo`或`useCallback`没有依赖数组，你可能用错了**
🙈 举例：
```javascript
❌ 不好
const MyComponent = () => {
   const functionToCall = useCallback(x: string => `Hello ${x}!`,[])
   const iAmAConstant = useMemo(() => { return {x: 5, y: 2} }, [])
   /* I will use functionToCall and iAmAConstant */
}
       
✅ 好
const I_AM_A_CONSTANT =  { x: 5, y: 2 }
const functionToCall = (x: string) => `Hello ${x}!`
const MyComponent = () => {
   /* I will use functionToCall and I_AM_A_CONSTANT */
}
```
💁‍♀️ **TIP：用hook封装你自定义上下文，会创建一个更优雅的API**
不仅要让它看起来更好，而且你只需要导入一个东西而不是两个。
🙈 举例：
```javascript
❌ 不好
// you need to import two things every time 
import { useContext } from "react"
import { SomethingContext } from "some-context-package"
function App() {
  const something = useContext(SomethingContext) // looks okay, but could look better
  // blah
}

✅ 好
// on one file you declare this hook
function useSomething() {
  const context = useContext(SomethingContext)
  if (context === undefined) {
    throw new Error('useSomething must be used within a SomethingProvider')
  }
  return context
}
// you only need to import one thing each time
import { useSomething } from "some-context-package"
function App() {
  const something = useSomething() // looks better
  // blah
}  
```
💁‍♀️ **TIP：在写之前想好你的组件该怎么用**
写API很难，[Readme Driven Development](https://tom.preston-werner.com/2010/08/23/readme-driven-development.html)是一个对设计更好APIs有用的技巧。我发现，当我第一次在实现之前编写API（组件将如何使用）时，通常会创建一个设计得更好的组件。

## 🧘 快乐设计

+ 💖 **通过删除冗余状态来避免状态管理的复杂性**
当你的逻辑理由冗余的状态，一些状态就可能不会同步更新：在一个复杂的交互过程中，你可能会忘记更新它们。除了避免同步错误之外，你会发现它也更容易理解，需要的代码也更少。[Don't Sync State. Derive It!](https://kentcdodds.com/blog/dont-sync-state-derive-it)

+ 💖 **给我香蕉，而不是拿着香蕉的猩猩和整个丛林**
Passing primitives is also a good idea if you want to use React.memo for optimization
为避免陷入这个陷阱，传递基本类型（boolean，string，number）是个好主意（当你优化React.memo的时候，传递基本类型也是个好主意）。
一个组件应该知道它该做什么，不用管其他的。组件应该尽可能地在不知道其他组件是什么，且它们能干什么的情况下，与其他组件互相协作。
当我们做到了这样，组件之间将更加解耦，依赖性也将更低。解耦使得在不影响其他组件的情况下，能更容易更改、替换或删除组件。

+ 💖 **保证你的组件小而精简**
  **什么是单一责任原则？**
  > 一个组件应当有且只有一个任务。它应该做到最小可能有用的事。它只有满足其意图的职责。

  具有不同职责的组件很难重用。如果你想重用组件，但又不是它的全部功能，几乎不可能只获取你想要的，它还可能和其他的代码耦合在一起。
  那些制作一件事，并且将其与应用中其他组件分离的组件，将更容易无影响修改和反复重用。
  **如何判断你的组件是单一责任？**
  > 尝试用一句话去描述组件。如果它是单一责任的，那么会很容易描述。如果要使用"和"/"或"，那么他很有可能就不是。

  检查组件所使用的状态，属性和钩子，包括声明在内的变量和方法（通常不要太多）。问问你自己：是否真的需要这些东西一起工作才能满足组件的意图？如果有一些不需要，那么就该考虑把它们移出到其他地方，或者把你的大组件拆分多个小组件。

+ 💖 复制比错误的抽象要容易得多（避免过早/不恰当的泛化）。
+ 避免用合成的方法使属性下沉。
`Context`不是所有状态分享问题的解决方案。
+ 分割大块的`useEffect`到多个独立的小块。
+ 正确抽取逻辑到hooks和辅助函数。
+ 分解大型组件，将其分位`逻辑`和`显示`组件，可能是个好主意（具体要看你的最佳实践判断）。
+ 最好使用基本类型作为`useCallback`，`useMemo`和`useEffect`的依赖。
+ 不要在`useCallback`，`useMemo`和`useEffect`里放置大量依赖。
+ 考虑到精简，如果state的某些value依赖其他state或上一个state的value，作为`useState`的替代方案，可以考虑使用`useReducer`。
+ `Context`不必放在应用的全局。把`Context`尽可能低地放在你的组件树上。同样的，你可以吧你的变量，组件，状态（甚至代码），尽可能近的放在用到它们的地方。

## 🧘 性能tips
+ **如果你觉得它很慢，用一个标准来衡量它。**[React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)的profiler可以帮到你。
+ 在可能开销打的地方使用`useMemo`。
+ 用`React.memo`，`useMemo`和`useCallback`来减少重复渲染，它们不应当有很多依赖，并且依赖应该都是基本类型。
+ 确保`React.memo`，`useMemo`和`useCallback`在做你想做的事。（是否真的阻止了重新渲染？）
+ 在修复渲染之前，先修复渲染缓慢。
+ 把你的state尽可能近地放在使用的地方，这不仅使你的代码阅读更简洁，而且能提高应用的速度。
+ `Context`应逻辑独立，不要把过多的值加入到一个context provider。
如果context中的任意值改变，所有监听这个context的组件都会重新渲染，即使那些组件没有用到实际改变的特定值。
+ 可以通过分隔`state`和`dispatch`函数来优化`context`。
+ 了解[lazy loading](https://nextjs.org/docs/advanced-features/dynamic-import)和[bundle/code splitting](https://reactjs.org/docs/code-splitting.html)。
+ 打包更小的应用通常意味着更快，可以通过可视化代码打包工具例如[source-map-explorer](https://create-react-app.dev/docs/analyzing-the-bundle-size/)或[@next/bundle-analyzer](https://www.npmjs.com/package/@next/bundle-analyzer)。
+ 如果你准备为你的表单使用一个包，推荐用[react-hook-forms](https://react-hook-form.com/)。我觉得它是优秀性能和开发体验的完美平衡。
