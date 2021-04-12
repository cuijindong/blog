---
title: 手写Promise
categories: 前端进阶
tags: 
- 手写
wordcount: true
---
# 为何写这篇
一是面试的时候始终被问，被问的烦了就自己写一个吧（开始没有太深究实现，不知道为啥，手写Promise一直有人问）。二是看过很多关于手写Promise的文章，也看过Promise/A+的文档，但感觉总是乱乱的，还是自己写一个吧，写了踏实（感觉没啥用啊）。
# Promise实现
看别的文章也是一步一步来实现的，我也一步一步来吧，要不然看起来太乱了。本文对应了Promise/A+地址如下：[中文](https://www.ituring.com.cn/article/66566) | [英文](https://promisesaplus.com/)
## 基本Promise
首先实现一个基本的Promise，不用考虑太多。

1. 一个 Promise 的当前状态必须为以下三种状态中的一种：等待态（Pending）、执行态（Fulfilled）和拒绝态（Rejected）
2. new Promise时接收一个方法，这里叫这个方法为executor。该方法接受两个参数resolve和reject。
3. Promise有一个then方法。该方法注册了两个回调函数，用于接收 promise 的终值或本 promise 不能执行的原因。
4. 如果状态是Pending时代表executor还没有执行完，需要把then的回调onFulfilled和onRejected放入onResolvedCallbacks和onRejectedCallbacks数组缓存起来等待执行。
```javascript
const PENDING = 'PENDING';
const FULFILLED = 'FULFILLED';
const REJECTED = 'REJECTED';
class Promise {
  constructor(executor) {
    if (typeof executor !== 'function') {
      throw TypeError(`Promise resolver ${executor} is not a function`)
    }
    // 状态
    this.status = PENDING;
    // 成功值
    this.value = undefined;
    // 失败原因
    this.reason = undefined;
    // 存放成功的回调
    this.onResolvedCallbacks = [];
    // 存放失败的回调
    this.onRejectedCallbacks= [];

    const resolve = (value) => {
      if (this.status === PENDING) {
        this.status = FULFILLED
        this.value = value
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }

    const reject = (reason) => {
      if (this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
        this.onRejectedCallbacks.forEach(fn => fn())
      }
    }
    try{
      executor(resolve, reject)
    } catch (error){
      reject(error)
    }
  }
  then(onFulfilled, onRejected) {
    if (this.status === PENDING) {
      // 如果promise的状态是 pending，需要将 onFulfilled 和 onRejected 函数存放起来，等待状态确定后，再依次将对应的函数执行
      this.onResolvedCallbacks.push(() => {
        onFulfilled(this.value)
      })
      this.onRejectedCallbacks.push(() => {
        onRejected(this.reason)
      })
    }
    if (this.status === FULFILLED) {
      onFulfilled(this.value)
    }
    if (this.status === REJECTED) {
      onRejected(this.reason)
    }
  }
}

module.exports = Promise
```
测试一下代码
```javascript
const Promise = require('./promiseMe')

new Promise(function (resovle, reject) {
  console.log('Promise start')
  setTimeout(() => {
    resovle('执行完毕')
  }, 1000)
}).then(res => {
  console.log(res)
})

// Promise start
1s后
// 执行完毕
```
## 完整Promise
小孩都知道Promise.then支持链式调用、穿透调用等。下面根据Promise/A+不断完善。
### 2.2.1 onFulfilled和onRejected都是可选的参数
* 2.2.1.1. 如果onFulfilled不是一个函数，它必须被忽略
* 2.2.1.2. 如果onRejected不是一个函数，它必须被忽略

也就是then的穿透性，例如Promise.then().then()，如果第一个then没有操作，Promise的value或resone会被传递到下个then()。换句话说就是假如第一个then的onFulfilled或onRejected不是个函数就将onFulfilled或onRejected赋值成一个函数。
```javascript
then(onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
  // 抛出 e 防治被resolve捕获。
  onRejected = typeof onRejected === 'function' ? onRejected : e => {throw e}
  if (this.status === PENDING) {
    // 如果promise的状态是 pending，需要将 onFulfilled 和 onRejected 函数存放起来，等待状态确定后，再依次将对应的函数执行
    this.onResolvedCallbacks.push(() => {
      onFulfilled(this.value)
    })
    this.onRejectedCallbacks.push(() => {
      onRejected(this.reason)
    })
  }
  if (this.status === FULFILLED) {
    onFulfilled(this.value)
  }
  if (this.status === REJECTED) {
    onRejected(this.reason)
  }
}
```
### 2.2.4. 在执行上下文栈中只包含平台代码之前，onFulfilled或onRejected一定不能被调用 [3.1]
* 3.1. 这里“平台代码”意味着引擎、环境以及promise的实现代码。在实践中，这需要确保onFulfilled和onRejected异步地执行，并且应该在then方法被调用的那一轮事件循环之后用新的执行栈执行。这可以用如setTimeout或setImmediate这样的“宏任务”机制实现，或者用如MutationObserver或process.nextTick这样的“微任务”机制实现。由于promise的实现被考虑为“平台代码”，因此在自身处理程序被调用时可能已经包含一个任务调度队列。

将onFulfilled和onRejected包在setTimeout中。
```javascript
then(onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
  onRejected = typeof onRejected === 'function' ? onRejected : e => {throw e}
  if (this.status === PENDING) {
    // 如果promise的状态是 pending，需要将 onFulfilled 和 onRejected 函数存放起来，等待状态确定后，再依次将对应的函数执行
    this.onResolvedCallbacks.push(() => {
      setTimeout(() => {
        onFulfilled(this.value)
      }, 0)
    })
    this.onRejectedCallbacks.push(() => {
      setTimeout(() => {
        onRejected(this.reason)
      }, 0)
    })
  }
  if (this.status === FULFILLED) {
    setTimeout(() => {
      onFulfilled(this.value)
    }, 0)
  }
  if (this.status === REJECTED) {
    setTimeout(() => {
      onRejected(this.reason)
    }, 0)
  }
}
```
### 2.2.7. then必须返回一个promise [3.3]
* 2.2.7.1. 如果onFulfilled或onRjected返回一个值x，运行promise解决程序[[Resolve]](promise2,x)
* 2.2.7.2. 如果onFulfilled或onRejected抛出一个异常e，promise2必须用e作为原因被拒绝
* 2.2.7.3. 如果onFulfilled不是一个函数并且promise1被解决，promise2必须用与promise1相同的值被解决
* 2.2.7.4. 如果onRejected不是一个函数并且promise1被拒绝，promise2必须用与promise1相同的原因被拒绝

关于 **2.2.7.1** 中的promise解决程序后续会完整的介绍，这里先设定 **resolvePromise()** 就是这个解决程序。

关于 **2.2.7.2** 实现思路为将 onFulfilled或onRejected 放在 **try...catch** 中，如果捕获到异常e，用promise2的reject拒绝。
```javascript
then(onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
  onRejected = typeof onRejected === 'function' ? onRejected : e => {throw e}
  let promise2 = new Promise((resovle, reject) => {
    if (this.status === PENDING) {
      // 如果promise的状态是 pending，需要将 onFulfilled 和 onRejected 函数存放起来，等待状态确定后，再依次将对应的函数执行
      this.onResolvedCallbacks.push(() => {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value)
            resolvePromise(promise2, x, resovle, reject)
          } catch (error) {
            reject(error)
          }
        }, 0)
      })
      this.onRejectedCallbacks.push(() => {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason)
            resolvePromise(promise2, x, resovle, reject)
          } catch (error) {
            reject(error)
          }
        }, 0)
      })
    }
    if (this.status === FULFILLED) {
      setTimeout(() => {
        try {
          let x = onFulfilled(this.value)
          resolvePromise(promise2, x, resovle, reject)
        } catch (error) {
          reject(error)
        }
      }, 0)
    }
    if (this.status === REJECTED) {
      setTimeout(() => {
        try {
          let x = onRejected(this.reason)
          resolvePromise(promise2, x, resovle, reject)
        } catch (error) {
          reject(error)
        }
      }, 0)
    }
  })
  return promise2
}
```
## Promise解决程序
关于promise解决程序应该是promise中最核心最绕的地方了。可以说前面的Promise就是利用了 **发布订阅** 模式完成依赖收集和触发执行实现Promise对于异步的处理。但是Promise的 **链式调用** 以及 **嵌套** 的实现都是基于promise解决程序。Promise/A+定义如下:

> promise解决程序是一个抽象操作，它以一个promise和一个值作为输入，我们将其表示为[[Resolve]](promise, x)。如果x是一个thenable，它尝试让promise采用x的状态，并假设x的行为至少在某种程度上类似于promise。否则，它将会用值x解决 promise。
这种thenable的特性使得Promise的实现更具有通用性：只要其暴露一个遵循Promise/A+协议的then方法即可。这同时也使遵循Promise/A+规范的实现可以与那些不太规范但可用的实现能良好共存。

我的理解是 **promise解决程序是为了解决x如果是一个Promise，那么promise2应该resolve/reject什么的一个定义。主要思路是如果x是一个Promise，那么执行x的then的方式确定promise2的状态，类似于将x.then往promise2.then上的移植**

* 2.3.1. 如果promise和x引用同一个对象，用一个TypeError作为原因来拒绝promise
* 2.3.2. 如果x是一个promise，采用它的状态：[3.4]
  - 2.3.2.1. 如果x是等待态，promise必须保持等待状态，直到x被解决或拒绝
2.3.2.2. 如果x是解决态，用相同的值解决promise
2.3.2.3. 如果x是拒绝态，用相同的原因拒绝promise

* 2.3.3. 否则，如果x是一个对象或函数
  - 2.3.3.1. 让then成为x.then。[3.5]
  - 2.3.3.2. 如果检索属性x.then导致抛出了一个异常e，用e作为原因拒绝promise
  - 2.3.3.3. 如果then是一个函数，用x作为this调用它。then方法的参数为俩个回调函数，第一个参数叫做resolvePromise，第二个参数叫做rejectPromise：
    - 2.3.3.3.1. 如果resolvePromise用一个值y调用，运行[[Resolve]](promise, y)。译者注：这里再次调用[[Resolve]](promise,y)，因为y可能还是promise
    - 2.3.3.3.2. 如果rejectPromise用一个原因r调用，用r拒绝promise。译者注：这里如果r为promise的话，依旧会直接reject，拒绝的原因就是promise。并不会等到promise被解决或拒绝
    - 2.3.3.3.3. 如果resolvePromise和rejectPromise都被调用，或者对同一个参数进行多次调用，那么第一次调用优先，以后的调用都会被忽略。译者注：这里主要针对thenable，promise的状态一旦更改就不会再改变。
    - 2.3.3.3.4. 如果调用then抛出了一个异常e,
  - 2.3.3.4. 如果then不是一个函数，用x解决promise
* 2.3.4. 如果x不是一个对象或函数，用x解决promise
```javascript
const resolvePromise = (promise2, x, resolve, reject) => {
  // Promise/A+ 2.3.1
  if (promise2 === x) {
    return reject(new TypeError('Chaining cycle detected for promise #<Promise>'))
  }
  // Promise/A+ 2.3.3.3.3 只能调用一次
  let called;
  // Promise/A+ 2.3.2
  // Promise/A+ 2.3.3
  if ((typeof x === 'object' && x != null) || typeof x === 'function'){
    try {
      // Promise/A+ 2.3.3.1
      let then = x.then
      // Promise/A+ 2.3.3.3
      if (typeof then === 'function') {
        then.call(x, y => { // Promise/A+ 2.3.3.3.1 -- 递归调用，y可能也是Promise
          if (called) return;
          called = true;
          resolvePromise(promise2, y, resolve, reject); 
        }, r => { // Promise/A+ 2.3.3.3.2
          if (called) return;
          called = true;
          reject(r)
        })
      } else {
        // Promise/A+ 2.3.3.4
        resolve(x)
      }
    } catch (error) {
      // Promise/A+ 2.3.3.2 在try中可能改变了status
      if (called) return;
      called = true;
      reject(error)
    }
  } else {
    // Promise/A+2.3.4
    resolve(x)
  }
}
```
## 完整代码
```javascript
const PENDING = 'PENDING';
const FULFILLED = 'FULFILLED';
const REJECTED = 'REJECTED';
const resolvePromise = (promise2, x, resolve, reject) => {
  // Promise/A+ 2.3.1
  if (promise2 === x) {
    return reject(new TypeError('Chaining cycle detected for promise #<Promise>'))
  }
  // Promise/A+ 2.3.3.3.3 只能调用一次
  let called;
  // Promise/A+ 2.3.2
  // Promise/A+ 2.3.3
  if ((typeof x === 'object' && x != null) || typeof x === 'function'){
    try {
      // Promise/A+ 2.3.3.1
      let then = x.then
      // Promise/A+ 2.3.3.3
      if (typeof then === 'function') {
        then.call(x, y => { // Promise/A+ 2.3.3.3.1 -- 递归调用，y可能也是Promise
          if (called) return;
          called = true;
          resolvePromise(promise2, y, resolve, reject); 
        }, r => { // Promise/A+ 2.3.3.3.2
          if (called) return;
          called = true;
          reject(r)
        })
      } else {
        // Promise/A+ 2.3.3.4
        resolve(x)
      }
    } catch (error) {
      // Promise/A+ 2.3.3.2 在try中可能改变了status
      if (called) return;
      called = true;
      reject(error)
    }
  } else {
    // Promise/A+2.3.4
    resolve(x)
  }
}
class Promise {
  constructor(executor) {
    if (typeof executor !== 'function') {
      throw TypeError(`Promise resolver ${executor} is not a function`)
    }
    // 状态
    this.status = PENDING;
    // 成功值
    this.value = undefined;
    // 失败原因
    this.reason = undefined;
    // 存放成功的回调
    this.onResolvedCallbacks = [];
    // 存放失败的回调
    this.onRejectedCallbacks= [];

    const resolve = (value) => {
      if(value instanceof Promise){
        return value.then(resolve,reject)
      }
      if (this.status === PENDING) {
        this.status = FULFILLED
        this.value = value
        this.onResolvedCallbacks.forEach(fn => fn())
      }
    }

    const reject = (reason) => {
      if (this.status === PENDING) {
        this.status = REJECTED
        this.reason = reason
        this.onRejectedCallbacks.forEach(fn => fn())
      }
    }
    try{
      executor(resolve, reject)
    } catch (error){
      reject(error)
    }
  }
  then(onFulfilled, onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : v => v
    onRejected = typeof onRejected === 'function' ? onRejected : e => {throw e}
    let promise2 = new Promise((resovle, reject) => {
      if (this.status === PENDING) {
        // 如果promise的状态是 pending，需要将 onFulfilled 和 onRejected 函数存放起来，等待状态确定后，再依次将对应的函数执行
        this.onResolvedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value)
              resolvePromise(promise2, x, resovle, reject)
            } catch (error) {
              reject(error)
            }
          }, 0)
        })
        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason)
              resolvePromise(promise2, x, resovle, reject)
            } catch (error) {
              reject(error)
            }
          }, 0)
        })
      }
      if (this.status === FULFILLED) {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value)
            resolvePromise(promise2, x, resovle, reject)
          } catch (error) {
            reject(error)
          }
        }, 0)
      }
      if (this.status === REJECTED) {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason)
            resolvePromise(promise2, x, resovle, reject)
          } catch (error) {
            reject(error)
          }
        }, 0)
      }
    })
    return promise2
  }
}

module.exports = Promise

```
## 源码地址
[源码](https://github.com/cuijindong/example/blob/master/js/Promise/promise.js)
## 代码出处
[面试官：“你能手写一个 Promise 吗”](https://zhuanlan.zhihu.com/p/183801144)