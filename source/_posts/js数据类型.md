---
title: js数据类型
date: 2019-08-07 23:03:53
tags:
---

## 一 分类

JS 的七种基本数据类型：Number、String、Boolean、null、undefined、Obejct、Symbol（本文暂不涉及Symbol的解释）。

对这六种数据类型，可以有不同的分类：“按存储类型分类”，“按是否可以拥有方法分类”

### 1 按存储类型分类

!['数据类型.png'](/images/data-type.png)
基本类型的数据大小是固定的，数值直接保存在栈内存中。

```js
var num = 123;
var str = 'string';
var bool = true;
```

基本数据类型数值都是不可改变的
疑问1：String类型的slice等方法不改变原数值吗？

```js
var str = 'the first string';
str.slice(4); // 'first string'
console.log(str); // 'the first string'

// String类型的方法都是生成一个新的字符串，不会改变原字符串的值
```

疑问2：赋值操作时不是改变数值了吗？

```js
var num = 1;
var num = 2;

// 此操作只是给变量重新赋值，基本数据的数值本身并没有被改变
```

引用类型的数据是存放在堆内存中的，栈中存放指向对应堆内存地址的指针。
引用类型数据的值是可以被修改的

```js
var obj = { name: 'tuffy', age: 1 };
obj.age = 20;
console.log(obj); // {name:'tuffy', age: 20}
```

### 2 按是否可以拥有方法分类

!['数据类型2.png'](/images/data-type2.png)

## 二 判断

### 1 typeof

typeof判断结果有几种情况：symbol/number/string/boolean/undefined/object/function
值得注意的：

```js
typeof null === 'object' // 从一开始出现JavaScript就是这样的

typeof function() {} === 'function'
typeof class C{} === 'function'
typeof new Function() === 'function'

typeof new Number(1) === 'object'
typeof new String('a') === 'object'
typeof new Boolean(true) === 'object'
```

可以看到，引用类型的数据除了“function”外都是“object”，判断得到“object”的情况可细分多种：数组、普通对象、日期对象、部分new操作所得...
故通过typeof判断后要进一步获取是否为数组，可使用Array.isArray

```js
var arr = [1, 2, 3];
var obj = { name: 'tuffy', age: 20 };
typeof arr; // "object"
typeof obj; // "object"
Array.isArray(arr); // true
Array.isArray(obj); // false
```

原理：变量在存储时，在其机器码的低3位可以分辨其类型：

- 000 对象
- 010 浮点数
- 100 字符串
- 110 布尔值
- 1 整数

null 和 undefined 比较特殊，undefined 为 -2^30，null 为全 0。故 typeof 判断类型时

```js
JS_PUBLIC_API(JSType)
JS_TypeOfValue(JSContext *cx, jsval v)
{
    JSType type = JSTYPE_VOID;
    JSObject *obj;
    JSObjectOps *ops;
    JSClass *clasp;

    CHECK_REQUEST(cx);
    if (JSVAL_IS_VOID(v)) { // 是否为undefined
      type = JSTYPE_VOID;
    } else if (JSVAL_IS_OBJECT(v)) { // 是否为对象
      obj = JSVAL_TO_OBJECT(v);
      if (obj &&
        (ops = obj->map->ops,
        ops == &js_ObjectOps
        ? (clasp = OBJ_GET_CLASS(cx, obj),
        clasp->call || clasp == &js_FunctionClass)
        : ops->call != 0)) { // 是否为function
          type = JSTYPE_FUNCTION;
      } else {
        type = JSTYPE_OBJECT;
      }
    } else if (JSVAL_IS_NUMBER(v)) { // 是否为number
      type = JSTYPE_NUMBER;
    } else if (JSVAL_IS_STRING(v)) { // 是否为string
      type = JSTYPE_STRING;
    } else if (JSVAL_IS_BOOLEAN(v)) { // 是否为boolean
      type = JSTYPE_BOOLEAN;
    }
    return type;
}
```

typeof 可以正常判断基本数据类型，但是object只能判断是否为function，同时null也判断为object。
typeof 可以对基本类型进行直接判断返回，但得到object的情况多种，具体是哪种类型的object还需进行进一步判断

### 2 instanceof

instanceof通过原型进行判断，如 a instanceof A 通过判断A的prototype属性是否在a的原型链上的任何位置，若在则返回true，不在则返回false。

原理：

```js
instanceof = function (object, cons) {
  let obj_proto = object.__proto__;
  while(obj_proto) {
    if (obj_proto === cons.prototype) {
      return true;
    }
    obj_proto = obj_proto.__proto__;
  }
  return false;
}
```

```js
var date = new Date();
date instanceof Date; // true
date instanceof Object; // true
// 从原型链上看，date的_proto_等于Date的prototype，通过原型链的回溯查找可以得到date的_proto_间接指向Object.prototype，故date instanceof Object结果也为true

var str = 'str';
str instanceof String; // false
```

只有对象才有原型，故基本类型数据不能用此方法进行判断

### 3 Object.prototype.toString

此方法返回this的具体类型，返回格式为'[Object,xxx]'

```js
// 由于toString是Object原型上的一个方法，为了让各个类型都可以正常执行Object上的toString方法，使用call来改变this指向
Object.prototype.toString.call(new Function()); // [Object Function]
Object.prototype.toString.call(new Date()); // [Object Date]
Object.prototype.toString.call([]); // [Object Array]
```

### 4 constructor

constructor 可以判断变量是否为某种类型

```js
var bar = 'str';
bar.constructor === String; // true
bar = 1;
bar.constructor === Number; // true
bar = null;
bar.constructor === Object; // 报错：Uncaught Type
bar = undefined;
bar.constructor; // 报错：Uncaught Type
```

当使用constructor去检测未知变量是否为某种类型时，如果未知变量为null或者undefined，使用该方法会报错。
该方法也可用于判断自定义对象，但当自定义对象重写了prototype时，原有的constructor会丢失

```js
function C() {};
C.prototype = { name: 'tuffy' };
var c = new C();
c.constructor === C; // false
```
