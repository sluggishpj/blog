---
title: understanding-prototype-in-js
date: 2018-06-28 23:14:22
tags:
- js
- prototype
- __proto__
- setPrototypeOf
- getPrototypeOf
categories:
- js
---

## 目的

理清楚 `__proto__`，`prototype`，`Object.getPrototypeOf/Object.setPrototypeOf`，以及**继承与原型**间的关系

<!-- more -->



## 简介

* `__proto__`，只有**对象**才有的属性，即**原型**，不建议直接通过 `__proto__` 获取，建议通过 `Object.getPrototype` 获取。根据ECMAScript标准，使用 `someObject.[[Prototype]]` **符号**指代 `someObject` 的原型 
* **原型链**是通过 `[[Prototype]]` 不断向上连接
* `prototype`，只有**函数**才有的属性，默认为空对象`{}`。对象也可以手动设置其值，但跟普通的键没区别
* `Object.setPrototypeOf(A, B)`，将A的原型设置为B，等价于 `A.__proto__ = B`
* `Object.getPrototypeOf(A)`，获取A的原型的**标准**方法
  * 当原型不为 `null` 时，等价于 `A.__proto__` 
  * 当原型为 `null` 时，通过 `obj.__proto__` 获取到的值为 `undefined`，但通过 `Object.getPrototypeOf(obj)`获取到的指为 `null`
* `__proto__ ` 可以直接设置对象字面量的原型

```js
let pro = {
    name: 'NAME'
}

let obj = {
    __proto__: pro
}

console.log(obj.name) // NAME
```



## 图解

* 说明：`Shape` 是父类，`Triangle` 是子类，`Triangle` 继承自 `Shape` 。图中所有的 `__proto__` 均指通过 `Object.getPrototype` 获取得到的原型
* 注意：当通过 `extend` 继承时，`Triangle.prototype.__proto__ === Shape.prototype`，当通过`setPrototypeOf` 方法继承时，`Triangle.prototype.__proto__ === Object.prototype`

![原型图](https://s2.ax1x.com/2019/10/13/uxRkVO.png)

