---
title: EffectiveJS笔记02之变量作用域
date: 2017-11-04 21:17:40
tags:
- js
- EffectiveJavaScript
- 作用域
categories:
- js
- EffectiveJS
---

## 理解变量声明提升

1、变量的作用域是整个函数，但仅在var语句出现的位置进行赋值

<!-- more -->

![变量声明提升](https://s2.ax1x.com/2019/10/13/ux2UHO.png)

2、同一函数中多次声明相同变量是合法的。这在写多个循环时会经常出现

``` js
function trimSections(header, body, footer) {
    for (var i = 0, n = header.length; i < n; i++) {
        header[i] = header[i].trim();
    }
    for (var i = 0, n = body.length; i < n; i++) {
        body[i] = body[i].trim();
    }
    for (var i = 0, n = footer.length; i < n; i++) {
        footer[i] = footer[i].trim();
    }
}
```

> `trimSections`函数好像声明了6个局部变量（3个变量i，3个变量n），但经过变量声明提升后其实只声明了2个。

3、`try...catch`语句将捕获的异常绑定到一个变量，该变量的作用域只是`catch`语句块

``` js
function test() {
    var x = 'var',
        result = [];
    result.push(x);
    try {
        throw 'exception';
    } catch (x) {
        x = 'catch';
    }
    result.push(x);
    return result;
}
console.log(test()); // [ 'var', 'var' ]
```

4、闭包存储的是其外部变量的引用而不是值

``` js
function wrapElements(a) {
    var result = [],
        i, n;
    for (i = 0, n = a.length; i < n; i++) {
        result[i] = function() {
            return a[i];
        };
    }
    return result;
}
var wrapped = wrapElements([10, 20, 30, 40, 50]);

var f = wrapped[0];
console.log(f()); // undefined
```

为了让`result[i]`执行后返回的是`a[i]`的值。解决办法是通过创建一个嵌套函数并立即调用它来强制创建一个局部作用域。

``` js
function wrapElements(a) {
    var result = [],
        i, n;
    for (i = 0, n = a.length; i < n; i++) {
        (function(j) {
            result[i] = function() {
                return a[j]; 
            };
        })(i);
    }
    return result;
}
```



## 担心命名函数表达式笨拙的作用域

1、匿名和命名函数表达式的官方区别在于后者会绑定到与其函数名相同的变量上，该变量将作为该函数内的一个局部变量。这可以用来写递归函数表达式

``` js
var f = function find(tree, key) {
    if (!tree) {
        return null;
    }
    if (tree.key === key) {
        return tree.value;
    }
    return find(tree.left, key) ||
        find(tree.right, key);
};

find('tree','foo'); // ReferenceError: find is not defined
```

> 注意，变量find的作用域只在其自身函数中。命名函数表达式不能通过其内部的函数名在外部被引用。
>
> 使用命名函数表达式进行递归似乎没有必要，因为使用外部作用域的函数名也可以达到同样的效果。

``` js
var f = function(tree, key) {
    if (!tree) {
        return null;
    }
    if (tree.key === key) {
        return tree.value;
    }
    return f(tree.left, key) ||
           f(tree.right, key);
};
```



## 担心局部块函数声明笨拙的作用域

``` js
function test(x) {
    function f() {
        return 'local';
    }
    var result = [];
    if (x) {
        result.push(f());
    }
    result.push(f());
    return result;
}
console.log(test(true)); // ['local', 'local']
console.log(test(false)); // ['local']
```

> JS没有块级作用域，所以内部函数f的作用域是整个test函数

``` js
function f() {
    return 'global';
}

function test(x) {
    var result = [];
    if (x) {
        function f() {
            return 'local';
        }
        result.push(f());
    }
    result.push(f());
    return result;
}
test(false);

// 报错
//13行 result.push(f());
//TypeError: f is not a function
```

> 避免将函数置于局部块或子语句中。
>
> 如果需要有条件的选择函数，使用var声明和函数表达式来实现

``` js
function f() {
    return 'global'; }

function test(x) {
    var g = f,
        result = [];
    if (x) {
        g = function() {
            return 'local'; }
        result.push(g());
    }
    result.push(g());
    return result;
}

console.log(test(true)); // [ 'local', 'local' ]
console.log(test(false)); // [ 'global' ]
```

