---
title: defineProperty 与 Proxy
date: 2019-12-20 21:41:17
tags:
---

## 基本用法

### defineProperty

Object.defineProperty(obj, prop, descriptor);
obj: 对象。要定义属性的对象。
prop：字符串。要定义的属性名称。
descriptor：描述符。configurable(默认 false)，enumerable(默认 false)，[value（默认 undefined）, writable（默认 false）]/[get（getter 方法）, set（setter 方法）]。

注意点：

1. [value, writable] 与 [get, set] 不能同时存在。
2. writable 为 true 时， value 才能被重新赋值。
3. configurable 表示对象能否被删除，以及除 writable 和 value 以外的属性能否被改变。

value 与 writable：

```js
const obj1 = {};
Object.defineProperty(obj1, "name", {
  value: "tuffy"
});
obj1.name; // 'tuffy'
obj1.name = "joyee";
obj1.name; // 'tuffy',值没有被重新赋值，因为writable默认为false

const obj2 = {};
Object.defineProperty(obj2, "name", {
  value: "tuffy",
  writable: true
});
obj2.name; // 'tuffy'
obj2.name = "joyee";
obj2.name; // 'joyee'，成功重新赋值
```

get 与 set

```js
const obj = {};
let value = "tuffy";
Object.defineProperty(obj, "name", {
  get: () => {
    return value;
  },
  set: v => {
    document.getElementById("name").innerHTML = v; // 绑定dom，当obj.name被重新赋值时，相应的dom内容也自动修改
    value = v;
  }
});
obj.name; // 'tuffy'
obj.name = "joyee";
obj.name; // 'joyee'
```

### Proxy

let p = new Proxy(target, handler)
target: 对象/函数/代理对象。要进行代理的目标。
handler: 对象。定义代理行为。

注意点：

1. Proxy 参数 handler 定义了代理的操作，但代理的对象是 p，不是 target。
2. Proxy 可以代理 13 中操作。包括 get, set, has, apply 等。

举例 get 和 set 的使用：

```js
const obj = { name: "tuffy" };
const p = new Proxy(obj, {
  /**
   * target: 目标对象，这里指obj
   * prop：属性名
   * receiver：代理实例，这里指p
   **/
  get: (target, prop, receiver) => {
    if (!target[prop]) {
      throw new Error("not this prop"); // 如果企图获取不存在的属性，报出错误
    }
    return `This is ${target[prop]}`;
  },
  /**
   * target: 目标对象，这里指obj
   * prop：属性名
   * value: 属性值
   * receiver：代理实例，这里指p
   **/
  set: (target, prop, value, receiver) => {
    target[prop] = value;
  }
});

p.name; // 'This is Tuffy'
p.age; // Uncaught Error: not this prop
p.age = 18;
p.age; // 18
```

## 相似与不同

defineProperty 和 Proxy 都可以拦截对象的赋值与获取，可以定义 get 和 set 操作执行的函数，实现赋值前验证等功能。但：

1. defineProperty 拦截的是对目标对象本身的操作，而 Proxy 则是对代理对象，对目标对象不起作用。
2. Proxy 代理的操作有 13 中，而 defineProperty 只有 get 和 set。
3. Proxy 更侧重于定义操作，而 defineProperty 可更改描述符。

## 延伸：Reflect

内置对象 Reflect 有 13 个静态方法，如 Reflect.set(), Reflect.get(), Reflect.apply() 等。

注意点：

1. 上述所说的方法都是静态方法，即都是通过 Reflect.xxx 来调用。
2. Reflect 并不是一个构造函数，不能通过 new 运算符获得实例。
3. Reflect 的 13 个方法与 Proxy 的代理操作一一对应，两者可配合使用。通过 Proxy 代理操作，通过 Reflect 实际操作对象相关属性等。

基本使用举例：

```js
const obj = {};
Reflect.set(obj, "name", "tuffy"); // true
Reflect.get(obj, "name"); // 'tuffy'

// 与 Proxy 结合使用
const obj = { name: "tuffy" };
const p = new Proxy(obj, {
  get: (target, prop, receiver) => {
    if (!target[prop]) {
      throw new Error("not this prop"); // 如果企图获取不存在的属性，报出错误
    }
    return `This is ${Reflect.get(target, prop)}`;
  },
  set: (target, prop, value, receiver) => {
    Reflect.set(target, prop, value);
  }
});
```

### Reflect 与 Object

1. 一些返回结果的不同；
2. 都为函数调用，而不是对 obj 的命令操作。

```js
let obj = 1;
obj["name"] = "tuffy"; // 'tuffy',对number类型属性赋值，并没有报错
obj; // 1，其实赋值没有成功

let obj = 1;
Reflect.set(obj, "name", "tuffy"); // Uncaught Error: Reflect.set called on non-object
```
