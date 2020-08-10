---
title: 讲烂了的 JavaScript 原型继承
date: 2020-08-10 20:16:00
categories:
  - Programming
tags:
  - JavaScript
  - ECMAScript
  - OOP
---

## 创建普通对象

### 对象字面量

```javascript
var object = {
  public_property: 1,
  public_method: function () {
    return this.public_property;
  }
};
```

### 内置构造函数

```javascript
var object = new Object();
object.public_property = 1;
object.public_method = function () {
  return this.public_property;
};
```

## 创建对象实例

### 工厂模式（闭包）

```javascript
function factory() {
  var private_property = 1;

  function private_method() {
    return private_property;
  }

  return {
    public_property: 1,
    public_method: function () {
      return [this.public_property, private_method()];
    }
  };
}

var object = factory();
```

### 构造函数

```javascript
function Class() {
  this.public_property = 1;
}

Class.prototype.public_method = function () {
  return this.public_property;
};

var object = new Class();
```

```javascript
Object.create = function (prototype) {
  function Temp() {}
  Temp.prototype = prototype;

  return new Temp();
};

var object = Object.create(Class.prototype);
```

## 子类继承父类

### 父类修饰子类

```javascript
function Parent() {
  this.public_property = 1;
}

Parent.prototype.public_method = function () {
  return this.public_property;
};

function Child() {
  Parent.call(this);

  this.public_property = 1;
}

Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

Child.prototype.public_method = function () {
  return Parent.prototype.public_method.call(this) * 2;
};

var parent = new Parent(),
  child = new Child();
```

```javascript
var outer = {};
var inner = Function.call(outer, 'return 1;');

inner(); // 1
outer(); // Uncaught TypeError: outer is not a function
```

### 子类修饰父类

```javascript
function Child() {
  var that = Object.setPrototypeOf(
    new (Parent.bind.apply(null, arguments))(),
    Child.prototype
  );

  that.public_property = 1;

  return that;
}
```

```javascript
function Child() {
  var that = Object.setPrototypeOf(new Parent(...arguments), Child.prototype);

  that.public_property = 1;

  return that;
}
```

<iframe
    style="width: 100%; height: 85vh"
    loading="lazy" scrolling="no" frameborder="no" allowtransparency="true" allowfullscreen="true"
    title="Prototype inheritance"
    src="https://codepen.io/tech_query/embed/wvMZdoX?height=548&theme-id=dark&default-tab=js,result"
>
  See the Pen
  <a href='https://codepen.io/tech_query/pen/wvMZdoX'>Prototype inheritance</a>
  by TechQuery
  (<a href='https://codepen.io/tech_query'>@tech_query</a>) on
  <a href='https://codepen.io'>CodePen</a>.
</iframe>
