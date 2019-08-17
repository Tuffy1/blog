---
title: js继承
date: 2019-08-16 22:32:28
tags:
---
## 构造函数

首先看一个具体的例子：

```js
function Animal(s) {
  this.sound = s;
}

Animal.prototype.getSound = function() {
  console.log(this.sound);
}

var dog = new Animal('WangWang');
dog.getSound(); // 'WangWang'

var cat = new Animal('MiaoMiao');
cat.getSound(); // 'MiaoMiao'
```

上面的例子中，我们写了一个方法 Animal ，以及对应创建了其两个实例 dog 和 cat，创建实例的过程中执行了 Animal 构造函数，并且构造函数中的 this 会指向实例对象，故两个实例都获得了自身的 sound 属性，而 Animal 上 prototype 的方法则作为各个实例的公有方法。

## 调用规则

!['js继承1.png'](/images/js继承1.png)

Animal.prototype 有共有函数 getSound，和一个指向Animal 构造函数的 constructor。dog 实例中有私有属性 sound，同时还有一个对象 __proto__,其中与 Animal.prototype 中内容完全相同，严格相等结果为 true。

当调用实例的某个属性时，首先会在该对象内部查找该属性，如果查找不到，则会在对象的 __proto__ 中找。
而我们从 dog.__proto__ === Animal.prototype 中知道，在对象的 __proto__ 查找，实际上是在其对象的原型（Animal.prototype）中查找。从继承的角度来看，如果在 Animal.prototype 中找不到，我们希望它再往上找，那么如果可以在 dog.__proto__ 中，还有再往上一级的 __proto__，以此类推，我们就可以一直沿着原型链往上找：

```js
// dog
sound: 'WangWang'
__proto__: // 对象原型
  getSound: f()
  constructor: f()
  __proto__: // 再上一级对象原型
    ...
    ...
    constructor: f()
    __proto__: // 再上一级对象原型
      ...
      ...
```

## 原型链继承

现在我们希望基于 Animal 创建子类 Dog：

```js
function Animal(s) {
  this.sound = s;
}

Animal.prototype.getSound = function() {
  console.log(this.sound);
}

function Dog(dogProperty) {
  this.dogProperty = 'dogProperty';
}

Dog.prototype.getDogProperty = function() {
  console.log(this.dogProperty);
}
var dog = new Dog('i am a dog');
dog.getDogProperty(); // 'WangWang'
```

上面代码中，此时 Dog 的原型为：
![js继承2.png](/images/js继承2.png)
我们希望 Dog.prototype 可以指向上一级。
基于上面的规则，原型链继承中，我们令对象原型等于上一级的实例对象，即 Dog.prototype = new Animal()，那么此时的 Dog 的原型为：
![js继承3.png](/images/js继承4.png)
此时我们在 Dog 的原型中可以去得到上一级的所有属性（包括私有属性sound）和方法。通过这种赋值，实例 dog 可以在 __proto__ 中不断找到上一级的属性和方法。

但是到目前为止的方法中，存在着两个缺点：

1. 通过 Dog.prototype = new Animal() 赋值得到新的 Dog 原型，之后再创建 Dog 的实例时，无法向 Animal 的构造函数传值，也就是我在创建实例 dog 时，无法对该实例进行 sound 这个属性的赋值。

2. 从图中也可以看到，sound 属性是放在 __proto__ 中的，意味着它是一个所有实例共用的属性，比如我现在再创建一个 dog2 实例：

```js
function Animal(s) {
  this.sound = s;
  this.color = ['red', 'black'];
}

Animal.prototype.getSound = function() {
  console.log(this.sound);
}

function Dog(dogProperty) {
  this.dogProperty = 'dogProperty';
}
Dog.prototype = new Animal();
Dog.prototype.getDogProperty = function() {
  console.log(this.dogProperty);
}

var dog = new Dog('i am a dog');
var dog2 = new Dog('i am dog2');
dog.color.push('white');
dog2.color // ['red', 'black', 'white']
```

