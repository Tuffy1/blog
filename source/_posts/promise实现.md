---
title: promise实现
date: 2019-08-07 23:20:07
tags:
---
## 一 promise 的常见使用例子

```js
// promise 使用实例
let p = new Promise(function (resolve, reject) {
  setTimeout(function () {
    let randomNumber = Math.random();
    if (randomNumber > 0.7) {
      resolve(randomNumber);
    } else {
      reject(randomNumber);
    }
  }, 0)
});
p
.then(function (data) {
  return Promise.resolve(data.toFixed(3));
}, function (data) {
  let d = data.toFixed(3);
  if (data < 0.5) {
    throw d;
  }
  return d;
})
.then((data) => {
  console.log(`success: ${data}`);
}, (data) => {
  console.log(`bad luck: ${data}`)
})
```

## 二 实现

promise 对象的构造函数参数为一个函数executor: function(resolve, reject){}

```js
class Promise {
  constructor(executor) {
    try {
      executor(resolve, reject);
    } catch {
      throw new Error('执行失败');
    }
  }
}
```

executor函数可以传入参数resolve和reject，对应的分别是成功和失败时执行的函数

```js
class Promise {
  constructor(executor) {
    this.resolve = (value) => {
      // 成功时的处理
    }
    this.reject = (reason) => {
      // 失败时的处理
    }
    try {
      executor(resolve, reject);
    } catch {
      throw new Error('执行失败');
    }
  }
}
```

promise存在三个状态：pending，resolved(fulfilled)，rejected。并且当pending往resolved或者rejected任何一个状态转变后，则其状态将不会再变化，即状态不可逆。所以promise需要一个变量来记录其当前状态，同时需要参数value和reason分别记录成功失败时的数据，用于之后执行then中的函数的参数

```js
const PENDING = 'pending';
const RESOLVED = 'resolved';
const REJECTED = 'rejected';
class Promise {
  constructor(executor) {
    this.state = PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.resolve = (value) => {
      // 成功时的处理
      this.state = RESOLVED;
    }
    this.reject = (reason) => {
      // 失败时的处理
      this.state = REJECTED;
    }
    try {
      executor(resolve, reject);
    } catch {
      throw new Error('执行失败');
    }
  }
}
```

promise有then函数，并且该函数接收两个函数参数onFulfilled, onRejected，当promise状态为resolved时执行onFulfilled,当promise状态为rejected时执行onRejected

```js
const PENDING = 'pending';
const RESOLVED = 'resolved';
const REJECTED = 'rejected';
class Promise {
  constructor(executor) {
    this.state = PENDING;
    this.value = undefined;
    this.reason = undefined;
    this.resolve = (value) => {
      // 成功时的处理
      this.state = RESOLVED;
      this.value = value;
    }
    this.reject = (reason) => {
      // 失败时的处理
      this.state = REJECTED;
      this.reason = reason;
    }
    try {
      executor(resolve, reject);
    } catch {
      throw new Error('执行失败');
    }
  }
  then(onFulfilled, onRejected) {
    if (this.state === RESOLVED) {
      onFulfilled(this.value);
    }
    if (this.state === REJECTED) {
      onRejected(this.reason);
    }
  }
}
```

当执行then方法时，可能此时的promise state处于pending状态，不能立即执行onFulfilled或者onRejected函数，需要等到state转为完成态时再相应执行。故定义两个数组onFulfilledCallbacks[]、onRejectedCallbacks[]，分别存放then的两个函数参数，等待完成态时执行。之所以是数组，是因为同一个promise可以执行多次then函数

```js
const PENDING = 'pending';
const RESOLVED = 'resolved';
const REJECTED = 'rejected';
class myPromise {
  constructor(executor) {
    this.state = PENDING,
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];
    this.resolve = (value) => {
      // 成功时的处理
      this.state = RESOLVED;
      this.value = value;
      this.onFulfilledCallbacks.forEach(cb => cb());
    };
    this.reject = (reason) => {
      // 失败时的处理
      this.state = REJECTED;
      this.reason = reason;
      this.onRejectedCallbacks.forEach(cb => cb());
    };
    try {
      executor(this.resolve, this.reject);
    } catch(e) {
      throw new Error(e);
    }
  }
  then(onFulfilled, onRejected) {
    if (this.state === RESOLVED) {
      onFulfilled(this.value);
    }
    if (this.state === REJECTED) {
      onRejected(this.reason);
    }
    if (this.state === PENDING) {
      this.onFulfilledCallbacks.push(() => {
        onFulfilled(this.value);
      });
      this.onRejectedCallbacks.push(() => {
        onRejected(this.reason);
      });
    }
  }
}
```

