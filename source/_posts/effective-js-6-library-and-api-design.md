---
title: EffectiveJS笔记06之库和API设计
date: 2017-11-16 21:43:17
tags:
- js
- EffectiveJavaScript
- API
categories:
- js
- EffectiveJS
---

## 将undefined看做“没有值”

在允许0、NaN或空字符串为有效参数的地方，绝不要通过真值测试来实现参数默认值

<!-- more -->

```js
function Point(x, y) {
    this.x = x || 20;
    this.y = y || 30;
}

var origin = new Point(0, 0);
console.log(origin.x); // 20
console.log(origin.y); // 20
```



## 接收关键字参数的选项对象

当传入函数的参数过多时，应考虑将其封装成对象



## 区分数组对象和类数组对象

测试一个对象是否是真数组，而不仅仅是类数组对象，Array.isArray方法比instanceof操作符更可靠

```js
let arr = [1, 2, 3];
console.log(Array.isArray(arr)); // true
```

在不支持ES5的环境中，可以使用标准的Object.prototype.toString方法测试一个对象是否为数组

```js
let arr = [1, 2, 3];

function isArray(x) {
    return Object.prototype.toString.call(x) === '[object Array]';
}
console.log(isArray(arr)); // true
```



## 当心丢弃错误

管理异步编程的一个比较困难的方面是对错误的处理，对于同步的代码，通过使用try语句块包装一段代码很容易一下子处理所有错误。

```js
try {
    f();
    g();
    h();
} catch (e) {
    // handle any error that occurred...
}
```

对于异步的代码，多步的处理器通常被分隔到事件队列的单词轮次中，因此，不可能将它们全部包装在一个try语句块中。事实上，异步的API甚至根本不可能抛出异常，因为，当一个异步的错误发生时，没有一个明显的执行上下文来抛出异常！相反，异步的API倾向于将错误表示为回调函数的特定参数，或使用一个附加的错误处理回调函数（有时被称为errbacks）。例如，异步下载文件的异步API可能会有一个额外的回调函数来处理错误。

```js
downloadAsync('http://example.com/file.txt', function(text) {
    console.log('file content' + text);
}, function(err) {
    console.log('error' + err);
});
```

