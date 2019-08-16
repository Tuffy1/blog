---
title: js隐式转换
date: 2019-08-10 09:47:33
tags:
---
## 一 发生隐式转换的情况

js的一些运算中，当遇到两种不同类型的操作数值，会进行隐式转换，比如：

```js
1 - '1' // 0
1 - undefined // NaN
1 - null // 1
1 * {} // NaN
1 / [] // Infinity
+ {} // NaN
+ [] // 0
1 + '1' // '11'
1 + {} // '1[Object Object]
1 + null // 1
'1' == true // true
[] == {} // false
[] == 0 // true
{} == [] // Uncaught SyntaxError: Unexpected token ==
{} + [] // 0
[] + {} // "[object Object]"
{} + {} // chrome运行结果:"[object Object][object Object]"; firefox运行结果：NaN
```

## 二 隐式转换过程涉及的三个主要函数

### 1 ToPrimitive(input, type?)

这是一个将值转为原始值的函数。input传的是要转换的值，type可为‘Number’/‘String’两种可能。
此函数的大体操作是：

当 type === ‘Number' 时，

1. （1）input为原始值，直接返回该值；（2）否则执行**input.valueOf()**；
2. （1）上一步所得是原始值，直接返回该值；（2）所得依然是object类型，将所得执行**toString()**；
3. （2）上一步所得是原始值，直接返回该值；（2）所得不是原始值，抛出错误；

当 type ==== 'String' 时，

1. （1）input为原始值，直接返回该值；（2）否则执行**input.toString()**；
2. （1）所得是原始值，直接返回该值；（2）所得依然是object类型，将所得执行**valueOf()**；
3. （2）上一步所得是原始值，直接返回该值；（2）所得不是原始值，抛出错误；

当隐式转换时，一般情况下 type 默认为 Number。除非 input 为 Date 类型，我们可以看一下 Date 类型数据在执行 valueOf() / toString() 这两个函数顺序不同时得到的结果时怎样的：

```js
var date1 = new Date(); // date1: Fri Aug 09 2019 22:42:33 GMT+0800 (中国标准时间)
var valueDate1 = date1.valueOf(); // valueDate1: 1565361753409
var result1 = valueDate1.toString(); // result1: '1565361753409'

var date2 = new Date(); // date2: Fri Aug 09 2019 22:42:33 GMT+0800 (中国标准时间)
var valueDate2 = date2.toString(); // valueDate2: 'Fri Aug 09 2019 22:42:33 GMT+0800 (中国标准时间)'
var result2 = valueDate2.valueOf(); // valueDate2: 'Fri Aug 09 2019 22:42:33 GMT+0800 (中国标准时间)'
```

可以看到，
当使用第一种顺序时，valueOf会将Date类型数据转为毫秒数，而接下来的toString方法实际上是将这个毫秒数转为了字符串；
当使用第二种顺序时，toString方法先将Date类型数据转为了字符串，已经是原始值了，此时再执行valueOf函数依然是返回这个值。
但我们调用ToPrimitive本意常常是要得到原始值，而不希望改变这个值，故执行ToPrimitive函数时，若input为Date类型，默认type为String。

### 2 ToNumber()

此函数的转换规则大体如下：

1. 数字： 返回原值
2. 字符串： 尝试解析，包含非数字字符即得‘NaN’，空字符串为0
3. 布尔值： true为1，false为+0
4. undefined: NaN
5. null: +0
6. object: （1）执行ToPrimitive(input, 'Number')得到原始值；（2）所得通过ToNumber接着转换

回头看篇首的几个例子，二元算术运算符除了‘+’外，都会将符号两边转为number进行运算。
故前面三行的运算相当于：

```js
1 - '1' // 相当于 ToNumber(1) - ToNumber('1') = 1 - 1
1 - undefined // 相当于 ToNumber(1) - ToNumber(undefined) = 1 - NaN
1 - null // 相当于 ToNumber(1) - ToNumber(null) = 1 - 0
```

