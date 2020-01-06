---
title: "AMD, CMD, CommonJS 与 ES6Module"
date: 2020-01-06 19:28:24
tags:
---

## AMD - Require.js

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>AMD - Require.js</title>
  </head>
  <body>
    <!-- 引用 cdn require.js ，模块主入口为 index.js -->
    <script
      src="https://cdn.bootcss.com/require.js/2.3.6/require.js"
      data-main="./index.js"
    ></script>
  </body>
</html>
```

```js
// index.js
// 请求模块 ./add
require(["./add"], function(addModule) {
  console.log("执行 index.js");
  console.log(addModule.add(1, 2));
});
```

```js
// add.js
define(["require"], function(require, factory) {
  "use strict";
  console.log("执行 add.js");
  var add = function(a, b) {
    // 一个放在return对象中的函数，暴露出去被使用
    console.log("执行 add.js 中 函数 add");
    return a + b;
  };
  return {
    add: add
  };
});
```

上面代码运行之后，可以看到控制台的输出：

```
执行 add.js
执行 index.js
执行 add.js 中 函数 add
3
```

在打印 `"执行 index.js"` 前已经打印出 `"执行 add.js"`，**这说明在 require 时，就对模块进行了加载和执行**，之后将模块中 return 出来的内容放在变量 `addModule` 中。

## CMD - Sea.js

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>CMD - Sea.js</title>
  </head>
  <body>
    <!-- 引用 cdn sea.js ，模块主入口为 index.js -->
    <script src="https://cdn.bootcss.com/seajs/3.0.3/sea.js"></script>
    <script src="./index.js"></script>
  </body>
</html>
```

```js
// index.js

define(function(require, exports, module) {
  console.log("执行 index.js");
  var addModule = require("./add"); // 请求模块 ./add
  console.log(addModule.add(1, 2));
});
```

```js
// add.js
define(["require"], function(require, factory) {
  "use strict";
  console.log("执行 add.js");
  var add = function(a, b) {
    return a + b;
  };
  return {
    add: add
  };
});
```

上面代码运行之后，可以看到控制台的输出：

```
执行 index.js
执行 add.js
3
```

打断点在 `var addModule = require("./add");` 语句之前，可以看到参数 module 中已经保存了 add.js 中的相关内容，**可见与 require.js 相同，sea.js 也会对模块进行提前加载**。
在打印 `"执行 add.js"` 前已经打印出 `"执行 index.js"`，**这说明与 require.js 不同的时，sea.js 只有在 require 了模块时，才对模块进行了执行**。之后将模块中 return 出来的内容放在变量 `addModule` 中。

## AMD 与 CMD 的区别

1. AMD 依赖前置，需要引用的模块的模块都会写在最前面；CMD 不需要一开始什么自己需要的全部依赖是哪些，直接在代码中 require，但两者其实都会提前加载模块内容。
2. 虽然两者都会提前加载模块内容，但执行时间是不同的。AMD 会在模块加载完毕后执行，而 CMD 会在代码执行到相应语句时，才执行模块中内容。

## ES6 - Module

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>ES6 - Module</title>
  </head>
  <body>
    <!-- type="module"表示加载的是 ES6 模块 -->
    <script type="module" src="./index.js"></script>
  </body>
</html>
```

```js
// index.js
console.log("执行 index.js");
import addModule from "./add.js";
console.log(addModule.add(1, 2));
```

```js
// add.js
console.log("执行 add.js");
var add = function(a, b) {
  return a + b;
};
export default { add };
```

上面代码运行之后，可以看到控制台的输出：

```
执行 add.js
执行 index.js
3
```

在 index.js 文件中，即使 import...from... 语句没有放在文件开头，依然先加载执行了 add.js 文件，打印出了 `"执行 add.js"`，并将 add.js 文件中 export 出来的内容放在了 addModule 中，**可见 `import...from...` 会被提升到最前面**。

```js
// index.js
console.log("执行 index.js");
import { obj as timeModule } from "./time.js";
console.log(timeModule.getTime());
timeModule.time = 100;
// timeModule.setTime(200);
console.log(timeModule.getTime());
```

```js
// time.js
console.log("执行 add.js");
var obj = {
  time: 0,
  getTime: function() {
    return this.time;
  },
  setTime: function(t) {
    this.time = t;
  }
};

export { obj };
```

上面代码运行之后，可以看到控制台的输出：

```
执行 add.js
执行 index.js
0
100
```

可以看到我们直接在 index.js 中对 timeModule.time 重新赋值，对应的模块中的 time 也会相应变化，可见 import 得到的并不是一份内容的拷贝，只是一个引用。这一点不止对对象适用，对基本数据类型也是一样的。

## Node - CommonJS

```js
// index.js
console.log("执行 index.js");
var addModule = require("./add");
console.log(addModule.add(1, 2));
```

```js
// add.js
console.log("执行 add.js");
var add = function(a, b) {
  return a + b;
};
module.exports.add = add;
```

打印结果：

```
执行 index.js
执行 add.js
3
```

与其他模块使用不同的是，CommonJS 不会提前执行，也不会提前加载，并且是同步加载（这与 Nodejs 应用于服务端有关系）。

## CommonJS 与 ES6Module

1. CommonJS 模块是代码执行时同步加载执行，ES6Module 是编译时输出接口，也会提前执行。
2. CommonJS 引入的模块内容是一份值的拷贝，而 ES6Module 是值的引用，无论是基本数据类型，还是对象。

## 参考

[ES6 系列之模块加载方案](https://juejin.im/post/5bea425751882508851b45d6#heading-0)
