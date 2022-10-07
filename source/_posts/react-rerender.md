---
title: React组件重新渲染理解 & 优化大全
date: 2022-10-07 10:00:37
tags:
- react
categories:
- 前端
top_img: /img/post/react-rerender.png
cover: /img/post/react-rerender.png
---

# 前言
React组件的渲染，是React机制中非常重要的部分，出于对性能优化的考虑，在开发过程中我们应该尽可能去避免重新渲染。
但很多时候，由于缺乏对React组件重新渲染的理解和意识，导致代码并未按照设想避免重复渲染。由此可见，要想写出性能高效的React代码，就必须了解组件渲染的机制。
在开始之前，我们要先理解React的渲染时机有两种场景：
+ 初始渲染（initial render）：不可避免的首次渲染
+ 重新渲染（re-render）：由于用户交互或异步请求导致数据更新，引发的渲染

> 重新渲染（re-render）并不会导致功能问题，因为React对组件（比如函数组件）的运行非常快，但会影响页面的性能。

# React组件重新渲染的理解误区
在了解重新渲染之前，首先要认识重新渲染的一些误区，不仅有助于避免写出无效的性能优化，也能为之后理解重新渲染的机制以及优化作对比。

## 避免组件重新挂载，初始渲染
在组件内定义子组件，是非常不好的写法，因为这样每次组件重新渲染，都会创建新的子组件定义，导致子组件每次都是重新挂载（初始渲染）而不是重新渲染，更不要提避免重新渲染的优化了。
而且每次挂载性能消耗要远大于重新渲染。并且因为是挂载，所以自定义子组件内的状态会不断初始化，effect函数会不断触发，因此这种组件定义的写法一定要避免。

![重新挂载而非重新渲染](/img/post/react-rerender/creating-components.png)

## 只要属性不变更，不会触发重新渲染
如果组件不是`记忆组件（memoized components）`，那么其属性的变更与否，就不是影响组件是否重新渲染的因素。
组件属性的变更总是与父组件的重新渲染有关，但并不是父组件更新了，即使子组件的属性没变，是否重新渲染还是要看其他因素。
> 只有记忆API被用到，比如React.memo，useMemo，useCallback等，只有搭配一起使用，属性不变更才可能不触发重新渲染

![非记忆组件属性变更与否不影响是否重新渲染](/img/post/react-rerender/nouse-usememo-usecallback.png)

# React组件重新渲染的主要场景
撇开初始渲染，在优化重新渲染前，我们先将能触发React重新渲染的场景按大类详细描述。

## 组件状态变更导致重新渲染
组件内部触发状态变更，是最常见的方式，作为数据驱动的一种，手动更新也是我们更新组件的主要途径。
![组件内状态变更导致重新渲染](/img/post/react-rerender/state-change.png)

## 父组件渲染导致子组件重新渲染
React是自上而下的单向数据流，因此也符合自上而下的渲染，从另一个角度说，如果父组件发生了重新渲染，那么子组件势必应该要重新渲染。
> 后续我们会继续针对这个场景做更多的优化

![父组件重新渲染导致子组件重新渲染](/img/post/react-rerender/parent-change.png)

## context变更导致引用的组件重新渲染
Context.Provider中的值发生变化，那么引用该context的组件，都会重新渲染。
> 此类重新渲染，即使组件有使用记忆API，也不能被阻止。

![context变更导致的组件重新渲染](/img/post/react-rerender/context-change.png)

## hooks（自定义，链）
函数组件一般都会使用hooks，当组件用了hooks后，组件与hooks内的变更就绑定在了一起。即使自定义hooks内引用其他hooks，这种链式的情况，也是一样，等于组件与hooks链绑定在了一起。
对于自定义hooks的应用，本质上是对组件逻辑复用的一种实现，所以其实与上面说的**组件状态变更**和**context变更**引起重新渲染的场景是一样的。
+ 自定义hooks内的状态变更，触发重新渲染
+ 自动以hooks内引用了context，context变更，触发重新渲染

![hooks导致的组件重新渲染](/img/post/react-rerender/hooks-change.png)

# React组件避免重新渲染的优化方案
上面介绍了重新渲染的四个触发方式，那么我们来看看在写代码的时候，如何能尽可能避免重新渲染。分几个大类，分别阐述其小点：

## 状态隔离，传递不变
这里的关键点在于尽可能小范围触发重新渲染，避免处于节点树上层的渲染，并尽可能在父组件不重复渲染的情况下，活用父组件的内部元素和参数不改变的传递给其子组件，避免不相关的触发重复渲染。

### 状态下放（state down）
这个和我们经常说的组件原子化非常相似，关键就是要让组件的状态精简，尽可能把逻辑和状态放在相关的组件内，而不要上浮到父组件，避免因为一个地方的状态变更，导致子组件的全量更新渲染。

![优化之状态下放](/img/post/react-rerender/state-down.png)

### 子组件作为children传递
这种通常是把需要经常触发重新渲染的组件单独抽离，并将要优化的子组件通过包裹的方式传递，作为children使用。
类似状态下放，其实也是将经常状态变更的组件原子化，区别在于需要优化的子组件是作为children传递。
这里的关键点在于：
+ 组件原子化，经常状态变更的组件抽离出去作为包裹组件，其内部状态变更不影响父组件及其兄弟组件
+ 子组件作为children传入，由于父组件不会重新渲染，子组件不变，所以在重新渲染的包裹组件里，children不发生变化