因为then方法可以链式调用，如p.then().then()，这意味着then方法本身返回一个Promise对象，并且是一个新的Promise对象，因为原Promsie对象的状态已经可能是完成态，并且这个状态是不能逆转的。

```js
const PENDING = 'pending';
const RESOLVED = 'resolved';
const REJECTED = 'rejected';
class myPromise {
  constructor(executor) {
    this.state = PENDING,
    this.value = undefined;
    this.reason = undefined;
    this.onFulfilledCallbacks = [];
    this.onRejectedCallbacks = [];
    this.resolve = (value) => {
      // 成功时的处理
      this.state = RESOLVED;
      this.value = value;
      this.onFulfilledCallbacks.forEach(cb => cb());
    };
    this.reject = (reason) => {
      // 失败时的处理
      this.state = REJECTED;
      this.reason = reason;
      this.onRejectedCallbacks.forEach(cb => cb());
    };
    try {
      executor(this.resolve, this.reject);
    } catch(e) {
      throw new Error(e);
    }
  }
  then(onFulfilled, onRejected) {
    let promise2;
    let x;
    if (this.state === RESOLVED) {
      promise2 = new myPromise((resolve, rejecte) => {
        x = onFulfilled(this.value);
        resolve(x);
      });
    }
    if (this.state === REJECTED) {
      promise2 = new myPromise((resolve, reject) => {
        x = onRejected(this.reason);
        reject(x);
      });
    }
    if (this.state === PENDING) {
      promise2 = new myPromise((resolve, rejecte) => {
        this.onFulfilledCallbacks.push(() => {
          x = onFulfilled(this.value);
          resolve(x);
        });
        this.onRejectedCallbacks.push(() => {
          x = onRejected(this.reason);
          reject(x);
        });
      });
    }
    return promise2;
  }
}
```

上面只是处理了then的参数是函数的情况，但then可以不传参，这种情况下链式调用依然可以正常进行，此外，then的参数函数也可能返回的是一个Promise，此时应该将其拆解，将其内容放在我们所定义的promise2中,所以写函数resolvePromise来处理then中参数的各种情况

```js
 then(onFulfilled, onRejected) {
    let promise2;
    let x;
    // 处理then参数不为函数的情况，比如直接没有传参
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : y => y;
    onRejected = typeof onRejected === 'function' ? onRejected : y => y;
    if (this.status === RESOLVED) {
      promise2 = new myPromise((resolve, reject) => {
        setTimeout(() => {
          x = onFulfilled(this.value);
          this._resolvePromise(promise2, x, resolve, reject);
        }, 0);
      });
    }
    if (this.status === REJECTED) {
      promise2 = new myPromise((resolve, reject) => {
        setTimeout(() => {
          x = onRejected(this.reason);
          this._resolvePromise(promise2, x, resolve, reject);
        }, 0);
      });
    }
    if (this.status === PENDING) {
      promise2 = new myPromise((resolve, reject) => {
        this.onFulfilledCallbacks.push(() => {
          setTimeout(() => {
            x = onFulfilled(this.value);
            this._resolvePromise(promise2, x, resolve, reject);
          }, 0);
        });
        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            x = onRejected(this.reason);
            this._resolvePromise(promise2, x, resolve, reject);
          }, 0);
        });
      });
    }
    return promise2;
  }
  _resolvePromise(promise2, x, resolve, reject) {
    let called = false;
    if (x !== null && (typeof x === 'object' || typeof x === 'function')) {
      try {
        let then = x.then;
        if (typeof then === 'function') {
          try {
            then.call(x, v => {
              if (called) return;
              called = true;
              this._resolvePromise(promise2, v, resolve, reject);
            }, r => {
              if (called) return;
              called = true;
              reject(r);
            })
          } catch(e) {
            reject(e);
          }
        } else {
          resolve(x);
        }
      } catch(e) {
        reject(e);
      }
    } else {
      resolve(x);
    }
  }
```
