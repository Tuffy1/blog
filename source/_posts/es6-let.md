---
title: es6 let
date: 2020-01-04 11:21:41
tags:
---

## let 特征

我们知道了 let 声明一个块级作用域的本地变量。拥有下面特征：

1. 作用域只作用于块级；
2. 不会变量提升（故而存在“暂存死区”）；
3. 不允许重复声明；

## let 与 for 循环

可以通过两个例子了解 let 在 for 循环中的行为表现：

```js
for (let i = 0; i < 3; i += 1) {
  let i = "a";
  console.log(i);
}
```

结果所得是：

```
a
a
a
```

这是因为 for 循环中的括号内 `for (let i = 0; i < 3; i += 1)` 和大括号内的内容已经处于两个不同作用域，上面的例子用 babel 转换后代码如下：

```js
for (var i = 0; i < 3; i += 1) {
  var _i = "a";
  console.log(_i);
}
```

可以看到 `for (let i = 0; i < 3; i += 1)` 和大括号内的 i 事实上不是同一个变量。其实也可以看成是大括号内是一个闭包，并且参数 i 保留其上下文变量：

```js
var _loop = function(i) {
  var i = "a";
  console.log(i);
};
for (var i = 0; i < 3; i += 1) {
  _loop(i);
}
```

可以看另一个例子，这个体现得更为明显：

```js
let fn = [];
for (let i = 0; i < 3; i += 1) {
  fn[i] = function() {
    console.log(i);
  };
}
fn[1](); // 1
```

我们知道，如果使用 var 来定义括号内的 i，那么最终输出结果为 3，但是使用 let 来定义括号内的 i，输出结果为 1。我们使用 babel 转换上面代码：

```js
var fn = [];

var _loop = function _loop(i) {
  var i = "abc";

  fn[i] = function() {
    console.log(i);
  };
};

for (var i = 0; i < 3; i += 1) {
  _loop(i);
}

fn[1]();
```

此时我们可以清楚看到循环的大括号内的内容被放在了另一个函数作用域内。这处理方式与我们上面例子是一样的。

## 暂存死区

```js
let i = "a";
if (true) {
  console.log(i); // a
}
```

```js
let i = "a";
if (true) {
  console.log(i); // ReferenceError
  let i = "b";
}
```

上面第一个例子中打印 i 是正常的，但第二个例子会报错，原因是：
第一个例子中 i 对于它整个所处的作用域起作用，它进行了声明并赋值，所以后面读取 i 可以正常读取；第二个例子中在 if 块中有自己的对 i 的声明，此时这个块中读取的 i 不再是 if 外的 i，但是在 `console.log` 的地方，i 还未被初始化（let 没有变量提升，不会像 var 一样对变量初始化为 undefined），故 `console.log` 正处于 i 的暂存死区，此时读取 i 值被报错，报的错误也是 `Cannot access 'i' before initialization`。
