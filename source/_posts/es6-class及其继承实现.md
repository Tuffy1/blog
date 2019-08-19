---
title: es6 class及其继承实现
date: 2019-08-19 22:34:22
tags:
---
## class的实现

传统生成实例对象的方法是通过构造函数，继承通过原型链。es6中的class对此部分内容进行了一个语法糖包装。

### es6代码

写一个class的简单例子如下：

```js
class Parent {
  constructor(p) {
    this.p_property = p; // 构造函数：实例属性p_property
  }
  all_property = 'all';
  static static_property = 'static';
  getParentProperty() { // 原型方法：getParentProperty
    console.log(this.p_property);
  }
  static getPrologue() { // 静态方法：getPrologue
    console.log('hello world');
  }
}

var p = new Parent('it is a parent property'); // 创建一个Parent实例对象p
Parent.getPrologue(); // 调用Parent的静态函数
p.getParentProperty(); // 调用实例对象方法
```

### 转为es5代码

将上面的代码转为es5如下:

```js
'use strict';

var _createClass = function () {
  function defineProperties(target, props) {
    for (var i = 0; i < props.length; i++) {
      var descriptor = props[i];
      descriptor.enumerable = descriptor.enumerable || false;
      descriptor.configurable = true;
      if ("value" in descriptor) descriptor.writable = true;
      Object.defineProperty(target, descriptor.key, descriptor);
    }
  }
  return function (Constructor, protoProps, staticProps) {
    if (protoProps) defineProperties(Constructor.prototype, protoProps);
    if (staticProps) defineProperties(Constructor, staticProps);
    return Constructor;
  };
}();

function _classCallCheck(instance, Constructor) {
  if (!(instance instanceof Constructor)) {
    throw new TypeError("Cannot call a class as a function");
  }
}

var Parent = function () {
  function Parent(p) {
    _classCallCheck(this, Parent);

    this.all_property = 'all';

    this.p_property = p; // 构造函数：实例属性p_property
  }

  _createClass(Parent, [{
    key: 'getParentProperty',
    value: function getParentProperty() {
      // 原型方法：getParentProperty
      console.log(this.p_property);
    }
  }], [{
    key: 'getPrologue',
    value: function getPrologue() {
      // 静态方法：getPrologue
      console.log('hello world');
    }
  }]);

  return Parent;
}();
Parent.static_property = 'static';

var p = new Parent('it is a parent property'); // 创建一个Parent实例对象p
Parent.getPrologue(); // 调用Parent的静态函数
p.getParentProperty(); // 调用实例对象方法
```

代码中除了最后的创建实例对象和调用方法，上面主要用到了变量_createClass，方法_classCallCheck，变量Parent

一、各函数的作用：

1. **_createClass**是一个立即执行的函数，返回值是一个匿名函数。该返回值函数接收三个参数Constructor（构造函数）, protoProps（原型方法）, staticProps（静态方法）；方法中将循环protoProps（原型方法）数组，将其一一放在Constructor.prototype中，成为构造函数的原型方法，循环staticProps（静态方法）数组，将其一一放在Constructor本身，成为静态方法，使得其可以由构造函数调用而不能被实例对象调用。
2. **classCallCheck**是一个检查函数，检查传入的对象是否是通过new操作符得到的。因为通过new操作符得到的对象的\__proto__等于构造函数的prototype，而如果是直接调用构造函数（比如如果这里是：Parent('it is a parent property')调用了Parent，而不是new操作符）。如果不是new，则抛出错误。
3. **Parent**是一个立即执行的函数，返回值是Parent构造函数。该立即执行函数会先执行_createClass方法，将所有的原型方法和静态方法都相应定义在构造函数原型和其本身上。

二、代码的执行：

1. 执行_createClass函数，该变量指向一个匿名函数；
2. 执行Parent函数，构造函数上设置了原型方法和静态方法，最后该变量指向同名构造函数；
3. 设置上构造函数的静态属性；
4. 执行p = new Parent('it is a parent property')创建实例，执行第2步中Parent构造函数，先调用classCallCheck检查对象是否为new操作符所得，再将实例对应的属性更新；
5. 调用构造函数静态方法：Parent.getPrologue();
6. 调用实例对象方法：p.getParentProperty();

值得注意的是，当我们将属性all_property写在构造函数之外时，它依然是作为实例属性，就像写在构造函数中一样。而如果要设置构造函数的静态属性，需要在前面指明static，如上面的static_property，如此就只有构造函数对象可以调用它，而实例对象不行。

## class继承的实现

还是基于上面的例子，写一个子类Child继承Parent。

### es6代码

