---
title: ES6-Iterator-Generator
date: 2018-08-28 21:34:49
tags:
- es6
- js
- iterator
- generator
categories:
- js
- es6
---

## 说明

Iterator & Generator

<!-- more -->

## Iterator

可以通过 Symbol.iterator 来访问对象默认的迭代器

```js
let arr = [1,2,3]
let iterator = arr[Symbol.iterator]()

console.log(iterator.next()) // { value: 1, done: false }
console.log(iterator.next()) // { value: 2, done: false }
console.log(iterator.next()) // { value: 3, done: false }
console.log(iterator.next()) // { value: undefined, done: true }
```



## for-of遍历

遍历的对象必须要有迭代器，**数组，字符串，set，map**已经内置了。其他可手动添加。遍历**字符串**能够正确访问**双字节**字符

### 创建可迭代对象

```js
let collection = {
    items: [],
    *[Symbol.iterator]() {
        for(let item of this.items) {
            yield item
        }
    }
}

collection.items.push(1)
collection.items.push(2)
collection.items.push(3)

for(let x of collection) {
    console.log(x)
}

// 1
// 2
// 3
```

```js
var myObject = {
    a: 2,
    b: 3
};

Object.defineProperty(myObject, Symbol.iterator, {
    enumerable: false,
    writable: false,
    configurable: true,
    value: function() {
        var o = this
        var idx = 0
        var ks = Object.keys(o)
        return {
            next: function() {
                return {
                    value: o[ks[idx++]],
                    done: (idx > ks.length)
                }
            }
        }
    }
})

// 用 for..of 遍历 myObject
for (var v of myObject) {
    console.log(v);
}
// 2
// 3
```

> 或者

```js
var myObject = {
    a: 2,
    b: 3,
    [Symbol.iterator]() {
        var o = this
        var idx = 0
        var ks = Object.keys(o);
        return {
            next: function() {
                return {
                    value: o[ks[idx++]],
                    done: (idx > ks.length)
                }
            }
        }
    }

}
```



## 内建迭代器

### 集合对象迭代器

3种类型的集合对象：数组，Map集合和Set集合

- entries() : 返回一个迭代器，其值为多个键值对。
  - 数组：第一个元素是数字类型的索引，第二个是索引对应的值
  - Set 集合，第一个元素与第二个元素都是值
  - Map集合，第一个元素为键名，第二个是键名对应的值
- values() : 返回一个迭代器，其值为集合的所有值
  - Set 集合：keys() 和 values() 返回的是相同的迭代器
- keys() : 返回一个迭代器，其值为集合中的所有键名

**在 for-of 循环中，如果没有显示制定则使用默认的迭代器，数组和 Set 集合的默认迭代器是 values() 方法，Map 集合的默认迭代器是 entries() 方法**

### NodeList 迭代器

DOM 定义的 NodeList 类型（定义在 HTML 标准中）也拥有默认迭代器，其行为和数组的默认迭代器一致



## Generator

- 判断Generator函数及生成器对象

```js
// 检测生成器函数
function isGeneratorFunction(fn) {
    const genFn = (function*() {}).constructor;
    return fn instanceof genFn;
}


// 检测生成器实例对象
function isGenerator(obj) {
    return obj.toString ? obj.toString() === '[object Generator]' : false;
}

function* genFn() {
    let a = 1;
    while (true) {
        yield a = a*2;
    }
}

const gen = genFn();
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next());  // { value: 4, done: false }
console.log(gen.next());  // { value: 8, done: false }

console.log(isGeneratorFunction(genFn)); // true
console.log(isGenerator(gen)); // true
console.log(isGenerator({})); // false
```



- 在一个**普通函数**中使用yield表达式，结果产生一个句法错误

```js
var arr = [1, [[2, 3], 4], [5, 6]];

var flat = function*(a) {
    a.forEach(function(item) {
        if (typeof item !== 'number') {
            yield* flat(item); // SyntaxError
        } else {
            yield item;
        }
    });
};
```

