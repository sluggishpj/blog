---
title: JavaScript忍者秘籍笔记01之函数
date: 2017-10-13 20:37:16
tags: 
- js
- 函数
- js忍者秘籍
categories: 
- js
- js忍者秘籍
---

## 函数的独特之处
> 浏览器的事件轮询是单线程的

## 函数声明
<!-- more -->
### 函数name属性
``` js
function isNimble() {
    return true;
}
console.log(isNimble.name === 'isNimble'); // true


var canFly = function() {
    return true;
};
console.log(canFly.name === 'canFly'); 
// true ,此处跟原书内容不同，原书内容name为空，实际测试不为空


window.wieldsSword = function swingsSword() {
    return true;
};
console.log(window.wieldsSword.name === 'swingsSword'); // true


var object2 = {
  someMethod1: function() {}
};
object2.someMethod2 = function() {}

console.log(object2.someMethod1.name); // 'someMethod1'
console.log(object2.someMethod2.name); // ''


// 你不能改变一个函数的 name 属性的值, 因为该属性是只读的
var object = {
  // someMethod 属性指向一个匿名函数
  someMethod: function() { }
};
object.someMethod.name = "otherMethod";
console.log(object.someMethod.name); // someMethod


// Function.bind() 所创建的函数将会在函数的名称前加上"bound "
function foo() {}; 
console.log(foo.bind({}).name); // "bound foo"

console.log((new Function).name); // 'anonymous'
```
> 更多请参考：[Function.name - JavaScript|MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/name)

### 函数声明提升
```js
function outer() {
    console.log(typeof inner === 'function'); // true, 函数声明提升
    function inner() {}
    console.log(window.inner === undefined); // true, inner()不在全局作用域内
}
outer();
```

### 作用域和函数
``` js
function outer() {    
    console.log(c); // undefined。变量定义提升，但赋值没有提升
    console.log(d); // undefined

    if(false) {
        var c = 3;
    }

    var d = 4;
    var d; // 重复声明不影响原值
    console.log(c); // undefined，并没有被赋值
    console.log(d); // 4
}

outer();
```
>* 变量定义的作用域在整个函数内，但变量赋值的作用域开始于被赋值的地方，结束于所在函数的结尾，都与代码嵌套无关。
> * 命名函数的作用域是指声明该函数的整个函数范围，与代码嵌套无关（机制提升）
> * 对于作用域声明，全局上下文就像一个包含页面所有代码的超大型函数

## 函数调用
``` js
function foo() {
    return this;
}
console.log(foo() === global); // true

var bar = foo;
console.log(bar() === global); // true
```

> 使用bar变量调用该函数，该函数也是作为函数进行调用的，而且函数上下文依然是global(在浏览器是window);

## 小结
1. 函数的形参列表和实际参数的长度可以是不同的。
   未赋值的参数被设置为undefined。
   多出的参数是不会绑定到参数名称的。
2. 每个函数调用都会传入两个隐式参数：
   arguments，实际传入的参数集合
   this，作为函数上下文的对象引用
3. 可以用不同的方法进行函数调用，不同的调用机制决定了函数上下文的不同。
   作为普通函数进行调用时，其上下文的全局对象（global/window）。
   作为方法进行调用时，其上下文是拥有该方法的对象。
   作为构造器进行调用时，其上下文是一个新分配的对象。
   通过函数的apply()或call()方法进行调用时，上下文可以设置成任何值。


## 递归
### 内联命名函数
``` js
var bar  = function foo() {
    console.log(foo === bar); // true
}
bar();

console.log(typeof foo); // undefined
```
> * 尽管可以给内联函数进行命名，但这些名称只能在自身函数内部才是可见的。
> * `!!`构造是一个可以将任意JavaScript表达式转化为其等效布尔值的简单方式。eg，`!!"a" === true`和`!!0 === false`

## 可变长度的参数列表
### 函数的length属性
函数的length属性等于该函数声明时所要传入的<b>形参</b>数量
``` js
function bar(a) {}
function foo(a, b, c) {}

console.log(bar.length); // 1
console.log(foo.length); // 3
```
> * 通过其length属性，可以知道声明了多少命名参数
> * 通过`arguments.length`，可以知道在调用时传入了多少参数
> * 利用参数个数的差异创建重载函数

### 利用参数的个数进行函数重载
``` js
function addMethod(object, name, fn) {
    var old = object[name];
    object[name] = function() {
        if (fn.length == arguments.length) {
            return fn.apply(this, arguments);
        } else if (typeof old === 'function') {
            return old.apply(this, arguments);
        }
    }
}

var myobj = {};

addMethod(myobj, 'a', function() {
    console.log('0个参数');
});
addMethod(myobj, 'a', function(n) {
    console.log('1个参数', n);
});
addMethod(myobj, 'a', function(n1, n2) {
    console.log('两个参数', n1, n2);
});

myobj.a(); // 0个参数
myobj.a(1); // 1个参数 1
myobj.a(1, 2); // 两个参数 1 2
```

## 函数判断
``` js
function foo() {}

// 第一种，有跨浏览器问题
console.log(typeof foo); // function

// 更好的解决方法
function isFunction(fn) {
    return Object.prototype.toString.call(fn) === '[object Function]';
}
console.log(isFunction(foo)); // true
```