```js
class Parent {
  constructor(p) {
    this.p_property = p; // 构造函数：实例属性p_property
  }
  getParentProperty() { // 原型方法：getParentProperty
    console.log(this.p_property);
  }
  static getPrologue() { // 静态方法：getPrologue
    console.log('hello world');
  }
}

// var p = new Parent('it is a parent property'); // 创建一个Parent实例对象p
// Parent.getPrologue(); // 调用Parent的静态函数
// p.getParentProperty(); // 调用实例对象方法

class Child extends Parent {
  constructor(p, c) {
    super(p);
    this.c_property = c;
  }
  getChildProperty() { // 原型方法：getParentProperty
    super.getParentProperty();
    console.log(`p: ${this.p_property}, c: ${this.c_property}`);
  }
}

var c = new Child('super parent', 'it is a child property');
c.getChildProperty();
c.getParentProperty();
```

### 转为es5代码

将上面的代码转为es5如下:

```js
'use strict';

var _get = function get(object, property, receiver) {
  if (object === null) object = Function.prototype;
  var desc = Object.getOwnPropertyDescriptor(object, property);
  if (desc === undefined) {
    var parent = Object.getPrototypeOf(object);
    if (parent === null) {
      return undefined;
    } else {
      return get(parent, property, receiver);
    }
  } else if ("value" in desc) {
    return desc.value;
  } else {
    var getter = desc.get;
    if (getter === undefined) {
      return undefined;
    }
    return getter.call(receiver);
  }
};

var _createClass = function () { function defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } } return function (Constructor, protoProps, staticProps) { if (protoProps) defineProperties(Constructor.prototype, protoProps); if (staticProps) defineProperties(Constructor, staticProps); return Constructor; }; }();

function _possibleConstructorReturn(self, call) {
  if (!self) {
    throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
  }
  return call && (typeof call === "object" || typeof call === "function") ? call : self;
}

function _inherits(subClass, superClass) {
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass, enumerable: false, writable: true, configurable: true
    }
  });
  if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
}

function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var Parent = function () {
  function Parent(p) {
    _classCallCheck(this, Parent);

    this.p_property = p; // 构造函数：实例属性p_property
  }

  _createClass(Parent, [{
    key: 'getParentProperty',
    value: function getParentProperty() {
      // 原型方法：getParentProperty
      console.log(this.p_property);
    }
  }], [{
    key: 'getPrologue',
    value: function getPrologue() {
      // 静态方法：getPrologue
      console.log('hello world');
    }
  }]);

  return Parent;
}();

// var p = new Parent('it is a parent property'); // 创建一个Parent实例对象p
// Parent.getPrologue(); // 调用Parent的静态函数
// p.getParentProperty(); // 调用实例对象方法

var Child = function (_Parent) {
  _inherits(Child, _Parent);

  function Child(p, c) {
    _classCallCheck(this, Child);

    var _this = _possibleConstructorReturn(this, (Child.__proto__ || Object.getPrototypeOf(Child)).call(this, p));

    _this.c_property = c;
    return _this;
  }

  _createClass(Child, [{
    key: 'getChildProperty',
    value: function getChildProperty() {
      // 原型方法：getParentProperty
      _get(Child.prototype.__proto__ || Object.getPrototypeOf(Child.prototype), 'getParentProperty', this).call(this);
      console.log('p: ' + this.p_property + ', c: ' + this.c_property);
    }
  }]);

  return Child;
}(Parent);

var c = new Child('super parent', 'it is a child property');
c.getChildProperty();
c.getParentProperty();

```

一、主要函数的作用：

1. **_get**:寻找要调用的方法，如果在子类中找不到，会在父类的原型中（此时Child.\_\_proto__等于父类原型）找，如果父类找不到，就再往父类的父类找，沿着原型链往上找。（具体原理可参考上一篇文章[js继承](/2019/08/16/js继承/)）
2. **_possibleConstructorReturn**:执行完父类构造函数后，会执行此函数。将实例本身和执行完父类构造函数的返回值作为参数，如果父类构造函数返回了一个对象，则此函数返回该对象，否则返回实例本身。
3. **_inherits**:通过创建一个对象，让对象的prototype等于父类的prototype，再将此对象赋值给子类prototype，实现子类继承父类原型上的方法（具体原理可参考上一篇文章[js继承](/2019/08/16/js继承/)）。同时将父类构造函数赋值给子类属性\_\_proto__。

二、代码的执行：

1. 执行_createClass函数，该变量指向一个匿名函数；
2. 执行Parent函数，构造函数上设置了原型方法和静态方法，最后该变量指向同名构造函数；
3. 执行Child函数，先调用_inherits函数得到父类上的原型方法，再在构造函数上设置了自身原型方法（和静态方法），最后改变量指向同名构造函数；
4. 执行c = new Child('super parent', 'it is a child property')创建实例，执行第2步中Child构造函数，（1）先调用_classCallCheck函数检查对象是否由new操作符所得；（2）调用_possibleConstructorReturn函数，对应执行父类构造函数（super()语句），得到父类上的实例属性；（3）设置子类的属性；
实例对应的属性被更新；
5. 调用子类方法：c.getChildProperty();
6. 调用父类方法：c.getParentProperty();
