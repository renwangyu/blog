---
title: 简要剖析Promise实现（上）
date: 2020-09-05 14:36:55
tags:
- javascript
categories:
- 前端
top_img: https://i.loli.net/2020/09/06/IcxbDBU6Tv5Hjyw.png
cover: https://i.loli.net/2020/09/06/IcxbDBU6Tv5Hjyw.png
---

# 前言
时光回到2015，当ES6一经问世，便快速的成为前端开发者的首选。为了避免异步调用带来的回调地狱，Promise作为ES6中最重要的异步解决方案特性之一，毫无疑问也成为现代前端开发者的必会技能。虽然我们在日常中不用再去自己实现，但通过实现一把Promise A+规范的部分功能，记录下Promise中的关键点，也能在今后的使用中不在迷茫。

# 撸个MyPromise类来剖析理解Promise
下面一步步通过实现一个简易的MyPromise，来理解Promise中的一些重要概念和用法。

## MyPromise状态
在Promise中，状态是一个很重要的概念，分为如下三种：
+ pedding：等待
+ fulfillment：成功
+ rejection：失败

> Promise实例的状态一经更改便**无法逆转**。

因此在代码中要先定义这三种必要状态常量。
```javascript
const PENDDING = "pedding";
const FULFILLMENT = "fulfillment";
const REJECTION = "rejection";
```

## 构造函数有点忙
回忆一下，当我们在实际应用中使用Promise时，在新建的时候会将一个执行函数传入为参，该函数在执行时会带入Promise的两个改变状态的函数（resolve，reject），我们就暂且称之为状态转换函数~来看看具体的代码。
首先我们定义了一个class，然后初始化一堆内部变量：
```javascript
class MyPromise {
  constructor(executor) {
    this.status = PENDDING; // 当前状态
    this.value = null; // 返回值
    this.msg = null; // 错误原因
    this.resolveQueue = []; // 成功返回回调队列
    this.rejectQueue = []; // 错误返回的回调队列
  }
}
```
其中```resolveQueue```和```rejectQueue```是状态改变后执行的回调函数的队列。
传入的```executor会在构造函数里立马执行```（这也就是为什么Promise传入的函数比后面的代码先执行），并带着resolve和reject作为参数，那接下去就来看看这两个关键状态转换函数的代码。
```javascript
class MyPromise {
  constructor(executor) {
    // ...
    const resolve = (val) => {
      if (this.status === PENDDING) {
        this.status = FULFILLMENT; // 改变状态为成功
        this.value = val;
        while(this.resolveQueue.length) {
          const res = this.resolveQueue.shift();
          res(val);
        }
      }
    };

    const reject = (reason) => {
      if (this.status === PENDDING) {
        this.status = REJECTION; // 改变状态为失败
        this.reason = reason;
        while(this.rejectQueue.length) {
          const rej = this.rejectQueue.shift();
          rej(reason);
        }
      }
    }

    try {
      executor(resolve, reject); // 立即执行~
    } catch(e) {
      reject(e.message);
    }
  }
}
```

