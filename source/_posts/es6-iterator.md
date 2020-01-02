---
title: es6 Iterator
date: 2020-01-02 21:50:23
tags:
---

## Iterator 迭代器

### 介绍

Iterator 是一个具有 next 方法的对象。通过调用 next 可以一步一步得到序列的值。

```js
// 迭代器对象可以通过重复调用next()显示迭代。
let map = new Map([
  ["a", 1],
  ["b", 2]
]);
let iter = map["Symbol.iterator"](); // Iterator迭代器
iter.next(); // {value: ["a", 1], done: false}.
iter.next(); // {value: ["b", 2], done: false}.
iter.next(); // {value: undefined, done: true}.
```

next() 方法的调用返回一个对象，对象中有两个属性 value 和 done。value 是序列中 next 的值，done 表示是否走到序列结束。

一些数据格式自带 Iterator 迭代器，比如 Array，Map，Set, NodeList 等。

!['prototype中的Symbol.iterator'](./images/iterator1.jpg)

注意：object 没有遍历器，因为本身键值没有顺序。若要对 object 遍历，实际上可以直接使用 Map。

### Iterator 简单实现

简单实现一个可以体现 Iterator 的函数，使之可以这样调用：

```js
const arr = ["a", "b", "c", "d"];
const iter = makeIterator(arr);
iter.next(); // {value: 'a', done: false}
iter.next(); // {value: 'b', done: false}
iter.next(); // {value: 'c', done: false}
iter.next(); // {value: 'd', done: false}
iter.next(); // {value: undefined, done: true}
```

那么 makeIterator 方法要返回一个对象，该对象中要有 next 方法，而 next 方法执行要返回一个对象，对象中有属性 value 和 done。

```js
function makeIterator(array) {
  let step = -1;
  const len = array.length;
  return {
    next: () => {
      step++;
      return {
        value: array[step],
        done: step >= len
      };
    }
  };
}
const arr = ["a", "b", "c", "d"];
const iter = makeIterator(arr);
iter.next(); // {value: 'a', done: false}
iter.next(); // {value: 'b', done: false}
iter.next(); // {value: 'c', done: false}
iter.next(); // {value: 'd', done: false}
iter.next(); // {value: undefined, done: true}
```

也可以利用生成器实现上面函数效果：

```js
function* makeIterator(array) {
  let step = -1;
  const len = array.length;
  while (step < len) {
    step++;
    yield array[step];
  }
}
const arr = ["a", "b", "c", "d"];
const iter = makeIterator(arr);
iter.next(); // {value: 'a', done: false}
iter.next(); // {value: 'b', done: false}
iter.next(); // {value: 'c', done: false}
iter.next(); // {value: 'd', done: false}
iter.next(); // {value: undefined, done: true}
```

上面例子通过函数简单实现数组的可迭代，实际上一个对象的迭代行为（比如 for...of）的过程中会调用该对象原型中 Symbol.iterator 接口。我们可以通过重写 Symbol.iterator 接口自定义可迭代对象：

```js
const myIterable = {
  *[Symbol.iterator]() {
    yield 1;
    yield 2;
    yield 3;
  }
};
for (let v of myIterable) {
  console.log(v); // 1 2 3
}
```

## for...of

不同内置可迭代对象 for...of 的表现：

```js
let map = new Map([
  ["a", 1],
  ["b", 2]
]); // Map(2) {"a" => 1, "b" => 2}
for (let pair of map) {
  console.log(pair); // (2)["a", 1], (2)["b", 2]
}

let set = new Set([1, 2, 3, 4]);
for (let i of set) {
  console.log(i); // 1, 2, 3, 4
}
for (let pair of set.entries()) {
  console.log(pair); // (2)[1, 1] (2)[2, 2] (2)[3, 3] (2)[4, 4]
}

let arr = new Array(1, 2, 3, 4);
for (let i of arr) {
  console.log(i); // 1, 2, 3, 4
}
for (let pair of arr.entries()) {
  console.log(pair); // (2)[0, 1] (2)[1, 2] (2)[2, 3] (2)[3, 4]
}
```

上面例子可以看到：

1. map 的 for...of 直接得到键值对；
2. set 的 for...of 得到元素值，而对 set.entries() 得到的是键值相同的键值对；
3. array 的 for...of 得到元素值，而对 arr.entries() 得到的是索引与值的键值对。

### for...in 与 for...of

```js
let arr = [1, 2, 3, 4];
for (let i in arr) {
  // 如果循环的是数组
  console.log(typeof i); // (4) string。数组索引为number，但这里返回string
}
for (let [i] of arr.entries()) {
  // 如果循环的是数组
  console.log(typeof i); // (4) number。正常返回number
}
```

从上面例子可以看到：

1. for...in 得到的 i 总是 string，即使数组的索引应该为 number；
2. for...of 得到的 i 则与索引是同类型的；
3. 那么 for...in 更适用 object，因为 object 的 key 值总是 string 类型。而 array 索引都为 number ； set, map 的键可以为各种类型.