可以看到，当其中一个实例修改了父类中的一个引用属性时，所有的实例的该属性都被修改了。因为这个属性并没有在每个实例对象中创建一个。即使是非引用型的属性，虽然不会修改到所有实例的相关属性，但其本质也是在实例上多定义了一个属性，这也不是我们期望的效果。

## 借用构造函数继承

那么我们需要父类的私有属性也可以正常被所有子类实例创建，可以借助构造函数继承，即在子类构造函数中调用父类的构造函数：

```js
function Animal(s) {
  this.sound = s;
  this.color = ['red', 'black'];
}
function Dog(s) {
  Animal.call(this, s);
}
var dog = new Dog('dogdog');
var dog1 = new Dog('dog1');

dog.sound // 'dogdog'
dog.color.push('white') // 3

dog1.sound // 'dog1'
dog1.color // ['red', 'black']
```

可以看到，创建实例 dog 和 dog1 时，执行 Dog 构造函数，而在 Dog 构造函数中，会将 this(即实例对象本身) 为参，用 call 调用其父类构造函数，这样两个实例对象都拥有了父类的私有属性，并且彼此间互不干扰。
![js继承5](/images/js继承5.png)
可以看到现在属性 color 和 sound 都是直接在 dog 下，而不是在 __proto__ 中。

但如果只有构造函数，那么就只能得到父类的私有属性，却不能得到其在原型上的方法和属性。

## 组合继承

既然上面的两个方法各实现了一部分目的，那么如果将它们组合起来使用，是不是就可以得到我们想要的效果：

```js
function Animal(s) {
  this.sound = s;
  this.color = ['red', 'black'];
}
Animal.prototype.getSound = function() {
  console.log(this.sound);
}

function Dog(s) {
  Animal.call(this, s);
}
Dog.prototype = new Animal();

var dog = new Dog('dogdog');
var dog1 = new Dog('dog1');

dog.color.push('white'); // 3
dog.getSound(); // 'dogdog'

dog1.color // ['red', 'black', 'white']
dog1.getSound(); // 'dog1'
```

这样一来，我们确实可以让实例可以拥有独立的属性，同时也可以调用各级原型上的方法。

![js继承6](/images/js继承6.png)

但是这样父类的构造函数执行了两次，可以看到 dog 实例中拥有两份相同的属性 color/sound。
其实我们最终期待的效果是，color/sound 直接存在对象下，而 __proto__ 则不需要这两个属性，只存原型上的方法。

## 寄生组合式继承

基于上面的方法，我们希望它可以拥有父类对象上的属性（通过call执行构造函数），同时又可以得到其原型上的方法（Animal.prototype）而不要再次执行构造函数。
寄生组合式继承的处理措施是：创建一个临时的函数对象来得到父类的原型，但不复制其构造方法。

```js
function Animal(s) {
  this.sound = s;
  this.color = ['red', 'black'];
}
Animal.prototype.getSound = function() {
  console.log(this.sound);
}

function Dog(s) {
  Animal.call(this, s);
}

var F = function() {}; // 创建一个临时的函数对象
F.prototype = Animal.prototype;
Dog.prototype = new F();
// 不同于“组合继承”，这里不是直接 Dog.prototype = new Animal();故不会在这里再次执行 Animal 构造函数，同时又成功获得原型上的方法
// 不能直接 Dog.prototype = Animal.prototype，会使它们指向同一个对象

var dog = new Dog('dogdog');
var dog1 = new Dog('dog1');

dog.color.push('white'); // 3
dog.getSound(); // 'dogdog'

dog1.color // ['red', 'black', 'white']
dog1.getSound(); // 'dog1'
```

此时 Dog.prototype 如下：
![js继承7](/images/js继承7.png)
此外由于对Dog.prototype重新赋值了，使得Dog.prototype中的constructor丢失，我们需要对constructor进行重新赋值

```js
var F = function() {}; // 创建一个临时的函数对象
F.prototype = Animal.prototype;
Dog.prototype = new F();
Dog.prototype.constructor = Dog;
```

此时 Dog.prototype 和实例 dog 如下：
![js继承8](/images/js继承8.png)

这个时候已经得到我们期待中的对象结构
