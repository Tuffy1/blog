---
title: demothodizing
date: 2019-07-08 22:14:19
tags:
---

toStr = Function.call.bind(Object.prototype.toString)

## 一 含义

### 函数调用

1. Object.prototype.toStrig: 表示 Object 对象的 toString 方法；

2. bind：是 Function.prototype 的一个方法，该方法会有创建一个新函数，并将该函数的 this 指向其参数，并返回这个函数；

3. call：是 Function.prototype 的一个方法，与 bind 不同，call 函数会直接执行，并将函数内 this 指向参数；

### 执行效果

bind 创建一个新函数 Function.call 并将其 this 指向 Object.prototype.toString，故上面的语句相当于

```js
toStr = Object.prototype.toString.call(params);


```

## 二 作用

该语句的使用常是：

```js
// 得到一个将参数转为 toString 的方法
const toStr = Function.call.bind(Object.prototype.toString);

// 调用
toStr(something);


```

为什么不直接使用toString方法呢？

toString 是在 Object 原型中的方法，一般只有 Object 才可以调用，比如：

```js
var obj = {a: 1, b: 2};
obj.toString(); // 这是可以的

var arr = [1, 2, 3];
arr.toString(); // 这是不行的

```

但是经过 Function.call.bind(Object.prototype.toString) 所得是一个 Function，那么各种类型的数据都可以去做同样的执行。

同样的，forEach 方法也是一样的例子：

```js
// 得到一个将参数转为 toString 的方法
const each = Function.prototype.call.bind([].forEach);

// 调用
each([1, 2, 3], function (a) { console.log(a); }); // 参数为 arr

each('12345', function (s) { console.log(s); }); // 参数为 str。同样可以执行

```

这就使得一个 Array.prototype 上的方法，可以被创建一个新的，被 str 调用。

为什么当用 call 方法调用 forEach 传 String 类型的参数可以正常执行？

forEach 方法本身执行对 String 类型是 work 的，只是在 Array.prototype 中，本来只提供给 Array 类型调用，call 方法使得 String 类型可以调用 forEach 方法。如果传参是 Number 类型，那么将不能起到同样的作用。这是该语句的使用场景。
