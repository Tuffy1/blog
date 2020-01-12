---
title: es6 Generator
date: 2020-01-12 15:34:30
tags:
---

## Generator 生成器

Generator 函数包含一组状态，调用它时返回一个 Iterator 迭代器对象，通过 Iterator 执行并一步一步读取 Generator 中的状态。

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
let ite = gen();
ite.next(); // 开始执行 gen 函数体内的语句，返回第一个状态 { value: 1, done: false }
ite.next(); // 返回第二个状态 { value: 2, done: false }
ite.next(); // 返回第三个状态 { value: 3, done: false }
ite.next(); // 返回第四个状态 { value: undefined, done: true }
```

`ite = gen()` 语句时并没有开始执行 gen 函数中的内容，而是直接返回了一个 Iterator。在第一个 `ite.next()` 时，才开始真正执行，并在遇到第一次 yield 时停止，以 `{ value: 1, done: false }` 的形式返回第一个 yield 表达式产生的值。value 是对应的值，done 表示是否结束，当函数执行完毕，或者遇到 return 时，done 为 true。

## Symbol.iterator 与 for...of

一些数据格式自带 Symbol.iterator 方法，比如 Array，Map，Set, NodeList 等，执行该方法可以得到一个 Iterator 对象，以数组为例：

```js
let arr = [11, 2, 33, 4];
let ite = arr[Symbol.iterator]();
ite.next(); // { value: 11, done: false }
ite.next(); // { value: 2, done: false }
ite.next(); // { value: 33, done: false }
ite.next(); // { value: 4, done: false }
ite.next(); // { value: undefined, done: true }
```

可以看到，通过调用数组的 Symbol.iterator 方法，得到的 ite，`ite.next()` 方法迭代读取的是数组的每个元素，这也是 for...of 的实现原理，通过对象的 Symbol.iterator 方法来获取每一步值。
假设我们重新实现以下 arr 的 Symbol.iterator 方法，再执行以下 for...of

```js
let arr = [11, 2, 33, 4];
arr[Symbol.iterator] = function*() {
  yield "arr yield 1";
  yield "arr yield 2";
  yield "arr yield 3";
};
for (let i of arr) {
  console.log(i);
}
```

输出结果：

```
arr yield 1
arr yield 2
arr yield 3
```

与 for...of 同样的，解构赋值，扩展与 Array.from 方法也是调用的 Iterator 接口：

```js
function* gen() {
  yield "a";
  yield "b";
  yield "c";
}
let ite = gen();
[...ite]; // ['a', 'b', 'c']
ite = gen(); // 注意这里不能直接使用ite，因为此时ite已经走完了处于close。要重新执行
let [x, y, z] = ite; // x: 'a', y: 'b', z: 'c'
```

## next 方法参数

当调用 next 方法时，可以往其中传参数，这个参数作为上一次状态的返回值。举个例子：

```js
let res1, res2, res3;
function* gen() {
  x = yield "a";
  y = yield `${x}b`;
  yield z;
}
let ite = gen();
res1 = ite.next(); // 第一步不传参，第一步传参的话无意义，因为这已经是第一个状态，不存在它的上一个状态
res2 = ite.next(res1.value); // 此时将 res1.value 为'a'，并将它传给了 x，得到 res2 为 'ab'
res3 = ite.next(res2.value); // 此时将 res2.value 为'ab'，并将它传给了 y，得到 res3 为'abc'
```

## Generator 与异步

### 自动执行 Generator：then 方法

```js
let fetch = require("node-fetch");
function* gen() {
  let data = yield fetch("https://api.github.com/users/github");
  console.log(data);
  let json = yield data.json();
  console.log(json);
}
function run(gen) {
  let ite = gen();
  function next(data) {
    let res = ite.next(data);
    if (res.done) return res.value;
    res.value.then(d => {
      next(d);
    });
  }
  next();
}
run(gen);
```

但这要求 res.value 需要是 Promise 对象，所以可以尝试对 res.value 进行 promisify，以帮助我们在 then 中调用 next 取回 gen 的执行权。此外我们还可以让 `run(gen)` 的返回值为 Promise 对象，这样可以进一步进行我们想要的异步操作，所以我们可以 new Promise 在其外面包装一层。同时添加一些错误处理：

```js
let fetch = require("node-fetch");
function* gen() {
  let data = yield fetch("https://api.github.com/users/github");
  let json = yield data.json();
  console.log(json);
  let email = yield json.email;
  console.log(email);
}
function run(gen) {
  function onFulfilled(v) {
    try {
      next(v);
    } catch (e) {
      return reject(e);
    }
  }
  function onRejected(err) {
    try {
      gen.throw(err);
    } catch (e) {
      return reject(e);
    }
  }
  return new Promise((resolve, reject) => {
    if (!gen || typeof gen !== "function") return resolve(gen);
    let ite = gen();
    if (!ite || typeof ite.next !== "function") return resolve(gen);
    next();
    function next(data) {
      let res = ite.next(data);
      if (res.done) return resolve(res.value);
      let value = toPromise(res.value);
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
      return onRejected(
        new TypeError(
          "You may only yield a function, promise, generator, array, or object, " +
            'but the following object was passed: "' +
            String(ret.value) +
            '"'
        )
      );
    }
  });
}
run(gen).then(() => {});
```

事实上，上面的核心代码来源一个模块 [co](https://github.com/tj/co)，一个帮助 Generator 函数自动执行的库。通过递归地在 `then` 中执行 `ite.next` 来达到当前异步操作执行完毕后就取回执行权的目的。

### 自动执行 Generator：thunk

关于 thunk 是什么，文章 [阮一峰 Thunk 函数的含义和用法](http://www.ruanyifeng.com/blog/2015/05/thunk.html) 对此有清晰的解释。总的来说，讲一个函数转为 thunk 函数有以下的效果：

```js
let fs = require("fs");
let thunkify = require("thunkify");
function cb(err, res) {
  if (err) {
    throw new Error(err);
  } else {
    console.log(res);
  }
}
let readFile = fs.readFile;
readFile("./package.json", cb);
let thunkReadFile = thunkify(readFile)("./package.json"); // thunkify
thunkReadFile(cb);
```

对比 `readFile("./package.json", cb)` 和 `thunkReadFile("./package.json")(cb)`，第一个函数调用是多参数的，而第二个可以**将回调函数作为一个单独的参数**。Thunk 转换的主要思路：

```js
let thunkify = fn => {
  return function() {
    let args = [...arguments];
    return function(callback) {
      args.push(callback);
      return fn.apply(this, args);
    };
  };
};
```

利用这一点，我们将 `ite.next` 这一步放在回调函数中，作为参数，使执行权回到 gen 函数：

```js
let fs = require("fs");
let thunkify = require("thunkify");
let readFile = fs.readFile;
let thunkReadFile = thunkify(readFile);
function* gen() {
  let pk = yield thunkReadFile("./package.json");
  console.log(pk);
  let pkl = yield thunkReadFile("./package-lock.json");
  console.log(pkl);
}
let ite = gen();
let res = ite.next();
res.value((err, data) => {
  // thunkReadFile("./package.json") 执行完毕后，将下一步操作作为回调
  res = ite.next(data);
  res.value((err, data) => {
    if (err) throw err;
    res = ite.next(data);
  });
});
```

通过将异步操作 thunkify，执行之后将下一步操作放在其 value（实际为一个 function）的参数执行。可以通过递归来使上面的调用更加普遍适用：

```js
let fs = require("fs");
let thunkify = require("thunkify");
let readFile = fs.readFile;
let thunkReadFile = thunkify(readFile);
function* gen() {
  let pk = yield thunkReadFile("./package.json");
  console.log(pk);
  let pkl = yield thunkReadFile("./package-lock.json");
  console.log(pkl);
}
function next(data) {
  let res = ite.next(data);
  if (res.done) return; // 判断结束
  res.value((err, data) => {
    if (err) throw err;
    next(data); // 递归调用
  });
}
let ite = gen();
next();
```

## 参考

[阮一峰 ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/generator)
