---
title: typescript 学习（一）
date: 2019-10-17 22:47:20
tags:
---

## 数据类型

### 一、基本数据类型及数组

基本数据类型有：number，string，boolean，null，undefined，symbol。
数组：两种表示方式如 number[]和 Array <number\>，number 可以换成其他类型，表示数组中的元素类型。
不同类型的变量不能互相赋值。特别的，null 和 undefined 是所有类型的子类型，可以被赋值到其他类型的变量。

```js
// 基本数据类型
let username: string = "tuffy";
let age: number = 12;
const u: undefined = undefined;
const n: null = null;
username = u; // 编译通过：将undefined类型赋值到string类型变量
username = n; // 编译通过：将null类型赋值到string类型变量
username = age; // 编译报错：将number类型赋值到string类型变量

// 数组
let arr: number[];
arr = [1, 2, 3];
arr = null;
arr = [1, null];
let arr1: string[] = undefined;
```

### 二、任意类型

any 表示允许赋值为任意类型。
当声明一个变量并没有指明数据类型时，将会推论为 any。但如果在声明的同时被进行赋值，那么该变量会被推断为该数值的类型。

```js
let notType = "123"; // 推断为 string 类型
notType = 1; // 编译报错:再赋值为 number 类型数值时报错

let anyType; // 推断为 any 类型
anyType = "123"; // 编译通过
anyType = 123; // 编译通过
```

### 三、联合类型

假如知道变量要么是 string 类型要么是 number 类型，不会是别的类型，但不能确定是 string 还是 number 时，可以使用联合类型。

```js
let unionTypes: string | number;
let a: number = unionTypes.length; // 编译报错
let b: string = unionTypes.toString(); // 编译通过

unionTypes = "123";
a = unionTypes.length; // 编译通过
unionTypes = 123;
a = unionTypes.length; // 编译报错
```

### 四、函数

函数的两种定义方式：函数声明和函数表达式。

```js
function add1(a: number, b?: number, c?: number): number {
  ...
}
let add2: (a: number, b?: number, c?: number) => number = function (a: number, b?: number, c?: number): number {
  ...
}
// 此处与 add2 不同，其实并没有直接对 add3 进行类型定义，是通过类型推断所得
let add3 = function (a: number, b?: number, c?: number): number {
  ...
}
```

函数可以定义**可选参数**，但可选参数必须放在必填参数后面，即后面不能再有必填参数。
函数可以定义**剩余参数**，但剩余参数必须放在参数最后，即必须是最后一个参数。

```js
function add(a: number, b?: number, ...items: number[]): number {
  ...
}
```

### 五、类

抽象类（类前面加 abstract ），不能被实例化，一般而言它存在的意义是被子类继承实现。

```js
abstract class Animal {
  public name: string;
  public constructor(name: string) {
    this.name = name;
  }
  public abstract say(); // 抽象类中有抽象方法 say
}
class Dog extends Animal {
  constructor(name: string) {
    super(name);
  }
  public say() { // 继承自 Animal 的类，必须实现其抽象方法，此处若无该函数会报错
    ...
  }
}
let animal: Animal = new Animal('wang'); // 编译报错：无法创建抽象类实例
let dog: Dog = new Dog('wang'); // 编译通过
```

### 六、接口

接口可以描述对象的外形，类的一部分外形。

1. 接口描述对象：当声明一个对象为某接口类型后，该对象属性需与接口属性保持一致。属性类型不能不同，不能多属性，不能少属性（除非接口中指明是可选属性，但这其实也是保持一致的一种方式）。此外，接口中可以由任意属性，即可以不写明属性名。

```js
interface IPerson {
  name: string;
  age: number;
  [prop: string]: any;
  [index: number]: string;
}
let person: IPerson;
person = {
  name: "tuffy",
  age: 12,
  hobby: "123"
};
person[0] = "123";
```

特别的，当定义了某个类型字段名的任意属性后，同类型字段的属性的类型必须是其同类型或其子类型，如下面编译将会报错：

```js
interface IPerson {
  name: string;
  age: number; // 编译报错
  [prop: string]: string;
}
let person: IPerson;
person = {
  name: "tuffy",
  age: 12,
  hobby: "123"
};
```