## 奇妙的then
当实例化一个Promise后，最关键的就是通过then来调用（避免回调地狱的救世主）。回忆一下，Promise的then接受至多两个参数：Promise的成功和失败情况的回调函数。而且，不论如何调用，一旦Promise实例的状态确定，之后的操作都是一致的。
```javascript
class MyPromise {
  // ...
  then(onResolve, onReject) {
    onResolve = typeof onResolve === 'function' ? onResolve : val => val;
    onReject = typeof onReject === 'function' ? onReject : msg => {
      const reason = msg instanceof Error ? msg.message : msg;
      throw new Error(reason);
    };
    // todo
  }
}
```
then函数的另一个特定是支持```链式调用```，在jquery年代，我们通过返回jquery实例来达到链式调用的目的；但在Promise的then里，我们通过返回一个新的Promise实例来实现这个效果，所以then的返回值就应该如下代码所示，是一个新的实例~
```javascript
class MyPromise {
  // ...
  then(onResolve, onReject) {
    // ...
    return new MyPromise((resolveFn, rejectFn) => {
      const fuifillmentFn = (val) => {
        // todo
      };
      const rejectionFn = (val) => {
        // todo
      };
      if (this.status === PENDDING) {
        this.resolveQueue.push(fuifillmentFn);
        this.rejectQueue.push(rejectionFn);
      }
      if (this.status === FULFILLMENT) {
        fuifillmentFn(this.value)
      }
      if (this.status === REJECTION) {
        rejectionFn(this.reason);
      }
  }
}
```
由此可见，在新返回的实例中，通过```闭包```将新实例的成功回调resolveFn和错误回调refectFn放入fuifillmentFn和rejectionFn中，并根据上一个实例的状态做如下判断
+ pendding时，将回调塞入```上一个实例```的```成功队列```和```失败队列```
+ fulfillment时，```同步```执行```fuifillmentFn```，并传入```上一个实例的value```
+ rejection时，```同步```执行```rejectionFn```，并传入```上一个实例的reason```

那新返回的Promise实例的执行函数resolveFn和rejectFn去哪了？当然是在闭包的fuifillmentFn和rejectionFn当中，以成功状态为例，来看下面的代码实现：
```javascript
const fuifillmentFn = (val) => {
  try {
    const result = onResolve(val);
    if (result instanceof MyPromise) {
      result.then(resolveFn, rejectFn);
    } else {
      resolveFn(result);
    }
  } catch(e) {
    rejectFn(e instanceof Error ? e.message: e);reason
  }
};
```
可以得知，在成功或失败的时候，都会先调用上一个实例相应的回调处理，得到的返回值```result```如果是个Promise实例，则执行它的then，并将```状态转换函数```当做处理回调传入其中，这是非常重要的！

来贴一下then的完整代码，附上过程序号，跟着步骤，我们来总结一下~
```javascript
then(onResolve, onReject) {
  onResolve = typeof onResolve === 'function' ? onResolve : val => val; // stage 1
  onReject = typeof onReject === 'function' ? onReject : reason => {
    reason = reason instanceof Error ? reason.message : reason;
    throw new Error(reason);
  };

  return new MyPromise((resolveFn, rejectFn) => { // stage 2
    const fuifillmentFn = (val) => {
      try {
        const result = onResolve(val); // stage 4
        if (result instanceof MyPromise) {
          result.then(resolveFn, rejectFn); // stage 5
        } else {
          resolveFn(result); // stage 6
        }
      } catch(e) {
        rejectFn(e instanceof Error ? e.message: e);
      }
    };

    const rejectionFn = (reason) => {
      try {
        const result = onReject(reason); 
        if (result instanceof MyPromise) {
          result.then(resolveFn, rejectFn);
        } else {
          rejectFn(result);
        }
      } catch(e) {
        rejectFn(e instanceof Error ? e.message: e);
      }
    };
    // stage 3
    if (this.status === PENDDING) { // 异步
      this.resolveQueue.push(fuifillmentFn);
      this.rejectQueue.push(rejectionFn);
    }
    if (this.status === FULFILLMENT) { // 同步
      fuifillmentFn(this.value)
    }
    if (this.status === REJECTION) { // 同步
      rejectionFn(this.reason);
    }
  });
}
```
分析之前，我们再重提一个理解共识：executor的参数是状态转移函数，主要用于Promise实例状态改变。而then函数是处理函数
+ stage 1：判断传参函数的类型做兼容，这就是为什么then()什么都不传也能透传结果的原因。
+ stage 2：返回新的Promise实例（记住状态转移函数resolveFn和rejectFn），并立即执行传入的函数。
+ stage 3：定义fuifillmentFn和rejectionFn之后（主要依赖闭包），根据当前状态判断：
  - pedding，表示异步逻辑，将成功失败回调放入相应的队列。等待上一个实例的状态转移调用改变了状态，再执行相应的回调。
  - 非pedding，进入同步逻辑，直接调用各自的回调。
