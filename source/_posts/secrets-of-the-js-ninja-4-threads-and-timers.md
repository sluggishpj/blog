---
title: JavaScript忍者秘籍笔记04之线程和定时器
date: 2017-10-25 00:03:48
tags:
- js
- 定时器
- 线程
- js忍者秘籍
categories:
- js
- js忍者秘籍
---
## 定时器和线程的工作方式
### 执行线程中的定时器执行
浏览器不会对特定interval处理程序的多个实例进行排队
<!-- more -->
### timeout和interval之间的区别
``` js
setTimeout(function repeateMe() {
    console.log('执行setTimeout');
    setTimeout(repeateMe,1000);
},10);

setInterval(function() {
    console.log('执行setInterval');
},1000);
```
> 以上两段代码功能似乎是相同的，实际上不是。在setTimeout()代码中，要在前一个callback回调执行结束并延迟1秒（可能更多，但不会更少）以后，才能再次执行setTimeout()。而setInterval()则是每隔1秒就尝试执行callback回调，而不关注上一个callback是何时执行的。

### 定时器延迟的最小化及其可靠性
大多数情况下，是使用闭包给定时器或间隔定时器“传递”数据的，但也可以在声明这些定时器时传入额外的参数，例如，`setTimeout(call,interval,arg1,arg2,arg3)`会给callback回调传递arg1、arg2、arg3三个参数。
``` js
setTimeout(sayHello,1000,'world'); // hello, world
function sayHello(name) {
    console.log('hello,',name);
}
```

## 中央定时器控制
``` js
var timeControler = {
    timerID: 0,
    timers: [],
    add: function(fn) {
        this.timers.push(fn);
    },
    start: function() {
        if (this.timerID) return; // 已经有定时器在执行
        console.log('开始了');
        (function runNext() {
            if (timeControler.timers.length > 0) {
                for (var i = 0; i < timeControler.timers.length; i++) {
                    if (timeControler.timers[i]() === false) {
                        timeControler.timers.splice(i, 1);
                        i--;
                    }
                }
                timeControler.timerID = setTimeout(runNext, 10);
            }
        })();
    },
    stop: function() {
        clearTimeout(this.timerID);
        this.timerID = 0;
    }
};

var box = document.getElementById('box'),
    x = 0,
    y = 0;

timeControler.add(function() {
    box.style.left = x + 'px';
    if (++x > 200) return false;
});

timeControler.add(function() {
    box.style.top = y + 'px';
    y+=2;
    if (y > 120) return false;
});

timeControler.start();
```
> * 自己测试时，要把box的position设为absolute或relative
> * 以这种方式组织定时器，可以确保回调函数总是按照添加的顺序进行执行。而普通的定时器通常不会保证这种顺序。


## eval()方法进行求值
### 正常模式
该方法将在当前上下文内，执行所传入字符串形式的代码，执行返回结果则是最后一个表达式的执行结果。
``` js
console.log(eval('var bar = 5;')); // undefined
console.log(bar); // 5;
console.log(window.bar); // 5;

// eval方法返回传入字符串最后一个表达式的执行结果
console.log(eval('3+4,5+6')); // 11
```

> 正常模式下，eval语句的作用域，取决于它处于全局作用域，还是处于函数作用域。

### 严格模式
``` js
"use strict";
var x = 2;
console.log(eval('var x = 5; x')); // 5
console.log(eval('var bar = 6; bar')); // 6

console.log(x); // 2
console.log(bar); // Uncaught ReferenceError: bar is not defined
```
> 严格模式下，eval语句本身就是一个作用域，不再能够生成全局变量了，它所生成的变量只能用于eval内部。

> 参考链接：http://www.ruanyifeng.com/blog/2013/01/javascript_strict_mode.html
