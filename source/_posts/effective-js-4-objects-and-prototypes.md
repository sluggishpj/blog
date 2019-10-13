---
title: EffectiveJS笔记04之对象和原型
date: 2017-11-09 20:31:24
tags:
- js
- EffectiveJavaScript
- 对象
- 原型
categories:
- js
- EffectiveJS
---

## `prototype`、`getPrototype`和`__proto__`之间的不同

1、`C.prototype`用于建立由`new C()` 创建的对象的原型

<!-- more -->

2、`Object.getPrototypeOf(obj)` 是ES5 中用来获取obj对象的原型对象的标准方法

3、`obj. __proto__`是获取obj对象的原型对象的非标准方法

``` js
function User(name) {
    this.name = name;
}
var u = new User('hehe');

console.log(Object.getPrototypeOf(u) === User.prototype); // true
console.log(u.__proto__ === User.prototype); // true
```

> 使用`Object.getPrototypeOf`函数而不要使用`__proto__`属性。因为在一些环境中`__proto__`实现不一致（拥有null原型的对象没有`__proto__`属性）。



## 使构造函数与new操作符无关

1、如果调用者忘记使用new关键字，那么函数的接收者将是全局对象

```js
function User(name) {
    this.name = name;
}

var u = User('hehe');
console.log(global.name); // hehe
console.log(u.hehe); // TypeError: Cannot read property 'hehe' of undefined
```

2、如果将User函数定义为严格模式，那么它的接收者默认为undefined

```js
function User(name) {
    'use strict';
    this.name = name;
}

var u = User('hehe');
// 报错
// this.name = name;
//           ^
// TypeError: Cannot set property 'name' of undefined
```

3、不管是否new都能正常工作的函数

```js
// 方法1
function User(name) {
    if (!(this instanceof User)) {
        return new User(name);
    }
    this.name = name;
}

var u = User('hehe');
var u2 = new User('haha');
console.log(u.name); // hehe
console.log(u2.name); // haha
```

```js
// 方法2
function User(name) {
    var self = this instanceof User ? this : Object.create(User.prototype);
    self.name = name;
    return self;
}

var u = User('hehe');
var u2 = new User('haha');

console.log(u.name); // hehe
console.log(u2.name); // haha
```

> Object.create需要一个原型对象作为参数，并返回一个继承自该原型对象的新对象。如下

```js
var animal = {
    sayHi: function() {
        console.log('hi');
    }
};

var people = Object.create(animal);
people.sayHi(); // hi
console.log(Object.getPrototypeOf(people) === animal); // true
```



## 只将实例状态存储在实例对象中

1、理解原型对象与其实例之间是一对多的关系



## 认识到this变量的隐式绑定问题

1、this变量的作用域总是由其最近的封闭函数所确定

```js
// CSV（逗号分隔型取值）文件格式是一种表格数据的简单文本表示，例
// 小明,1996,Beijing,China

function CSVReader(separators) {
    this.separators = separators || [','];
    this.regexp = new RegExp(this.separators.map(function(sep) {
        return '\\' + sep[0];
    }).join('|'));
}

CSVReader.prototype.read = function(str) {
    var lines = str.trim().split(/\n/);
    return lines.map(function(line) {
        console.log(this.regexp); // undefined
        return line.split(this.regexp); // wrong this!
    });
};

var reader = new CSVReader();
console.log(reader.read('a,b,c\nd,e,f\n')); // [ [ 'a,b,c' ], [ 'd,e,f' ] ]
```

2、解决方法一：数组的map方法可以传入一个可选的参数作为其回调函数的this绑定

```js
CSVReader.prototype.read = function(str) {
    var lines = str.trim().split(/\n/);
    return lines.map(function(line) {
        console.log(this.regexp); //  /\,/
        return line.split(this.regexp);
    },this);
};

var reader = new CSVReader();
console.log(reader.read('a,b,c\nd,e,f\n')); // [ [ 'a', 'b', 'c' ], [ 'd', 'e', 'f' ] ]
```

3、解决方法二：使用词法作用域的变量来存储额外的外部this绑定的引用

```js
CSVReader.prototype.read = function(str) {
    var lines = str.trim().split(/\n/);
    var self = this;
    return lines.map(function(line) {
        console.log(self.regexp); //  /\,/
        return line.split(self.regexp);
    });
};

var reader = new CSVReader();
console.log(reader.read('a,b,c\nd,e,f\n')); // [ [ 'a', 'b', 'c' ], [ 'd', 'e', 'f' ] ]
```

## 避免继承标准类

Array是一个很好的例子，一个操作系统的库可能希望创建一个抽象的目录，该目录继承了数组的所有行为。

```js
function Dir(path, entries) {
    this.path = path;
    for(var i = 0, n = entries.length; i<n; i++) {
        this[i] = entries[i];
    }
}

Dir.prototype = Object.create(Array.prototype);

var dir = new Dir('/web/project',['index.html','script.js','style.css']);
console.log(dir.length); // 0

console.log(Object.prototype.toString.call(dir)); // [object Object]
console.log(Object.prototype.toString.call([])); // [object Array]
```

> 失败的原因是length属性只对内部被标记为“真正的”数组的特殊对象起作用。ECMAScript标准规定它是一个不可见的内部属性，称为[[Class]]，仅仅作为一个标签。
>
> 同理，函数被加上了值为“Function”的[[Class]]属性，依此类推



