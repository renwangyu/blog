---
title: 用react实现弹性定位滚动
date: 2021-08-24 17:19:31
tags:
- react
categories:
- 前端
keywords:
- react scroll
top_img: /img/post/css3-scroll.jpg
cover: /img/post/css3-scroll.jpg
---

### 前言
滚动(scroll)对于前端开发而言，应该都已经习以为常了。超出固定宽高，就会出现相应的滚动条；要去除也很简单，overflow设置下就行了。但在实际场景里，也会有一些比较特殊的滚动应用，比如实现下图所示的弹性吸附滚动，并且每一个元素的高度都是不同的，也就是无法确定固定的滚动高度，但又要每次滑动都能吸附到相应元素的顶部，我们应该怎么做呢？
![弹性滚动](/img/post/elastic.gif)

### css3能实现么
如果在网上搜吸附，会有很多关于用js控制`position: fixed`或`position: sticky`的文章。但细细看来，仍然不是我们所希望的功能。
> 注：sticky是**实验性**的。

那么我们来搜罗下常用的scroll的css属性，但不会深挖细节，只是看看能否实现我们想要的。

#### -webkit-scrollbar
`-webkit-scrollbar`是由七个伪元素组成的属性：
+ ::-webkit-scrollbar：整个滚动条
+ ::-webkit-scrollbar-button：滚动条上的按钮（上下箭头）
+ ::-webkit-scrollbar-thumb：滚动条上的滚动滑块
+ ::-webkit-scrollbar-track：滚动条轨道
+ ::-webkit-scrollbar-track-piece：滚动条没有滑块的轨道部分
+ ::-webkit-scrollbar-corner：当同时有垂直和水平滚动条时交汇的部分
+ ::-webkit-resizer：某些元素的交汇部分的部分样式（类似textarea的可拖动按钮）

可以看到，这个属性系列，主要是定制滚动条的外观效果，和弹性行为没任何关系，不是我们想要的。

#### scroll-behavior
很多时候为了让页面滚动更平滑，会在全局写这么个样式：
```css
html {
  scroll-behavior: smooth;
}
```
+ auto：滚动框立即滚动
+ smooth：滚动框通过一个用户代理定义的时间段使用定义的时间函数来实现平稳的滚动，用户代理平台应遵循约定，如果有的话

所以这个属性，类似与js中的一些和滚动相关的原生API：
```javascript
window.scrollBy({
  behavior: 'smooth',
});
window.scrollTo({
  behavior: 'smooth',
});
document.querySelector('#scroll').scrollIntoView({ 
    behavior: 'smooth' 
});
```
主要就是为了让滚动看起来更平滑，所以也无法实现我们的功能。

#### overscroll-behavior
在应用中，我们会碰到如果父子均有滚动条的场景，在子滚到边界后，在滚动，就会滚动父的，这是浏览器的默认情况。但在很多情况下这可能并不是我们想要的，我们期望的可能是：滚动到底部，滚动就停止。
> 这种滚动到边界，不停止继续滚动后面的内容的现象，称为`滚动链接(Scroll Chaining)`。

