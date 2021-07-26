---
title: 前端面试笔记：vue
date: 2021-07-24 13:40:01
tags:
- 面试
categories:
- 前端
keywords: vue
top_img:
---

### vue性能优化点
+ functional（函数化组件）：vue在进行patch更新的时候，如果patch到组件vnode，就会递归执行子组件初始化，而匹配到函数化vnode，会直接通过render函数生成对应的vnode。相比之下少了递归的过程，就提升了运行性能
+ 子组件拆分：当组件重新渲染时，会反复执行其中的函数，但是函数只需要执行一次，而vue组件更新是组件级别的，所以当我们把函数抽离成组件时，即使父组件更新，因为子组件没有更新点，所以不会重新渲染
+ 局部变量：每次访问响应式变量，会触发其getter方法，进行一次依赖收集过程。反复访问就会重复这个比较复杂的过程，导致性能下降，所以采用局部变量的模式就能只执行一次依赖收集，提升性能
+ v-if/v-show：v-if在解析后是三目运算符，也就是初始化时对v-if为false的不会进行初始化（render-vnode-patch）操作的，v-show的话会对为false进行初始化，但是随后通过dom上添加display属性进行控制。当遇到频繁更新时，修改css属性比反复创建dom的成本会低很多
+ keepalive：keepalive会在组件经过第一次渲染后缓存其vnode和dom的，当二次切入时，直接拿缓存渲染，不需要走render和patch的。但是会增加内存负担