接口的只读属性，只读是指该接口的这个属性不能被重新赋值，只能通过赋值给这个对象得以更新这个属性。

```js
interface IPerson {
  readonly name: string,
  age: number
}
let person: IPerson;
person.name = 'tuffy'; // 编译报错。即使person还未被赋值过，也不能直接赋值这个属性

interface IPerson {
  readonly name: string,
  age: number
}
let person: IPerson;
person = {
  name: 'tuffy',
  age: 12
};
person = {
  name: 'tuffy1', // 编译通过。这样赋值整个对象是可以的
  age: 12
};
```

2. 接口描述类数组：类数组与数组是不同的，可以结合任意属性描述类数组。

```js
interface IArgument {
  [index: number]: any;
  length: number;
  callee: Function;
}
```

3. 接口描述函数：

```js
interface IAdd {
  (a: number, b?: number, c?: number): number
}
let add: IAdd = function (a: number, b?: number, c?: number): number {
  ...
}
```

```js
interface Person {
  (name: string): string;
  age: number;
  setName(): void;
}
function personFun(): Person {
  const person = <Person>function(name: string): string { return 'tuffy' };
  person.age = 12;
  person.setName = function(): void {};
  return person;
}
```

4. 接口被类实现 与 接口继承类

（1）接口被类实现：

```js
interface IInfo {
  name: string;
}
interface IClasses {
  className: string;
}
class Student implements IInfo, IClasses {
  // 一个类可以实现多个接口
  name: string;
  className: string;
}
```

（2）接口继承类

```js
class InfoManage {
  role: number;
  public setRole(role: number) {}
}
interface IInfo extends InfoManage {
  name: string
}
let info: IInfo = {
  name: 'tuffy',
  role: 1,
  setRole: function (role: number) {}
}
```

### 七、type

1. 当描述对象时，type 与 interface 表现很相似，都可以进行继承，并且两者之间可以互相继承：

```js
// 语法略有不同，type 需要等号
interface IUser1 {
  name: string;
}
type IUser2 = {
  name: string
};
```

```js
// interface 与 type 的继承
interface IUser1 {
  name: string;
}
interface IUserExtend1 extends IUser1 {
  age: number;
}
let userExtend: IUserExtend1 = {
  name: "tuffy",
  age: 123
};

type IUser2 = {
  name: string
};
type IUserExtend2 = IUser2 & {
  age: number
};
let userExtend2: IUserExtend2 = {
  name: "tuffy",
  age: 123
};

// interface 和 type 之间相互继承也是可以的
// interface IUserExtend1 extends IUser2 {}
// type IUserExtend2 = IUser1 & {}
```

2. 不同的是，type 也可以描述一个基本数据类型（即给基本数据类型别名），联合类型

```js
type s = string; // 相当于给了string类型一个别名s
type IUser2 = {
  name: string
};
type IUser3 = s | IUser2;
let typeUser: IUser3;
typeUser = "123"; // 编译通过
typeUser = { name: "123" }; // 编译通过
```

```js
// 利用 type 限制了 str 的几种可能取值
type str = "str1" | "str2" | "str3";
let strNew: str;
strNew = "str1";
strNew = "str";
```

而 interface 的合并，type 则没有：

```js
// 下面是可行的，IUser1 会合并
interface IUser1 {
  name: string;
}
interface IUser1 {
  age: number;
}
let user: IUser1 = {
  name: "tuffy",
  age: 2
};

// 下面是不可行的，会报错
type IUser2 = {
  // 编译报错
  name: string
};
type IUser2 = {
  // 编译报错
  age: number
};
```

### 八、泛型

实现函数、接口或者类的时候，一些属性需要保证是同一类型，但具体是什么类型需要使用时确定，这时可以使用泛型。

```js
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T { // 使得泛型T一定要有属性length
  console.log(arg.length);
  return arg;
}
```

泛型函数：

