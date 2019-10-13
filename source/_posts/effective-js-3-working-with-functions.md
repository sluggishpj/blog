---
title: EffectiveJS笔记03之使用函数
date: 2017-11-07 23:42:58
tags:
- js
- EffectiveJavaScript
- 函数
categories:
- js
- EffectiveJS
---

## 函数调用，方法调用，构造函数调用

1、方法不过是对象的属性恰好是函数而已

2、一个非方法的函数调用会将全局对象作为接收者，将this绑定到全局对象上。而严格模式下会将this变量的默认值改为undefined

<!-- more -->

``` js
function hello() {
    return 'hello ' + this.username;
}

function hi() {
    'use strict';
    return 'hi ' + this.username;
}
var obj = {
    hello: hello,
    hi: hi,
    username: 'bob'
}
console.log(obj.hello()); // hello bob
console.log(hello()); // hello undefined

console.log(obj.hi()); // hi bob
console.log(hi());
// 报错
// return 'hi ' + this.username;
//                    ^
// TypeError: Cannot read property 'username' of undefined
```

3、构造函数调用将一个全新的对象作为this变量的值，并隐式返回这个新对象作为调用结果。构造函数主要职责是初始化该新对象。



## 高阶函数

1、高阶函数无非是那些将函数作为参数或返回值的函数

2、使用数组`map`方法，可以完全消除循环，仅使用一个局部函数就可以实现对元素的逐个转换

``` js
var names = ['fred','Wilma','Pebbles'];
var upper = names.map(function(name) {
    return name.toUpperCase();
});

console.log(upper); // [ 'FRED', 'WILMA', 'PEBBLES' ]
```

3、使用`call、apply`去调用函数时，如果函数中没有引用this变量，则可以简单的将第一个参数设为`null`

``` js
function ave() {
    var sum = 0,
        len = 0;
    for (var i = 0, len = arguments.length; i < len; i++) {
        sum += arguments[i];
    }
    return sum / len;
}

console.log(ave.call(null, 1, 2, 3)); // 2
console.log(ave.apply(null, [1, 2, 3])); // 2
```



## 永远不要修改arguments对象

``` js
function callMethod(obj, method) {
    var shift = [].shift;
    shift.call(arguments);
    shift.call(arguments);
    return obj[method].apply(obj, arguments);
}

var obj = {
    add: function(x, y) {
        return x+y;
    }
};

callMethod(obj,'add',1,2);
// 报错：TypeError: Cannot read property 'apply' of undefined
```

> 该函数出错的原因是`arguments`对象并不是函数参数的副本。所有的命名参数都是`arguments`对象中对应索引的别名。
>
> 即使通过shift方法移除`arguments`对象中的元素之后，`obj`仍然是`arguments[0]`的别名，`method`仍然是`arguments[1]`的别名。
>
> 这意味着我们似乎是在提取`obj['add']`，但实际上是在提取`17[25]`。引擎将`17`转换为Number对象并提取其`25`属性（该属性不存在），结果产生`undefined`，然后试图提取`undefined`的`apply`属性并将其作为方法来调用，因此报错。

严格模式下，函数参数不支持对其`arguments`对象取别名。

``` js
function strict(x) {
    'use strict';
    arguments[0] = 'modified';
    return x === arguments[0];
}

function nonstrict(x) {
    arguments[0] = 'modified';
    return x === arguments[0];
}

console.log(strict('unmodified')); // false
console.log(nonstrict('unmodified')); // true
```

> 也就是，在严格模式下，arguments对象只是参数的一个副本，之后的修改不影响参数值。
>
> 例外情况：传入的是对象（或数组）的引用，函数内`arguments`修改的是**对象的属性**，则参数会同时被修改【这点不管在严格模式还是非严格模式都一样】

```js
function strict(obj) {
    'use strict';
    arguments[0].name = 'hehe';
    console.log(obj.name); // 输出'hehe'
}

strict({name: 'haha'});
```

因此，永远不要修改arguments对象是更为安全的。通过一开始复制参数中的元素到一个真正的数组的方式，很容易避免修改arguments对象，如下：

```js
var args = [].slice.call(arguments);
```



## bind方法

1、**bind()**方法创建的是一个新的函数

```js
var siteDomain = 'baidu';
var paths = ['image', 'music', 'cloud'];

function simpleURL(protocol, domain, path) {
    return protocol + '://' + domain + '/' + path;
}

var urls = paths.map(function(path) {
    return simpleURL('http', siteDomain, path);
});

var urls2 = paths.map(simpleURL.bind(null, 'http', siteDomain));

console.log(simpleURL === simpleURL.bind(null)); // false

console.log(urls);
console.log(urls2);
// 两个输出的结果都是：
// [ 'http://baidu/image',
//   'http://baidu/music',
//   'http://baidu/cloud' ]
```

> 传入null或undefined作为接收者的参数来实现函数柯里化