> 上面代码也会产生句法错误，因为forEach方法的参数是一个普通函数，但是在里面使用了yield表达式。一种修改方法是改用for循环。



### 给迭代器传递参数

yield表达式本身没有返回值，或者说总是返回 undefined 。next 方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值。特例是**第一次调用 next() 方法时无论传入什么参数都会被丢弃**

```js
function* f() {
    for (var i = 0; true; i++) {
        var reset = yield i;
        console.log(i, reset);
        if (reset) { i = -1; }
    }
}

var g = f();

console.log(g.next()); // { value: 0, done: false }
console.log(g.next()); // { value: 1, done: false }
console.log(g.next()); // { value: 2, done: false }
console.log(g.next(true)); // { value: 0, done: false }

// { value: 0, done: false }
// 0 undefined
// { value: 1, done: false }
// 1 undefined
// { value: 2, done: false }
// 2 true
// { value: 0, done: false }
```



### 在迭代器中抛出错误

```js
function *createIterator() {
    let first = yield 1
    let second = yield first + 2
    yield second + 3 // 不会被执行
}

let iterator = createIterator()
console.log(iterator.next()) // { value: 1, done: false }
console.log(iterator.next(4)) // { value: 6, done: false }
console.log(iterator.throw(new Error('Boom'))) // 抛出错误
console.log(iterator.next())
```

捕获错误，调用 throw()  方法也会像调用 next() 方法一样返回一个结果对象。由于在生成器内部捕获了这个错误，因而会继续执行下一条 yield 语句，最终返回数值 9

```js
function *createIterator() {
    let first = yield 1
    let second
    try {
        second = yield first + 2
    }catch(ex) {
        second = first + 2
    }
    yield second + 3
}

let iterator = createIterator()
console.log(iterator.next()) // { value: 1, done: false }
console.log(iterator.next(4)) // { value: 6, done: false }
console.log(iterator.throw(new Error('Boom'))) // { value: 9, done: false }
console.log(iterator.next()) // { value: undefined, done: true }
```



### 生成器返回语句

在生成器中，return 表示所有操作已经完成，属性 done 被设置为 true，如果同时提供了相应的值，则属性 value 会被设置为这个值

```js
function *createIterator() {
    yield 1
    return 2
    yield 3
}

let iterator = createIterator()
console.log(iterator.next()) // { value: 1, done: false }
console.log(iterator.next()) // { value: 2, done: true }
console.log(iterator.next()) // { value: undefined, done: true }
```



### 委托生成器

```js
function *createNumberIterator() {
    yield 1
    return 2
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator()
    yield result
    yield 3
}

let iterator = createCombinedIterator()
console.log(iterator.next()) // { value: 1, done: false }
console.log(iterator.next()) // { value: 2, done: false }
console.log(iterator.next()) // { value: 3, done: false }
console.log(iterator.next()) // { value: undefined, done: true }
```



### 异步任务执行器

```js
function run(taskDef) {
    let task = taskDef()
    let result = task.next()

    function step() {
        if(!result.done) {
            if(typeof result.value === 'function') {
                result.value(function(err, data) {
                    if(err) {
                        result = task.throw(err)
                        return
                    }
                    result = task.next(data)
                    step()
                })
            } else {
                result = task.next(result.value)
                step()
            }
        }
    }

    // 开始迭代执行
    step()
}

function async1() {
    return function(cb) {
        setTimeout(() => {
            cb(null, 'done1')
        }, 1000)
    }
}

function async2() {
    return function(cb) {
        setTimeout(() => {
            cb(null, 'done2')
        }, 1000)
    }
}

run(function*() {
    let content1 = yield async1()
    console.log('content1', content1) // content1 done1

    let content2 = yield async2()
    console.log('content2', content2) // content2 done2
})
```

 

> 参考链接1：http://blog.csdn.net/kaelyn_X
>
> 参考链接2：http://es6.ruanyifeng.com/
>
> 参考书籍：《UNDERESTANDING ECMASCRIPT 6》