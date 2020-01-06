---
title: es6 module
date: 2020-01-05 16:47:17
tags:
---

## module 性质

es6 中 module 是编译时加载，故在静态分析阶段不能做到的事情它也不能做，比如不能条件加载。（与 CommonJS 不同，CommonJS 是运行时加载）当运行代码时，模块都加载完毕，故 import 和 export 语句无论是在代码的何处，都相当于提升到文件顶部执行。

## 导入导出默认值

```js
// module.js
let time = new Date().getTime();
const TimeMgr = {
  date: ""
};
export { time as default, TimeMgr };

// 相当于：
// let time;
// export default time = new Date().getTime();

// export const TimeMgr = {
//   date: ""
// };
```

```js
// index.js
import time, { TimeMgr } from "./module";
console.log(time); // 1577669922564
console.log(TimeMgr.data); // ''
```

1. export 导出值需要放在大括号中，如果要视为 default 导出，可使用 as 重命名为 default。export 导出值也可直接在变量前加关键字 export 导出单个，如果要视为 default 导出，则在 export 和变量间加关键字 default，相当于将变量赋给 default。
2. 一个文件导出变量中只能有一个默认值 default。
3. import 引用的变量需要放在大括号中，若不放在大括号中，则是引用对应导出文件的 default 变量

## import 和 export 的复合写法

相当于先进行了 import 然后紧接着 export。

```js
export {rectData} form 'rectEle';
export const area = rectData.width * rectData.height;
```

此模块继承中体现了复合写法：

```js
// rectEle.js
const rectData = {
  width: 10,
  height: 20
};
export { rectData };
```

```js
// rect.js
export { rectData } from "./rectEle";
export default function getArea(w, h) {
  return w * h;
}
// 可视作：
// default = function getArea(w, h) { return w * h;}
// export {default}
```

```js
// index.js
import getArea, { rectData } from "./rect";

console.log(rectData); // { width:10, height: 20 }
console.log(`Rect: ${getArea(rectData.width, rectData.height)}`); // 'Rect: 200'
```

上面例子中，rect.js 导入导出了 rectEle 中的变量 rectData，之后 index.js 中只需导入 rect，也可以读取 rectData。

## 动态引入

```js
// module.js
let time = new Date().getTime();
setTimeout(() => {
  time = new Date().getTime();
}, 1000);

let TimeMgr = {
  date: ""
};

export { time as default, TimeMgr };
```

```js
// index.js
import time, { TimeMgr } from "./module";

console.log(time);
setTimeout(() => {
  console.log(time);
}, 2000);

TimeMgr = "a"; // 报错："TimeMgr" is read-only
TimeMgr.date = new Date(); //对引用的值的某个属性重新进行赋值不会报错
console.log(TimeMgr.date);
```

1. 利用 setTimeout 使被引用的模块内容 time 发生改变，可以看到引用的值也会相应改变。
2. 对 TimeMgr 这个引用的值直接重新进行赋值会报错，因为其有属性 read-only。但对 TimeMgr 的其中一个属性 data 重新赋值不会报错。（不过不建议这么直接赋值，会让错误变得难以定位。可以在被引用文件中写类似函数 updateDate，通过调用函数更新属性值）。
   （与 CommonJS 不同，CommonJS 是值的拷贝）

## 参考

[阮一峰 ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/module)
