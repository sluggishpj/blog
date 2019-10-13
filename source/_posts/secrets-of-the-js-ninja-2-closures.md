---
title: JavaScript忍者秘籍笔记02之闭包
date: 2017-10-21 00:46:23
tags:
- js
- 闭包
- js忍者秘籍
categories:
- js
- js忍者秘籍
---

## 闭包的作用域
<!-- more -->
``` js
var later;

function outerFunction() {
    var innerValue = true;

    function innerFunction(paramValue) {
        console.log(innerValue); // true
        console.log(paramValue); // true
        console.log(tooLate); // true
    }
    later = innerFunction;
}

outerFunction();

var tooLate = true; // 这个声明必须在later调用前，否则later函数中的tooLate为undefined
later(true);
```

## 绑定函数上下文
``` js
var button = {
    clicked: false,
    click: function() {
        this.clicked = true;
        console.log(this);
        console.log(button.clicked);
    }
};

var elem = document.getElementById('test');
elem.addEventListener('click', button.click, false);
```
> 在本例中，浏览器的事件处理系统认为函数调用的上下文(this)是**事件的目标元素**，所以才导致其上下文是`<button>元素`，而不是`button对象`

``` js
// 解决方法1，使用匿名函数
elem.addEventListener('click', function() {
    button.click();
}, false);

// 解决方法2，使用bind方法
elem.addEventListener('click',button.click.bind(button),false);
```
> setTimeout和setInterval也是同样的解决方法，区别是这两个绑定的上下文(this)是**window对象**

> 更多请参考：[Function.prototype.bind() - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)


## 偏应用函数
### 函数科里化
``` js
Function.prototype.curry = function() {
    var fn = this,
        args = Array.prototype.slice.call(arguments); // 将类数组对象arguments转换为数组
    return function() {
        var allArgs = Array.prototype.slice.call(arguments).concat(args);
        return fn.apply(this, allArgs);
    }
}
function add(n1, n2) {
    return n1 + n2;
}

console.log(add.curry(1)(4)); // 5
```

### “分部”函数
``` js
Function.prototype.partial = function() {
    var fn = this,
        args = Array.prototype.slice.call(arguments);
    return function() {
        var j = 0;
        for (var i = 0; i < args.length && j < arguments.length; i++) {
            if (args[i] === undefined) {
                args[i] = arguments[j++];
            }
        }
        return fn.apply(this, args);
    };
};

var delay = setTimeout.partial(undefined, 1000);
delay(function() {
    console.log('delay 1 s'); // 1秒后输出
});
```


## 使用闭包实现缓存记忆功能
``` js
Function.prototype.memoized = function(key) {
    this._values = this._values || {};
    console.log(this);
    return this._values[key] ? this._values[key] : this._values[key] = this.apply(this, arguments);
}

Function.prototype.memoize = function() {
    var fn = this;
    return function() {
        return fn.memoized.apply(fn, arguments);
    }
}

function isPrime(num) {
    var prime = num != 1; // != 的优先级高于 =
    for (var i = 2; i < num; i++) {
        if (num % i == 0) {
            prime = false;
            break;
        }
    }
    return prime;
}

var isPrimeMemo = isPrime.memoize();

console.log(isPrimeMemo(4)); // false
console.log(isPrimeMemo(7)); // true
console.log(isPrime._values); // { '4': false, '7': true }
```

## 即时函数
> (...)()中，第一组圆括号仅仅是用于划定表达式的范围，而第二个圆括号则是一个操作符。eg，将函数引用通过圆括号括起来是合法的：

``` js
var someFunction = function() {... };
result = (someFunction)();
```

### 通过参数限制作用域内的名称
``` html
    <img src="../images/ninja-with-pole.png">
    <script type="text/javascript">
    $ = function() {alert('not jQuery!');}; // 定义一个$表示其他内容，而不是jQuery
    (function($) {
        $('img').on('click', function(event) {
            $(event.target).addClass('clickedOn');
        });
    })(jQuery); // 在调用即时函数时，将jQuery作为参数传递进去，就会将jQuery绑定到$参数上了
    </script>
```