1 \* {} 相当于 ToNumber(1) \* ToNumber({})
ToNumber(1) = 1
ToNumber({})（1）执行ToPrimitive({}, 'Number'),此函数中先执行toString()得到"[object Object]"，再执行valueOf()得到"[object Object]"。（2）执行ToNumber函数，得到NaN
![pic](/images/pic1.png)
1 \* NaN = NaN，所以结果是NaN

1 / []
相当于 ToNumber(1) / ToNumber([])
ToNumber(1) = 1
ToNumber([])（1）执行ToPrimitive({}, 'Number'),此函数中先执行toString()得到""，再执行valueOf()得到""。（2）执行ToNumber函数，得到0
![pic](/images/pic2.png)
1 / 0 = Infinity，所以结果是Infinity

```js
+ {} // NaN
+ [] // 0
// 一元加号同样是将其转为number，思路与上面相同
```

### 3 ToString()

此函数的转换规则大体如下：

1. 数字： 返回其对应字符串
2. 字符串： 返回原值
3. 布尔值： 返回'true'或'false'
4. undefined: 返回'undefined'
5. null: 返回'null'
6. object: （1）执行ToPrimitive(input, 'String')得到原始值；（2）所得通过ToString接着转换

## 三 + 运算符的几个例子

回头看篇首的几个例子，二元运算符‘+’外，除了可能是算术运算进行加法外，也可能是进行字符串的连接，那么什么时候是加法运算，什么时候是字符串连接呢？

1. 两个操作数都先进行ToPrimitive得到原始值
2. 第一步所得两个都是string，直接字符串连接；
3. 第一步所得有一个是string，另一个ToString转为字符串，再进行字符串连接；
4. 第一步所得两个都不是string，都执行ToNumber转为数字，进行加法运算；

```js
1 + {}
// 首先得到原始值 ToPrimitive(1), ToPrimitive({}) 分别为 1, "[Object Object]"
// 符合第3点，字符串连接

1 + null
// 首先得到原始值 ToPrimitive(1), ToPrimitive(null) 分别为 1, null
// 符合第4点，相当于ToNumber(1) + ToNumber(null) = 1 + 0 = 1

```

## 四 == 不严格相等的几个例子

同类型直接比较，不同类型时：

1. undefined == null且这两个值与其他数值都判断为false
2. Number/String/Boolean基本数据类型间都转为Number后进行比较
3. Object与Number/String/Boolean，Object执行ToPrimitive得到原始值后，基于前两点进行比较

```js
'1' == true // 相当于判断ToNumber('1') == ToNumber(true)

[] == {} // 得到原始值分别是''和'[Object Object]'

{} == [] // Uncaught SyntaxError: Unexpected token ==
// == 前面不能直接写对象，如需比较需要将其赋值给一个变量

[] == 0 // true
// []获取原始值为“”，相当于比较 “” == 0，将两边都ToNumber，相当于 0 == 0
```

## 五 特别的例子：浏览器对代码块和对象字面量的解析

```js
{} == [] // Uncaught SyntaxError: Unexpected token ==
{} + [] // 0
[] + {} // "[object Object]"
{} + {} // chrome运行结果:"[object Object][object Object]"; firefox运行结果：NaN
```

上面几个例子与我们原来设想的结果不一样，这是因为当{}前面没有运算符时，它有可能会被解析为一个代码块，而不是一个对象字面量。

所以：

```js
{} == []
// 相当于
{
};
== []
```

这个时候被认为 == 的使用是语法错误的。

```js
{} + []
// 相当于
{
};
+[]
```

此时就相当于一个ToNumber(\[])，因此所得为0。而\[] + {}中{}前面有运算符，依然被是为是对象字面量而不是代码块，所以可以进行正常的字符串连接。

```js
{} + {}// chrome运行结果:"[object Object][object Object]"; firefox运行结果：NaN
// firefox中应是将它这样解析：
{
};
+ {}

// chrome则是将第一个{}解析为对象字面量
```

{} + {} 在chrome和firefox中的运行结果不同。firefox应是将前面没有运算符的{}都解析为代码块，而chrome却不一定，具体规律是什么就没有深究了。
