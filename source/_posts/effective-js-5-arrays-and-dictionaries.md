---
title: EffectiveJS笔记05之数组和字典
date: 2017-11-10 15:47:55
tags:
- js
- EffectiveJavaScript
- 数组
- 字典
categories:
- js
- EffectiveJS
---

## 使用数组而不要使用字典来存储有序集合

1、`for...in`循环除了枚举出对象“自身”的属性外，还会枚举出继承过来的属性

<!-- more -->

```js
function Animal() {
    this.type = 'animal';
}
Animal.prototype.eat = function() {
    console.log('eat ing...');
};

var dog = new Animal();
dog.age = 1;

for (var attr in dog) {
    console.log(attr);
}
// 输出：
// type
// age
// eat
```

>  `for...in` 循环的顺序不固定，如果需要依赖一个数据结构中的条目顺序，请使用数组



2、使用`hasOwnProperty`方法可以排除继承过来的属性

```js
// 其余代码同上
for (var attr in dog) {
    if(dog.hasOwnProperty(attr)) {
        console.log(attr);
    }
}
// 输出：
// type
// age
```



3、使用`Object.defineProperty`方法可以定义一个对象的属性并指定该属性的元数据。

可以设置其枚举属性为`false`使其在`for...in`循环中不可见

```js
function Animal() {
    this.type = 'animal';
}

Object.defineProperty(Animal.prototype, 'eat', {
    value: function() {
        console.log('eat ing...');
    },
    writable: true,
    enumerable: false, // 设置为不可枚举
    configurable: true
});

var dog = new Animal();
dog.age = 1;

for (var attr in dog) {
    console.log(attr);
}
// 输出：
// type
// age
```



## 避免在枚举期间修改对象

1、如果被枚举的对象在枚举期间添加了新的属性，那么在枚举期间并不能保证新添加的属性能够被访问。

2、当迭代一个对象时，如果该对象的内容可能在循环期间被改变，应该使用while循环或经典的for循环来代替`for...in`循环



## 数组迭代要优先使用for循环而不是for...in循环

使用`for...in`循环，对象的key始终是字符串

```js
var scores = [1,2,3,4,5];
var total = 0;
for(var score in scores) {
    total += score;
}
console.log(total); // 001234
```



## 迭代方法优于循环

1、通过现有的数组建立一个新的数组。可以使用`Array.prototype.map`方法

```js
var trimmedArr = oldArr.map(function(s) {
    return s.trim();
});
```

2、计算一个新的数组，该数组只包含现有数组的一些元素。可以使用`Array.prototype.filter`。如果元素应该存在于新数组则返回真值，如果元素应该被剔除则返回假值。

```js
var oldArr = [1, 10, 100, 1000, 10000];

var filteredArr = oldArr.filter(function(item) {
    return item >= 100 && item <= 10000;
});

console.log(filteredArr); // [ 100, 1000, 10000 ]
```

3、循环只有一点优于迭代函数，那就是循环有控制流操作，如`break`和`continue`。举例来说，使用`forEach`方法来实现`takeWhile`函数将是一个尴尬的尝试。

```js
// 使用for循环
function takeWhile(a, pred) {
    var result = [];
    for (var i = 0, n = a.length; i < n; i++) {
        if (!pred(a[i], i)) {
            break;
        }
        result[i] = a[i];
    }
    return result;
}

var prefix = takeWhile([1, 2, 10, 3, 30], function(n) {
    return n < 10;
});
console.log(prefix); // [ 1, 2 ]
```

```js
// 使用forEach
function takeWhile(a, pred) {
    var result = [];
    a.forEach(function(x, i) {
        if(!pred(x)) {
            return; // 此处无法使用break！
        }
        result[i] = x;
    });
    return result;
}

var prefix = takeWhile([1, 2, 10, 3, 30], function(n) {
    return n < 10;
});
console.log(prefix); // [ 1, 2, <1 empty item>, 3 ]
```

> return只是跳过了那一次循环，并没有终止掉循环。要想在`forEach`中终止循环，可以使用一个内部异常来提前终止该循环

```js
function takeWhile(a, pred) {
    var result = [];
    var earlyExit = {};
    try {
        a.forEach(function(x, i) {
            if (!pred(x)) {
                throw earlyExit;
            }
            result[i] = x;
        });
    } catch (e) {
        if (e !== earlyExit) { // only catch earlyExit
            throw e;
        }
    }
    return result;
}

var prefix = takeWhile([1, 2, 10, 3, 30], function(n) {
    return n < 10;
});
console.log(prefix); // [ 1, 2 ]
```

4、ES5的数组方法`some`和`every`可以用于提前终止循环

* `some`方法返回一个布尔值表示其回调对数组的**任何一个元素**是否返回一个**真值**
* `every`方法返回一个布尔值表示其回调是否对**所有元素**返回了一个**真值**

```js
[1, 10, 100].some(function(x) { return x > 5; }); // true
[1, 10, 100].some(function(x) { return x < 0; }); // false

[1, 2, 3, 4, 5].every(function(x) { return x > 0; }); // true
[1, 2, 3, 4, 5].every(function(x) { return x < 3; }); // false
```

> 这两个方法都是短路循环。如果对some方法的回调一旦产生了一个真值，则some方法会直接返回，不会执行其余的元素。类似的，every方法的回调一旦产生了假值，则会立即返回

可以使用`every`实现`takeWhile`函数

```js
function takeWhile(a, pred) {
    var result = [];
    a.every(function(x, i) {
        if (!pred(x)) {
            return false;
        }
        result[i] = x;
        return true; // continue
    });
    return result;
}

var prefix = takeWhile([1, 2, 10, 3, 30], function(n) {
    return n < 10;
});
console.log(prefix); // [ 1, 2 ]
```



## 在类数组对象上复用通用的数组方法

1、DOM的NodeList类是另一个类数组对象的实例

2、数组对象的基本契约总共有两个简单的规则，满足后能使一个对象“看起来像数组”

* 具有一个范围在0到2^32-1的整型length属性
* length属性大于该对象的最大索引。索引是一个范围在0到2^32-2的整数，它的字符串表示的是该对象的一个key

```js
var arrayLike = {
    0: 'a',
    1: 'b',
    2: 'c',
    length: 3
};

var result = Array.prototype.map.call(arrayLike, function(s) {
    return s.toUpperCase();
});

console.log(result); // [ 'A', 'B', 'C' ]
```

3、字符串也表现为不可变的数组，因为它们是可索引的，并且其长度也可以通过length属性获取。因此，Array.prototype中的方法操作字符串时并不会修改原始数组。

```js
var result = Array.prototype.map.call('abc', function(s) {
    return s.toUpperCase();
});

console.log(result); // [ 'A', 'B', 'C' ]
```



## 数组字面量优于数组构造函数

1、`['hello']`和`new Array("hello")`的行为相同，但`[17]`和`new Array[17]`的行为完全不同

2、如果使用单个数字参数来调用Array构造函数，它试图创建一个没有元素的数组，但其长度属性为给定的参数