---
title: JS深浅复制
date: 2018-03-12 23:28:36
tags:
- 复制
- js
categories:
- js
- js技巧
---

## 深复制

* 含义：复制后的对象和原来的对象一模一样但没有联系了，对复制后的对象的操作不影响原对象


<!-- more -->


### 方法实现

* 深复制数组或对象

```js
function clone(v) {
    if (typeof v !== 'object') {
        return v
    }

    let obj = Array.isArray(v) ? [] : {}
    for (let key in v) {
        // 加入hasOwnProperty则不深复制原型链中的方法
        if (v.hasOwnProperty(key)) {
            obj[key] = clone(v[key])
        }
    }

    return obj
}
```

```js
var obj = {
    hobits: ['a', 'b', 'c']
};

var cp = clone(obj);
console.log(cp); // { hobits: [ 'a', 'b', 'c' ] }

cp.hobits.push('d');
console.log(cp); // { hobits: [ 'a', 'b', 'c', 'd' ] }
console.log(obj); // { hobits: [ 'a', 'b', 'c' ] }
```



### JQuery.extend()

* 深复制对象：jQuery.extend([deep], target, object1, [objectN])

```js
var obj = {
    hobits: ['a', 'b', 'c']
};

let cp = {};
$.extend(true, cp, obj);
console.log(cp); // { hobits: [ 'a', 'b', 'c' ] }

cp.hobits.push('d');
console.log(cp); // { hobits: [ 'a', 'b', 'c', 'd' ] }
console.log(obj); // { hobits: [ 'a', 'b', 'c' ] }
```



### JSON.parse

对象深复制（**不能复制方法**）：JSON.parse(JSON.stringify(oldObj);

```js
var obj = {
    hobits: ['a', 'b', 'c']
};

var cp = JSON.parse(JSON.stringify(obj));
console.log(cp); // { hobits: [ 'a', 'b', 'c' ] }

cp.hobits.push('d');
console.log(cp); // { hobits: [ 'a', 'b', 'c', 'd' ] }
console.log(obj); // { hobits: [ 'a', 'b', 'c' ] }
```



## 浅复制

* 含义：属性是object类型的，复制后的属性依然是原来对应属性的引用，其他普通属性则复制



### 数组浅复制

slice方法，concat方法，ES6的数组解构方法，其实跟for循环一个一个复制一样，for循环这里不展开了

```js
var oldArr = [['a', 'b'], 'c'];

var cp = oldArr.concat();
// 或：
// var cp = oldArr.slice();
// 或：
// var cp = [...oldArr];

console.log(cp); // [ [ 'a', 'b' ], 'c' ]

cp[0].push('d');
console.log(cp); // [ [ 'a', 'b', 'd' ], 'c' ]
console.log(oldArr); // [ [ 'a', 'b', 'd' ], 'c' ]
```



### 对象浅复制

* ES6对象解构

```js
var obj = {
    name: 'obj',
    hobits: ['a', 'b', 'c']
};

var cp = {...obj};
console.log(cp); // { name: 'obj', hobits: [ 'a', 'b', 'c' ] }

cp.name = 'cp';
cp.hobits.push('d');

console.log(cp); // { name: 'cp', hobits: [ 'a', 'b', 'c', 'd' ] }
console.log(obj); // { name: 'obj', hobits: [ 'a', 'b', 'c', 'd' ] }
```



- Object.assign()

```js
var obj = {
    name: 'obj',
    hobits: ['a', 'b', 'c']
};

var cp = Object.assign({}, obj);
console.log(cp); // { name: 'obj', hobits: [ 'a', 'b', 'c' ] }

cp.name = 'cp';
cp.hobits.push('d');

console.log(cp); // { name: 'cp', hobits: [ 'a', 'b', 'c', 'd' ] }
console.log(obj); // { name: 'obj', hobits: [ 'a', 'b', 'c', 'd' ] }
```



* Object.getOwnPropertyDescriptors)

该方法用来获取一个对象的所有自身属性的描述符
```js
function copy(o) {
    return Object.create(
        Object.getPrototypeOf(o),
        Object.getOwnPropertyDescriptors(o)
    );
}
```

```js
var obj = {
    name: 'obj',
    hobits: ['a', 'b', 'c']
};

var cp = copy(obj);
console.log(cp); // { name: 'obj', hobits: [ 'a', 'b', 'c' ] }

cp.name = 'cp';
cp.hobits.push('d');

console.log(cp); // { name: 'cp', hobits: [ 'a', 'b', 'c', 'd' ] }
console.log(obj); // { name: 'obj', hobits: [ 'a', 'b', 'c', 'd' ] }
```

> 参考链接：http://javascript.ruanyifeng.com/oop/object.html#toc3