```js
interface GenericIdentityFn {
  <T>(arg: T): T;
}
function identity<T>(arg: T): T {
  return arg;
}
let myIdentity: GenericIdentityFn = identity;

// 另一种写法
interface GenericIdentityFn<T> {
  (arg: T): T;
}
function identity<T>(arg: T): T {
  return arg;
}
let myIdentity: GenericIdentityFn<number> = identity;
```

### 九、元组

元组是数组，但是指定了 index 对应的元素的类型。

```js
// 当直接给某元组类型的变量赋值时，元素数不能多，也不能少，类型顺序也不能换
type tupleArr1 = [string, number];
let tArr1: tupleArr1;
tArr = ["tuffy", 12]; // 编译通过
tArr = ["tuffy"]; // 编译报错
tArr = ["tuffy", 12, 2019]; // 编译报错

// 可以通过给变量的某index赋值
type tupleArr2 = [string, number];
let tArr2: tupleArr2;
tArr2[0] = "tuffy"; // 编译通过

// 可以通过push等方法，使数量超过
type tupleArr3 = [string, number];
let tArr3: tupleArr3;
tArr3 = ["tuffy", 12];
tArr3.push(2019); // 编译通过
tArr3.push("shenzhen"); // 编译通过
tArr2.push(true); // 编译报错，只能push该元组各index类型的联合类型，意思是这里只能push的类型是“string|number”
```

### 十、类型断言

可以手动指定一个值的类型。语法：<类型>值 或者 值 as 类型。（类型断言不是类型转换）

```js
function getLength(data: string | number): number {
  if ((<string>data).length) { // 此处不加类型断言会编译报错，因为number没有length属性
    return (<string>data).length; // 此处不加类型断言会编译报错，因为number没有length属性
  } else {
    return data.toString().length;
  }
}
```

### 十一、声明语句与声明文件

1. 全局变量声明文件
   当我们使用 <script\></script\> 引入一个第三方库 jquery 时，可以直接使用 \$ 或者 jQuery，虽然使用上是正常的，但 ts 编译时会报错，表示没有找到这个变量，此时可以做一下类似声明 declare var jQuery: (selector: string): any; 来应对编译器的检查，使之知道 jQuery 是一个全局变量不必报错。声明本身只是一个对于编译器的变量及其类型的告知，并没有赋值之类的意义。
   一个文件的声明语句可以放在对应声明文件中，一般以 .d.ts 后缀结尾。

```js
declare var str: string
declare function fun(s: string): string
declare enum Num{}
declare class Person {}
declare namespace Obj {}
interface IProp {} // interface 和 type 前不需要 declare 也可以表示声明
type IType = {}
```

声明是可以进行合并的，但它们需是同一类型，比如都是 string，都是对象等。

```js
declare function fun(s: string): string
declare namespace fun {
  let settting: any
}
fun('321');
fun.settting = '123';
```

下面的则会报错：

```js
declare function fun(s: string): string;
declare var fun: string; // 报错
```

全局变量声明文件不能有 import 或者 export，不然将被视为一个包或者库，那么当该文件需要引入其他库的声明文件或者其他全局变量声明文件时，使用三斜线指令：

```js
/// <reference types="jquery" /> // types用来指明对某一个库的依赖
/// <reference path="helloUtil.d.ts" /> // path用来指明对其他文件的依赖
```

2. 局部变量声明文件
   当 declare 前加上导出 export 时，此时该文件中变量不再是全局变量，而是需要相应文件中 import 才能应用的变量声明。

```js
// ./types/hello.d.ts

declare var str: string
declare function fun(s: string): string
declare enum Num { }
declare class Person { }
declare namespace Obj { }
type IType = {}
export interface IProp { }
```

```js
// ./hello.ts

import { IProp } from "./types/hello";
let prop: IProp = {
  // 不报错
};

fun("321"); // 报错
fun.settting = "123";
```

如果在局部变量声明文件中，某个变量希望被扩展成全局变量，可以使用下面方法：

```js
// ./types/hello.d.ts
declare global { var num: number }
declare function fun(s: string): string
export interface IProp { }
```

```js
// ./hello.ts

import { IProp } from "./types/hello";
let prop: IProp = {
  // 不报错
};

fun("321"); // 报错
fun.settting = "123";

num = 123; // 不报错
```