很多同学常见的想法就是在滚动事件里，用阻止冒泡或取消默认行为的方法，或者动态判断到达边界后改变页面的overflow属性。
> 注：js当然可以，但要记住，此时要处理的不是scroll事件，而是每当用户使用鼠标滚轮或触摸板时触发的**[wheel](https://developer.mozilla.org/en-US/docs/Web/Events/wheel)**事件

此时就可以考虑`overscroll-behavior`，这个属性可以控制一个容器或页面body容器滚动时发生的默认行为。可以使用这个属性取消滚动链接、禁用、自定义下拉刷新，禁用在iOS上的回弹效果等。
+ auto：其默认值。元素（容器）的滚动会传播给其祖先元素。有点类似JavaScript中的冒泡行为一样。
+ contain：阻止滚动链接。滚动行为不会传播给其祖先元素，但会影响节点内的局部显示。例如，Android上的光辉效果或iOS上的回弹效果。当用户触摸滚动边界时会通知用户。
> 注：overscroll-behavior: contain在html元素上使用，可以阻止导航滚动操作

+ none：和contain一样，但它也可以防止节点本身的滚动效果

虽然属性很棒，但仍然不是我们想要的。

#### -webkit-overflow-scrolling
这个属性控制元素在移动设备上是否使用滚动回弹效果。
+ auto：不支持momentum-based scrolling，当使用者结束滑动手势、手指离开屏幕后，页面滚动立刻停止。
+ touch：支持momentum-based scrolling，当使用者结束滑动手势、手指离开屏幕后，页面滚动仍会持续一阵子，依据使用者滑动的速度及力道随时间渐渐停止。

> 注：iOS预设了支持momentum-based scrolling。
嗯，回弹效果？看着有那么点意思，但仔细想想，这个回弹只是让页面在移动设备上滚动效果不那么生硬，也不能满足我们的需要。

#### pointer-events
这个属性设置为none，可以干掉元素的touch事件。显然对我们这个需求没有任何用。
> 注：如果元素有了这个属性为none，其实就好像是变成了一个影像，本体已经不在那，所以什么click，touch事件就没用了。另外引用zxx大神的结论，pointer-events: none提高页面滚动时候的绘制性能是不准确的。

### 回归js，用react实现
上面整理了下css的关于scroll的属性，貌似都不能解决我们的问题，那就只能放大招，用js来实现了。
如果用js来实现滚动，方式可以归为以下几种：
+ 监听scroll或wheel事件，实时控制scrollTop或相关属性。
+ 监听touch等事件，实时计算元素的position的top或left位置。
+ 监听touch等事件，用计算transform的位置，替代计算元素的position位置。

从性能和难易度，毫无疑问，我们选择第三种。

#### 思路设计
根据我们的需求，做了如下技术点拆分：
+ 实现一个通用react组件。
+ 包裹外部传入的children元素，元素高度可以不固定。
+ children元素根据上下互动一个个展示，不能跳过。
+ 监听touch相关事件，能区分上下滑动。
+ 并在滑动中能获取滚动到下一个元素的高度值。
+ 第一个和最后一个元素要有边界条件限制，不能再上滑或下滑。

#### 先看外框ElasticScroll
首先，我们需要定义以下几个状态来记录我们的应用数据：
+ 记录滚动偏移值：const [offsetY, setOffsetY] = useState(0)
+ 传入的元素总数：const [total, setTotal] = useState(children.length)
+ 当前滚动的元素index：const [currIndex, setCurrIndex] = useState(0)
+ 手指触摸屏幕的Y坐标：const [startPageY, setStartPageY] = useState(0)
+ 滚动画布的dom：const wrapRef = useRef()
+ 一个用来放元素dom数组的空间：const domsRef = useRef([])

```javascript
function ElasticScroll(props) {
  const { className, children, limit = 30 } = props;
  const [offsetY, setOffsetY] = useState(0);
  const [total, setTotal] = useState(children.length);
  const [currIndex, setCurrIndex] = useState(0);
  const [startPageY, setStartPageY] = useState(0);
  const wrapRef = useRef();
  const domsRef = useRef([]);
  // ...
  const elasticElemList = children; // 待处理
  return (
    <div
      onTouchMove={handleTouchMove}
      onTouchStart={handleTouchStart}
      onTouchEnd={handleTouchEnd}
    >
      <div className={styles.wrap} ref={wrapRef}>
        { elasticElemList }
      </div>
    </div>
  )
}
```

然后轮到监听事件，此处我们需要实现监听的touchstart，touchmove和touchend事件。
+ touchstart比较简单，就是记录用户`初始触摸点Y坐标`，然后把画布元素的transition设置为空。

```javascript
const handleTouchStart = (e: any) => {
  setStartPageY(e.changedTouches[0].pageY);
  wrapRef.current.style.transition = '';
}
```
+ 再看touchmove，根据滑动后的Y坐标的delt值判断上下方向，并且根据当前元素索引来控制边界条件(第一个元素下滑或最后一个元素上滑默认不处理)。在把结合`上一次的画布偏移值`加上`本次Y方向delt值`，计算得到的值实时放入画布dom的transfom值，以达到滑动中实时的有滚动的效果。
> 注：加入防抖是为了控制触发频率，用transslate3d是为了开启硬件加速。

```javascript
const handleTouchMove = debounce((e) => {
    const dy = e.changedTouches[0].pageY - startPageY; // down：>0，up：<0
    const isDown = dy > limit;
    // 当第一张的时候，下滑没有反应
    if (currIndex === 0 && isDown) {
      return;
    }
    // 当最后一张的时候，上滑没有反应
    if (currIndex === total - 1 && !isDown) {
      return;
    }
    const transY = dy + offsetY > 0 ? 0  : dy + offsetY;
    wrapRef.current.style.transform = `translate3d(0, ${transY}px, 0)`;
  })
```

+ 最后是touchend，这里主要是触发结束的位置如果大于limit值，触发弹性滚动，`元素索引值递增或递减`，并记录下来，此处要防止越界。再把画布元素的transition附上值，记录下新的画布Y偏移值(`上一次的偏移值`加上`本次的Y坐标delt值`)。

```javascript
const handleTouchEnd = (e) => {
  const delt = e.changedTouches[0].pageY - startPageY;
  if (delt > limit) { // down
    const index = currIndex - 1;
    setCurrIndex(index < 0 ? 0 : index);
  }
  if (delt < -limit) { // up
    const index = currIndex + 1;
    if (index > total - 1) {
      return;
    }
    setCurrIndex(index > total ? total : index);
  }
  wrapRef.current.style.transition = 'transform .3s';
  setOffsetY(offsetY + delt);
}
```
再来我们通过依赖currIndex的变化，来实时计算当前元素之前(不含当前元素)的所有元素高度和，作为画布dom的新偏移值，打到弹性到索引元素的效果。
```javascript
useEffect(() => {
  let transY = 0;
  for (let i = 0; i < currIndex; i++) {
    transY -= domsRef.current[i].offsetHeight;
  }
  wrapRef.current.style.transform = `translate3d(0, ${transY}px, 0)`;
  setOffsetY(transY)
}, [currIndex]);
```
至此，外框的逻辑代码基本完成。但此时要注意，我们的`elasticElemList`元素集合还没有经过处理，光靠children是无法获取元素dom的高度的，此时我们需要再来个wrap组件为我们做这个每个children元素引用的事。

#### 再看内裹ElasticWrapper
ElasticWrapper的主要作用就是获取当前元素的dom实例，我们来看看是怎么包裹的：
```javascript
const ElasticWrapper = forwardRef((props, ref) => {
  const { index, callback } = props;
  ref = useMemo(() => {
    return ref || createRef();
  }, [ref]);
  useEffect(() => {
    const $dom = findDOMNode(ref.current)
    callback({ index, $dom });
  }, [ref]);
  return (
    <Box ref={ref} {...props} />
  );
});
```
此处的createRef是用来保证ref有个默认值，反常规使用，在function组件里效果一样。然后重点是在useEffect里的传入的回调函数callback，这个就是把`当前的元素在scroll里的索引和实例`回传给ElasticScroll关键步骤。我们再回到ElasticScroll中，看看这个callback是什么：
```javascript
const cb = (params) => {
  const { index, $dom } = params;
  domsRef.current[index] = $dom;
}
```
至此，ElasticScroll里的children，通过ElasticWrapper包裹后，把`自身实例放到了domsRef空间`保存起来，这样一来，之后的touch滑动，在ElasticScroll里的useEffect需要计算的元素高度依据，就有了~这样就能实现弹性的吸附效果了。
最后在渲染中，这个ref和传递的props又给了Box组件，那么我们来看看Box是什么：
```javascript
class Box extends Component {
  render () {
    return this.props.children;
  }
}
```
可以看到，只是一个class组件而已，因为function组件没有实例，所以这么写，当然了，如果你用forwardRef，也是可以的，这里不赘述。

### 所有代码
ElasticScroll.jsx
```javascript
import clsx from 'clsx';
import { debounce } from 'lodash';
import { Children, useEffect, useRef, useState } from 'react';
import styles from './ElasticScroll.module.scss';
import ElasticWrapper from './ElasticWrapper';

function ElasticScroll(props) {
  const { className, children, limit = 30 } = props;
  const [offsetY, setOffsetY] = useState(0);
  const [total, setTotal] = useState(children.length);
  const [currIndex, setCurrIndex] = useState(0);
  const [startPageY, setStartPageY] = useState(0);
  const wrapRef = useRef();
  const domsRef = useRef([]);

  useEffect(() => {
    let transY = 0;
    for (let i = 0; i < currIndex; i++) {
      transY -= domsRef.current[i].offsetHeight;
    }
    wrapRef.current.style.transform = `translate3d(0, ${transY}px, 0)`;
    setOffsetY(transY)
  }, [currIndex]);
  const handleTouchStart = (e) => {
    setStartPageY(e.changedTouches[0].pageY);
    wrapRef.current.style.transition = '';
  }
  const handleTouchMove = debounce((e) => {
    const dy = e.changedTouches[0].pageY - startPageY; // down：>0，up：<0
    const isDown = dy > limit;
    // 当第一张的时候，下滑没有反应
    if (currIndex === 0 && isDown) {
      return;
    }
    // 当最后一张的时候，上滑没有反应
    if (currIndex === total - 1 && !isDown) {
      return;
    }
    const transY = dy + offsetY > 0 ? 0  : dy + offsetY;
    wrapRef.current.style.transform = `translate3d(0, ${transY}px, 0)`;
  })

  const handleTouchEnd = (e) => {
    const delt = e.changedTouches[0].pageY - startPageY;
    if (delt > limit) { // down
      const index = currIndex - 1;
      setCurrIndex(index < 0 ? 0 : index);
    }
    if (delt < -limit) { // up
      const index = currIndex + 1;
      if (index > total - 1) {
        return;
      }
      setCurrIndex(index > total ? total : index);
    }
    wrapRef.current.style.transition = 'transform .3s';
    setOffsetY(offsetY + delt);
  }

  const cb = (params: any) => {
    const { index, $dom } = params;
    domsRef.current[index] = $dom;
  }

  const elastElemList = Children.map(children, (child, index) => {
    return (
      <ElasticWrapper index={index} callback={cb}>
        { child }
      </ElasticWrapper>
    );
  });

  return (
    <div
      className={clsx({ [styles.container]: true, [className]: !!className })}
      onTouchMove={handleTouchMove}
      onTouchStart={handleTouchStart}
      onTouchEnd={handleTouchEnd}
    >
      <div className={styles.wrap} ref={wrapRef}>
        { elastElemList }
      </div>
    </div>
  )
}

export default ElasticScroll;
```
ElasticScroll.module.css
```css
.container {
  height: 100vh;
  overflow: hidden;
}
.wrap {}
```
ElasticWrapper.jsx
```javascript
import { Component, createRef, forwardRef, useEffect, useMemo } from 'react';
import { findDOMNode } from 'react-dom';

class Box extends Component {
  render () {
    return this.props.children;
  }
}

const ElasticWrapper = forwardRef((props, ref) => {
  const { index, callback } = props;
  ref = useMemo(() => {
    return ref || createRef();
  }, [ref]);
  useEffect(() => {
    const $dom = findDOMNode(ref.current)
    callback({ index, $dom });
  }, [ref]);
  return (
    <Box ref={ref} {...props} />
  );
});

export default ElasticWrapper;
```

### 总结
我们通过实现一个弹性吸附滚动组件，搜罗了常用scroll的css属性，在无法用css实现的情况下，用react达到效果。当然，这个组件还可以退化为模拟常规滚动（但没有-webkit-overflow-scrolling: touch的效果）。还有一些对于hooks中ref的骚操作，也可以记录下，今后活用。

### 参考文章
[改变用户体验的滚动新特性](https://zhuanlan.zhihu.com/p/41687572)