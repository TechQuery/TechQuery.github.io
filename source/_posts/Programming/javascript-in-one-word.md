---
title: JavaScript 一句话总结
date: ‎2019‎-01-02 ‏‎23:00:00
author: 南漂一卒
categories:
  - Programming
tags:
  - JavaScript
  - one-word
---


> JavaScript 是一门原生支持**函数式编程**范式的、基于原型的**面向对象**语言，也是一门**弱类型**动态脚本语言


## What's this ?

> JavaScript 函数的 `this` 是由**函数调用者**在调用前确定的 —— 继承自 LISP 语言

用 ECMAScript 6 解释如下：

| 调用方式                  | 调用者      | 等效代码                                                          |
|:------------------------:|:-----------:|:----------------------------------------------------------------:|
| `func(...params)`        | JS 引擎      | `func.apply(null, params)`                                       |
| `obj.func(...params)`    | 一个对象     | `func.apply(obj, params)`                                        |
| `new func(...params)`    | `new` 运算符 | `func.apply(Object.setPrototypeOf({ }, func.prototype), params)` |
| `element.onclick = func` | DOM 事件回调 | `func.call(element, event)`                                      |

上述最后一种其实是**运行时 API** 级别的，与语言本身无关，用 JS 写的公共库需要**回调函数**时，也是库内部调用时如上手动指定的。


## 闭包

通俗解释：

> 孩子靠着自创资产和自己可掌控的父母遗产，继续活下去

技术解释：

> **局部作用域**中创建的函数，若引用了其上级作用域中的变（常）量，又在上级作用域外有引用，上级作用域执行结束被销毁时，此**函数及其引用的数据**形成一个不被销毁的**闭包**。
>
> —— 继承自 LISP 语言

```javascript
var closure = function () {

    const privateData = { };

    return {
        set:  function (key, value) {

            privateData[key] = value;
        },
        get:  function (key) {

            return privateData[key];
        }
    };
};

closure.set('A', 1);

console.log( closure.get('A') );    //  1
```

闭包在 ECMAScript 5 及更早的时代，常用于模拟**块级作用域**、**模块作用域**，在 ECMAScript 6 引入这两种新**局部作用域**后，它们又成了形成闭包的上级作用域之一。


## 原型

> **基于原型的面向对象语言** 可看作把 **基于类的面向对象语言**的**运行时内部构造** 开放了出来

|           | 类                              | 原型                                                                      |
|:---------:|:-------------------------------:|:-------------------------------------------------------------------------:|
| 对象的创建 | 由 `class` 的构造函数修饰 `this` | 直接 `new` 一个函数作为构造函数                                             |
| 对象的继承 | 只知其然，不知所以然             | 对象内部引用**构造函数的原型对象**，在引用对象未定义成员时，在原型上找同名成员   |
| 类的继承   | 只知其然，不知所以然             | `Child.prototype = new Parent()`                                          |
| 私有成员   | 只知其然，不知所以然             | 用**局部作用域**“对外不可访问性”保存的私有 `Symbol`（运行时唯一值）命名对象成员 |


## 异步

> **异步函数**先记下要做什么（传入的参数、回调函数）并交给 JS 引擎所在**运行时平台**的其它线程，然后立马返回，让 JS 自身线程继续执行完后面的**同步代码**，再按异步任务完成的顺序，一一用相应结果数据调用**回调函数**。

所有的异步任务内部都基于回调函数实现：

```javascript
function asyncFunc(callback) {

    setTimeout(callback, 1000);
}

asyncFunc(function () {

    console.log('See you later')
});
```

但需要回调函数的不一定是异步任务：

```javascript
function syncFunc(callback) {

    console.log('See you ' + callback());
}

syncFunc(function () {

    return 'now';
});
```
