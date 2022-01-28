---
title: React Suspense实现图片懒加载
date: 2022-01-28 11:44:39
tags:
- react
categories:
- 前端
keywords:
- react
- suspense
- lazy
top_img: /img/post/react-suspense.png
cover: /img/post/react-suspense.png
---

### 前沿
React发展至今，针对页面复杂渲染会导致卡顿、组件渲染waterfall的不合理性、不能及时响应用户操作等，提出了一些很棒的设计思想和解决方案：
+ CPU密集型问题（渲染卡顿，不及时响应用户事件），16版本推出了fiber架构，重构了以前的stack架构。
+ I/O密集型问题（组件渲染waterfall，本来应该并行的却变成了序列化串行），16版本提出了Suspense机制。
最近在在项目中尝试区别常规方式，用Suspense方式实现了图片懒加载的功能，顺带把原理记录下。

### 什么是Suspense
如果没有概念，那么我们可以先看下官网介绍的[Suspense](https://reactjs.org/docs/concurrent-mode-suspense.html)。
Suspense的官网定义：`Suspense lets your components “wait” for something before they can render`。
不过多叙述，简单看来：
+ 这是个实验性的功能，并不推荐在开发环境中使用。
+ 在我看来，Suspense就是让我们的组件在可渲染之前，在执行某些操作（比如异步请求数据等）的同时，先渲染预先准备的内容的**机制**。
+ 以目前情况来说，大部分场景是用在懒加载模块上的。
+ 也能单独用，但要顺从Suspense的机制。

> 从官网的定义上还可以看到Suspense也不是传统意义上的fetch库，虽然表现给人感觉很像。

Suspense的三个不是：
+ 不是数据获取的一种实现。
+ 不是一个可以直接用于数据获取的客户端。
+ 它不是数据获取与视图层代码耦合。

为什么我觉得Suspense是机制，因为如果真的是需要在渲染之前因为数据等没准备好，完全可以采用`loading ? <Loading /> : <Component />`这种形式，但Suspense是要**解决那些本可以并发的异步操作（包含Promise的组件），变成序列化串行（waterfall）的问题，也就是上面提到的I/O密集型问题，而不单单是组件的显示内容问题**。

### 传统方式 vs Suspense
如果一个组件的渲染依赖异步数据的请求，我们一般会这么做：
+ **Fetch-on-render（渲染之后获取数据）**: 先开始渲染组件，每个完成渲染的组件都可能在它们的effects或者生命周期函数中获取数据。这种方式经常导致`waterfall`问题。
+ **Fetch-then-render（接收到全部数据之后渲染）**：先尽早获取下一屏需要的所有数据，数据准备好后，渲染新的屏幕。但在数据拿到之前，我们什么事也做不了。

如果引入Suspense，那会是怎么样呢：
+ **Render-as-you-fetch（获取数据之后渲染，如：使用了 Suspense 的 Relay）**：先尽早获取下一屏需要的所有数据，然后立刻渲染新的屏幕——在网络响应可用之前就开始。在接收到数据的过程中，React迭代地渲染需要数据的组件，直到渲染完所有内容为止。

这里不展开对比的详细过程，官网的例子已经阐述的很明确了，具体的对比可以参考官网：[传统方式 vs Suspense的官网demo](https://reactjs.org/docs/concurrent-mode-suspense.html#traditional-approaches-vs-suspense)

### 常规使用场景
目前，大部分项目在使用Suspense的场景是代码分割，也就是`import`引入模块。
此时可以搭配`React.lazy`，此函数能让你像渲染常规组件一样处理动态引入（的组件）。
```javascript
import { lazy, Suspense } from 'react';

const OtherComponent = lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```
这种动态模块引入场景下，`Suspense`配合`lazy`，`import`就能发挥出很好的加载效果。

### 如何玩出花来
如果我们单独在组件外围包上Suspense，有可能什么也不会发生（没有Promise），因为Suspense需要children的Promise来触发，然后再操作结束返回结果后再正确渲染children。
所以我们想单用Suspense，那就需要对其children组件做一些处理。
+ children组件内要有Promise。
+ children组件的Promise要在pendding状态抛出，让Suspense接收。
+ Promise成功后，Suspense会重新渲染children组件，是的，重新渲染。
+ 重新渲染的children组件不再执行Promise，因为需要请求的数据已经有了。
满足上述步骤，就可以在自己的组件上包上一层Suspense，来达到请求的时候展示fallback。
> 最后的不执行Promise很重要，不然会循环渲染子组件。可以把请求过程放在全局，也可以靠作用域通过使用变量来控制执行，不管何种方式，目的就是避免再次执行Promise。

### 传统方式实现图片懒加载
传统的图片懒加载组件，我们一般通过组件内的变量控制，如果图片没加载完，就显示loading，加载完就显示img，代码如下：
```javascript
function LazyImg(props) {
  const { url, alt } = props;
  const [loaded, setLoaded] = useState(false);
  const [src, setSrc] = useState('');

  useEffect(() => {
    const myImg = new Image();
    myImg.onload = () => {
      setSrc(url)
      setLoaded(true);
    };
    myImg.src = url;
  }, []);

  return (
    { loaded ? <img src={src} alt={name}/> : <div>loading</div> }
  );
}
```
这是比较正常的实现思路，代码也很简单，不用多解释。

### Suspense实现图片懒加载
按上述单用Suspense的原理，需要对被包裹的子组件进行一步步的拆分。可以先提前看下[Suspense方式的图片懒加载完整代码](https://codesandbox.io/s/vigorous-almeida-0minv?file=/src/lazyImg.tsx)。

#### 剥离fetch
首先，我们把图片fetch的操作剥离出来
```javascript
function preFetchImg(src: string) {
  return new Promise<string>((resolve) => {
    const img = new Image();
    img.onload = function () {
      resolve(src);
    };
    img.src = src;
  });
}
```

#### 处理Promise的wrap函数
然后写一个能让Promise在不同状态，以不同形式处理的wrap函数。
```javascript
const wrapPromise = (promise: Promise<string>) => {
  let status = "pending";
  let result = "";
  const suspender = promise.then(
    (r) => {
      status = "success";
      result = r;
    },
    (e) => {
      status = "error";
      result = e;
    }
  );
  return {
    read() {
      if (status === "pending") {
        throw suspender;
      } else if (status === "error") {
        throw result;
      } else if (status === "success") {
        return result;
      }
    }
  };
};
```
+ 当Promise处于请求中状态为pending的时候，会`throw suspender`，这就等于告诉Suspense执行fallback的渲染。
+ 当Promise请求返回后，suspender会根据Promise请求后的结果，去更改状态和结果，并以闭包的形式保存下来。此时，Suspense内部会判断请求success，从而重新渲染children组件。
+ 如果这个被包裹Promise后的wrap函数放在全局执行的话，那之后即使再重新渲染子组件，只需要调用这个函数返回的对象的read方法（此时状态不是pendding，所以不会再被Suspense捕获），就能获取请求得到的数据。
+ 如果wrap函数不放在全局执行，那就需要修改这个wrap函数，内部耦合变量，来判断请求已经ok，直接拿请求后的结果即可。
> 这样给人的感觉就是loading的渲染和异步请求看着像并行的，等数据请求完成后，顺利成章的渲染子组件。

在这个业务场景中，因为图片的url很多，所以我采用了内部耦合变量的方式，将所有东西包裹在一个组件中。

#### 组件初现
LazyImg内部使用了Suspense，而且创建了load的ref（下面会用到）。所以在外部使用的时候，不会感觉到Suspense的存在。
```javascript
function LazyImg(props: IProps): JSX.Element {
  const { src, alt } = props;
  const load = useRef(false);

  const Img = (props: IProps) => {
    // 下面会提到
  }

  return (
    <div>
      <Suspense fallback={<div className="loading">loading</div>}>
        <Img src={src} alt={alt} />
      </Suspense>
    </div>
  );
}
```
比较常规，我们看最后的重点：Img组件。

#### 防止重复渲染的Img组件
有了上述返回Promise的fetch函数和包裹Promise的wrap函数，我们就可以将这两部分填进Img组件，其中需要注意的就是防止重复请求。
```javascript
const Img = (props: IProps) => {
  const { src, alt } = props;
  const wrapPromise = (promise: Promise<string>) => {
    let status = "pending";
    let result = "";
    const suspender = promise.then(
      (r) => {
        status = "success";
        result = r;
        load.current = true; // 这个再更上层。
      },
      (e) => {
        status = "error";
        result = e;
      }
    );
    return {
      read() {
        if (status === "pending") {
          throw suspender;
        } else if (status === "error") {
          throw result;
        } else if (status === "success") {
          return result;
        }
      }
    };
  };
  if (!load.current) {
    wrapPromise(preFetchImg(src)).read();
  }
  return <img src={src} alt={alt} />;
};
```
从这里可以看出：
+ 此处用LazyImg定义的ref来标识图片url是否已经加载完成，并且只有在第一次的时候调用了wrap函数的read方法，所以是在图片url未加载完的时候通知Suspense去处理fallback渲染。
+ 当加载完成，Suspense重新渲染img的时候，因为有了标识符，就不会在执行wrap函数，这也就避免了重复渲染，从而正确地渲染出img。

至此，Suspense方式的图片懒加载就可以正常使用了。完整代码可以参考[Suspense方式的图片懒加载完整代码](https://codesandbox.io/s/vigorous-almeida-0minv?file=/src/lazyImg.tsx)。

### 总结
通过上述例子，说明Suspense即可以结合import和lazy来使用，也可以单独使用，但是要改写子组件的请求方式，最后要记得一点，防止子组件的重复渲染。
