---
title: 简要剖析Promise实现（下）
date: 2020-09-07 15:00:11
tags:
- javascript
categories:
- 前端
top_img: https://i.loli.net/2020/09/06/IcxbDBU6Tv5Hjyw.png
cover: https://i.loli.net/2020/09/06/IcxbDBU6Tv5Hjyw.png
---

# 前言
有了上一篇对Promise的简易实现，我们大概了解了如何手写一个MyPromise，这里接着上一篇的内容，继续完善MyPromise剩余的功能函数，并且从中体会和加深对Promise的理解，在实际应用中更能知根知底~

## MyPromise.resolve
按照对Promise的理解，这里只要直接返回一个『同步』的MyPromise实例就行了~
```javascript
class MyPromise {
  static resolve(val) {
    return new MyPromise((resolveFn, rejectFn) => {
      resolveFn(val);
    });
  }
  // ...
}
```

## MyPromise.reject
和resolve一样，换成reject就行了~
```javascript
class MyPromise {
  static reject(reason) {
    return new MyPromise((resolveFn, rejectFn) => {
      rejectFn(reason);
    });
  }
  // ...
}
```

## MyPromise.all
all的作用不用多说，我们直接来看实现吧~
```javascript
class MyPromise {
  static all(promises) {
    return new MyPromise((resolve, reject) => {
      try {
        const len = promises.length;
        const result = new Array(len);
        let count = 0;
        for (let i = 0; i < len; i += 1) {
          const p = promises[i];
          p.then(res => {
            result[i] = res;
            count += 1;
            if (count === len) {
              resolve(result);
            }
          });
        }
      } catch(e) {
        reject(e.message);
      }
    });
  }
  // ...
}
```

## MyPromise.race
好了，最后就剩下race啦，比起all，先到先得，更简单了~
```javascript
class MyPromise {
  static race(promises) {
    return new MyPromise((resolve, reject) => {
      try {
        for (let i = 0; i < promises.length; i += 1) {
          const p = promises[i];
          p.then(res => {
            resolve(res);
          });
        }
      } catch(e) {
        reject(e.message);
      }
    });
  }
  // ...
}
```

# MyPromise源码
```javascript
const PENDDING = "pedding";
const FULFILLMENT = "fulfillment";
const REJECTION = "rejection";

class MyPromise {
  static resolve(val) {
    return new MyPromise((resolveFn, rejectFn) => {
      resolveFn(val);
    });
  }

  static reject(reason) {
    return new MyPromise((resolveFn, rejectFn) => {
      rejectFn(reason);
    });
  }

  static all(promises) {
    return new MyPromise((resolve, reject) => {
      try {
        const len = promises.length;
        const result = new Array(len);
        let count = 0;
        for (let i = 0; i < len; i += 1) {
          const p = promises[i];
          p.then(res => {
            result[i] = res;
            count += 1;
            if (count === len) {
              resolve(result);
            }
          });
        }
      } catch(e) {
        reject(e.message);
      }
    });
  }

  static race(promises) {
    return new MyPromise((resolve, reject) => {
      try {
        for (let i = 0; i < promises.length; i += 1) {
          const p = promises[i];
          p.then(res => {
            resolve(res);
          });
        }
      } catch(e) {
        reject(e.message);
      }
    });
  }

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

  catch(reject) {
    this.then(null, reject);
  }
}

/* 测试case */
const p = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    console.log('resolve:hello world');
    resolve('hello world')
  }, 1000);
});

p.then(res => {
  console.log('then:', res);
  return new MyPromise((resolve, reject) => {
    setTimeout(() => {
      resolve(`${res}，hello js11`);
    }, 1000);
  })
  // return `${res}，hello js`;
})
.then()
.then(res => {
  console.log('经过透传：', res);
})
setTimeout(() => {
  MyPromise.resolve('静态resolve').then(res => console.log(res))
}, 1000)
console.time();
const list = [
  new MyPromise((resolve, reject) => {
    setTimeout(() => {
      console.log('test all: 1 done');
      resolve('one')
    }, 5000);
  }),
  new MyPromise((resolve, reject) => {
    setTimeout(() => {
      console.log('test all: 2 done');
      resolve('two')
    }, 3000);
  }),
  new MyPromise((resolve, reject) => {
    setTimeout(() => {
      console.log('test all: 3 done');
      resolve('three')
    }, 2000);
  }),
];
MyPromise.all(list).then(res => {
  console.log('MyPromise.all===>', res);
  console.timeEnd();
});
MyPromise.race(list).then(res => {
  console.log('MyPromise.race===>', res);
  console.timeEnd();
});
```

# 总结
总体来说，比起上一篇，这一篇并没有过多的解释，因为知道Promise的功能，简易实现的代码都非常通俗易懂，而且没有什么太绕的逻辑。所以通过简易实现MyPromise，做一次温故知新，继续加油~