> 一般有scroll，mousemove事件的组件可以考虑作为包裹组件

![优化之子组件作为children传递](/img/post/react-rerender/as-children.png)

### 子组件作为props传递
原理基本同children传递一样，区别在于children传递的子组件是一个整体，不能在包裹组件里拆分，而props传递可以。

![优化之子组件作为props传递](/img/post/react-rerender/as-props.png)

## 记忆组件
通常，我们只需要使用React为我们提供的API（[React.memo](https://reactjs.org/docs/react-api.html#reactmemo)），就可以把组件变为记忆组件。在属性不变（或没有属性）的情况下，记忆组件可以阻止重新渲染。
> 这里可以对比上述提到的四大触发渲染方式之一的父组件渲染和理解误区中的属性不变不会渲染。

![优化之记忆组件](/img/post/react-rerender/memo.png)

### 带参数的记忆组件
如果是带参数的记忆组件，那么就要保证每个参数都是初始值，我们可以用另一个API（[useMemo](https://reactjs.org/docs/hooks-reference.html#usememo)）来记忆属性值。
一句话描述，**如果想要避免重新渲染，组件memo，属性memo**。

![优化之记忆组件属性不变](/img/post/react-rerender/memo-props.png)

对于useCallback和useEffect等，其依赖项一样可以被memo。

![优化之依赖memo](/img/post/react-rerender/necessary-usememo-dep.png)


### 记忆组件作为props或children传递
这个其实和上述的属性参数初始值一样，既然值不能变，那么如果是组件传递，也需要memo一下，不变才能不重新渲染。

![优化之记忆组件传递](/img/post/react-rerender/memo-as-props.png)

### 思考useMemo的使用
useMemo可以帮我们记忆值，避免在重新渲染中再次计算（尤其是大计算量的值）。
但是useMemo本身也是有性能代价的，并不推荐滥用，应该尽可能去**记忆React的元素**，尤其是那些要作为组件一部分的组件元素。
类似组件元素的缓存。
> 这里可以对比上述提到的理解误区中的避免组件重新挂载，初始渲染

![优化之useMemo记忆组件元素](/img/post/react-rerender/necessary-usememo-complex.png)

## 列表中的重新渲染
我们一般都会在列表的元素上添加key作为属性，以此来减小对DOM的变动开销引发的性能问题。
其实，key的话，只是在diff阶段，减少了因为增删导致的移动等，如果想要避免重新渲染，还是要通过记忆组件，配合属性不变。

![优化之列表](/img/post/react-rerender/use-memo-list.png)

永远不要在key上用随机数，这样会导致父组件重新渲染的是，list全是重新挂载。
+ 使列表的渲染性能拉胯
+ 对非受控组件可能会产生bug
  
![优化之避免key随机数](/img/post/react-rerender/anti-memo-list.png)

## context中的优化

### 记忆Provider的值
因为Provider的值变化会触发所有引用它的组件的更新和重新渲染，所以记忆Provider的值是一个比较常见的思路。
> 尤其是作为root的包裹场景。

![优化之记忆Provider的值](/img/post/react-rerender/context-provider-memo.png)

### 拆分context的功能层
如果一个Context包含的数据很多（比如多个业务维度，多个功能维度，又或者数据和API糅合在一起），那么就会经常发生改动，引用其的组件即使用到的数据并没有改变（即不需要更改重新渲染），但也被迫更新了。因此将其从维度上拆分，是一个好的解决方案。

![优化之拆分context不同功能](/img/post/react-rerender/context-split-api.png)

![优化之拆分context不同数据](/img/post/react-rerender/context-split-data.png)

### context高阶组件记忆选择属性
上面两种是从context的值记忆，以及波及的引用组件范围做了优化，但是如果一旦context变化，那么只要引用它的组件，则不可避免都会重新渲染。
我们可以结合上面记忆组件的方法，用高阶组件去包裹需要优化的组件。类似于一层阻止重新渲染的缓冲层。

![优化之拆分context高阶组件记忆选择属性](/img/post/react-rerender/context-selectors.png)

最后分析下上图：
+ 图左未优化，因为hooks链，所以Component组件势必会因为context的改变而触发重新渲染
+ 图右优化后，因为高阶组件的存在，Component在使用时被缓存了，而且没有直接引用context（改为高阶组件引用了）而是只用了something这个属性，所以一旦context发生变化，高阶组件会发生更改，但是基于上面提到的记忆组件的原则，原来的Component只要something不发生改变，就不会重新渲染

# 总结
上面讲了React的重新渲染的时机，也阐述了我们在实际开发中的误区和优化方向，一般我们比较容易的是对状态和属性的优化，一些children和props的传递技巧也应该在开发中使用。对于context的处理，尽可能做到前两个优化方法，对于高阶组件记忆选择属性，一般可以作为一个通用方法（如果真的常用的话）。多多理解，从根本上了解React的重新渲染的时机，多写优化的代码。

# 参考文献 & 图片来源
[React re-renders guide: everything, all at once](https://www.developerway.com/posts/react-re-renders-guide)
