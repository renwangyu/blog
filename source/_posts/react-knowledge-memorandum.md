---
title: React知识理解拾遗备忘录
tags:
- react
- javascript
categories:
- 前端
top_img: https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTflbVbtodGIurSp_4HfoWv3J7i2e9rkn70eTb96ArwhwQbPdcvIA&s
---

### 前言
自从进入tx，项目都是采用vue，使用多了难免怀旧react。而且react自从16.8.0版本后发布hooks的后，更是风靡前端界,`react-router`，`react-redux` 等陆续拥抱hooks，就可以看出业界对此的赞同。耐不住真香的诱惑，在工作之余，自己用react hooks体验了一把[个人主页](https://renwangyu.com/)的搭建，真的是直呼过瘾！开文记录下学习过程中遇到的点点滴滴，方便今后查询，温故知新。

### hook的理解
react在16.8之前，函数组件仅仅做为呈现静态组件存在。现代web应用强交互性而言，函数组件的能力远远若于类组件，而且没有实例，相比类组件缺乏很多能力`(没有生命周期，ref不能引用等)`。

另一方面，函数式编程在前端业界呼声高涨，其带来的优势迅被众多前端开发者所拥护，如果不知道函数式编程，[这篇文章](https://blog.renwangyu.com/2020/05/19/modern-javascript-concepts-1-functional-programming/)阐明了些基本概念。react从一开始便以状态机的姿态使开发者眼前一亮，单项数据流使业务数据更清晰，便于理解和调试。

基于如此，做为纯函数的函数组件，如何做到像类组件一样的能力，并能使用函数式编程，就成了一件非常有意义的事。令人兴奋的是，react给我们的答案就是hooks。在函数中引入`状态(useState)`和`副作用(useEffect)`的hook，就能让我们像类组件编程一样，又是以函数式在书写，不得不说react连命名都如此贴心和规范，名如其意：在函数式编程中引入状态和副作用！

### 运行理解
组件最终都会被编译成react元素(Vitrual Dom)。react内部会知道何时调用render函数(函数组件本身可以当做render)。如果一个页面是由好多的组件形成，那每一个组件又在独立的状态下显示，函数组件在运行中就好像状态随着时间推移不断发生变化，假设是一部动画，那么每次的render可以理解为一帧。

#### useState
做为官网第一个讲解hook，极为代表性。
```javascript
function hello(props) {
  const [name, setName] = useState(props.name);
  return (
    <div>hello { name }</div>
  )
}
```
而useState返回的name和setName则会被保存在组件对应的`Fiber节点`上，**每个react函数每次执行hook的顺序必须是相同的**，这也正是[react hook的规则](https://zh-hans.reactjs.org/docs/hooks-rules.html)。通俗理解，函数组件在首次render时，会依次运行hooks，然后在`Fiber节点`上创建一个链表结构，表明每个hook。下次再渲染时，会根据次序来确认每个hook，这也就是为什么不能搞乱次序的原因。
```javascript
function demo(props) {
  const [value1, setValue1] = useState(1);
  const [value2, setValue2] = useState(2);
  return (
    <>
      <div>{value1}</div>
      <div>{value2}</div>
    </>
  )
}
```
在执行的时候，由于执行了两次useState，会在`Fiber`上保存一个 **{ value, setValue } -> { value2, setValue2 }** 这样的链表结构。所以在demo里的结构，其实可以理解为一次从另一个存储地方的拷贝。当每次setValue时会重新render，在重新render时候，就会直接在链表上获取变量值，如果次序一致那么就能保证每次获取的变量都是正确的。

### useRef
useRef就好像生成了一个可以存放数据的不可变的「盒子」，然后区别于setState，每次内容改变时候，不会出发重新render。

### 自定义hook
自定义hook也必须以use开头，这是react hook在设计上的约定，也是react知道是hook的原因，然后hook中可以继续使用hook，切记不要在普通函数中使用hook。

### 未完待续。。。