+ 以成功为例，不论同步或异步，等resolve（还记得开头定义的reolve嘛，就是那个）调用后，this.value作为参数调用fuifillmentFn。
+ stage 4：先调用在then函数中的处理函数onResolve，并用resolve后的参数，返回处理后的结果。
+ stage 5：如果onResolve处理后返回一个新的Promise实例，意味出现了一个新的节点，因此就需要把这个实例执行then，并把状态转移函数resolveFn和rejectFn当做处理函数传递过去，等到那个实例状态改变调用处理函数的时候，就可以通过resolveFn把stage 2返回的实例的状态改变，这样如果之后继续有then的调用，就能把链式调用继续下去。
  > 如同一个链表a->b，当a返回新的Promise实例，那么就如同a->c->b，那么c就必须next才能到b，所以这里的then就相当于这个next。
+ stage 6：如果不是Promise实例，则直接把stage 2返回的实例改变状态，链式调用得以继续。
通过以上对成功状态的总结，大致就可以理解then函数的调用，关键是如同钩子一般的设计，非常巧妙。

# 总结
通过上述分析，我们就得到了一个简单的MyPromise，代码全体如下：
```javascript
const PENDDING = "pedding";
const FULFILLMENT = "fulfillment";
const REJECTION = "rejection";

class MyPromise {
  constructor(executor) {
    this.status = PENDDING; // 当前状态
    this.value = null; // 返回值
    this.reason = null; // 错误原因

    this.resolveQueue = []; // 成功返回回调队列
    this.rejectQueue = []; // 错误返回的回调队列

    const resolve = (val) => {
      if (this.status === PENDDING) {
        this.status = FULFILLMENT;
        this.value = val;
        while(this.resolveQueue.length) {
          const res = this.resolveQueue.shift();
          res(val);
        }
      }
    };

    const reject = (reason) => {
      if (this.status === PENDDING) {
        this.status = REJECTION;
        this.reason = reason;
        while(this.rejectQueue.length) {
          const rej = this.rejectQueue.shift();
          rej(reason);
        }
      }
    }

    try {
      executor(resolve, reject);
    } catch(e) {
      reject(e.message);
    }
  }

  then(onResolve, onReject) {
    onResolve = typeof onResolve === 'function' ? onResolve : val => val;
    onReject = typeof onReject === 'function' ? onReject : reason => {
      reason = reason instanceof Error ? reason.message : reason;
      throw new Error(reason);
    };
    
    return new MyPromise((resolveFn, rejectFn) => {
      const fuifillmentFn = (val) => {
        try {
          const result = onResolve(val);
          if (result instanceof MyPromise) {
            result.then(resolveFn, rejectFn);
          } else {
            resolveFn(result);
          }
        } catch(e) {
          rejectFn(e instanceof Error ? e.message: e);
        }
      };

      const rejectionFn = (reason) => {
        try {
          const result = onReject(reason); 
          if (result instanceof MyPromise) {
            result.then(resolveFn, rejectFn);
          } else {
            rejectFn(result);
          }
        } catch(e) {
          rejectFn(e instanceof Error ? e.message: e);
        }
      };

      if (this.status === PENDDING) {
        this.resolveQueue.push(fuifillmentFn);
        this.rejectQueue.push(rejectionFn);
      }
      if (this.status === FULFILLMENT) {
        fuifillmentFn(this.value)
      }
      if (this.status === REJECTION) {
        rejectionFn(this.reason);
      }
    });
  }
}
```
Promise的设计模式就是```观察者模式```，then函数订阅，实例状态改变直接通知相应的调用。这里只是实现了必要的功能，还有部分未实现的，将会在下篇中继续~