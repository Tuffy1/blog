---
title: es6 Map,WeakMap 与 Set,WeakSet
date: 2019-12-18 22:52:16
tags:
---

## Map

Map 与 Object 一些区别：

1. Map 是有序的；
2. Map 有 size 方法；
3. Map 键可以为任意类型，不止 string 和 symbol；
4. Map 有 entries 方法，可使用 for...of 得到键值数组；

## WeakMap

Map 与 WeakMap 区别：

WeakMap 只有 get, set, delete, has 方法，没有遍历相关方法。因为 WeakMap 对其键是弱引用，其键是不可枚举的。

> 强引用：当某变量被强引用时，即使该变量设为 null，该块内容也不会被垃圾回收，因为还存在着引用它的地方。

> WeakMap 的键是弱引用，它不能阻止对应的 key 的内容被垃圾回收，当被回收时，对应的键值对也自动消失。

## Set

Set 是一组值的集合，且其中的元素都是唯一的。像对象那样拥有 keys/values（entries）方法，基本操作方法：add/delete/has/size，同时可以与 Array 相互转换。

```js
// 数组转为Set
const arr1 = [1, 2, 3];
const set1 = new Set(arr1);

// Set转为数组
const arr2 = [...set1];
// or
const arr3 = Array.from(set1);
```

利用 Set 中元素的唯一性以及与数组的互换可以达到数组去重，取交集，取差集等目的。

```js
// 去重
let arr = [1, 2, 2, 3];
arr = [...new Set(arr)]; // [1, 2, 3]

// 取交集
let arr1 = [1, 2, 3, 4, 5];
let arr2 = [2, 3, 5, 6];
let set2 = new Set(arr2);
arr = arr1.filter(i => set2.has(i)); // [2, 3, 5]

// 取差集
arr1 = [1, 2, 3, 4, 5];
arr2 = [2, 3, 5, 6];
set2 = new Set(arr2);
arr = arr1.filter(i => !set2.has(i)); // [1, 4]
```

## WeakSet

Set 与 WeakSet 的关系，与 Map 同 WeakMap 的关系类似，WeakSet 中的值只能存放对象的引用，不能是基本数据类型，同时对其元素只保持了弱引用（不可枚举）。
