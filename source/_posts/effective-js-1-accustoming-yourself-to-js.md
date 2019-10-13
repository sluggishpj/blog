---
title: EffectiveJS笔记01之让自己习惯JavaScript
date: 2017-11-02 23:10:09
tags:
- js
- EffectiveJavaScript
categories:
- js
- EffectiveJS
---



## 了解你使用的JavaScript版本

1、"use strict"指令只有在脚本或函数的顶部才能生效

2、合并多个不同模式下的js文件时。解决方案：将其自身包裹在立即的函数表达式（IIFE）中的方式连接多个文件。

<!-- more -->

``` js
// no strict-mode directive
(function() {
    // file1.js
    "use strict";

    function f() {
        // ...
    }
    // ...
})();

(function() {
    // file2.js
    // no strict-mode directive
    function f() {
        // ...
    }
    // ...
})();
```
> 要想构建代码以获得最大的兼容性，最简单的方法是在严格模式下编写代码，并显式地将代码内容包裹在本地启用了严格模式的函数中。



## 理解JavaScript的浮点数

1、JavaScript中的所有数字都是双精度浮点数，都是64位编码数字

2、所有位运算符的工作方式都是相同的。它们将操作数转换为整数，然后使用整数位模式进行运算，最后将结果转换为标准的JavaScript浮点数

3、浮点数运算只能产生近似的结果，四舍五入到最接近的可表示的实数。
``` js
var a = (0.1 + 0.2) + 0.3;
var b = 0.1 + (0.2 + 0.3);
console.log(a); // 0.6000000000000001
console.log(b); // 0.6
```
> 对于整数运算，不必担心舍入误差，只需担心计算范围在**-2^(53) ~ 2^(53)**



## 担心隐式的强制转换

1、算术运算符`-、*、/`和%在计算之前都会尝试将其转换为数字
``` js
'17'*3; // 51
'8'|'1'; // 9
```

2、标准的库函数也不是很可靠，因为它带有自己的隐式强制转换，会将参数转换为数字（isNaN函数的一个更精确的名称可能是coerecesToNaN）
``` js
var a = isNaN(NaN);
var b = isNaN('foo');
var c = isNaN(undefined);
var d = isNaN({});
var e = isNaN({ valueOf: 'foo' });

console.log(a); // true
console.log(b); // true
console.log(c); // true
console.log(d); // true
console.log(e); // true
```
更通用的方法如下：
``` js
function isReallyNaN(x) {
    return x !== x;
}
```

3、valueOf 与 toString
``` js
var a = 'j' + {
    toString: function() {
        return 's';
    }
};

var b = 2 * {
    valueOf: function() {
        return 3;
    }
};

var obj = {
    toString: function() {
        return '[object MyObject]';
    },
    valueOf: function() {
        return 66;
    }
}

console.log(a); // js
console.log(b); // 6
console.log('object' + obj); // object66
```
> valueOf 方法才是真正为那些代表数值的对象（如Number对象）而设计的。对于这些对象，toString和valueOf方法应返回一致的结果（相同数字的字符串或数值表示），因此，不管是对象的连接还是对象的相加，重载的运算符+总是一致的行为。最好避免使用valueOf方法，除非对象的确是一个数字的抽象，并且obj.toString() 能产生一个obj.valueOf()的字符串表示



## 原始类型优于封装对象

1、不同于原始的字符串，String对象是一个真正的对象

``` js
var s1 = new String('hello');
var s2 = 'world';

console.log(typeof s1); // object
console.log(typeof s2); // string
```

2、每个String对象都是一个单独的对象，其总是只等于自身

``` js
var s1 = new String('hello');
var s2 = new String('hello');

var s3 = 'hello';
var s4 = 'hello';

console.log(s1 == s2); // false
console.log(s3 === s4); // true

console.log(s1 == s3); // true
console.log(s1 === s3); // false
```

3、当对原始值提取属性和进行方法调用时，它表现得就像已经使用了对应的对象类型封装了该值一样，隐式封装

``` js
console.log('hello'.toUpperCase()); // HELLO

var str = 'hello';
str.child = 'world';
console.log(str.child); // undefined
```
> 不可以对原始值设置属性，无效。因为每次隐式封装都会产生一个新的String对象，更新第一个封装对象并不会造成持久的影响。



## 避免对混合类型使用==运算符

1、== 运算符的强制转换规则

| 参数类型1                      | 参数类型2                      | 强制转换                                     |
| :------------------------- | :------------------------- | :--------------------------------------- |
| null                       | undefined                  | 不转换，总是返回true                             |
| null或undefined             | 其他任何非null或undefined的类型     | 不转换，总是返回false                            |
| 原始类型：string、number或boolean | Date对象                     | 将Date对象转换为原始类型（优先尝试toString方法，再尝试valueOf方法） |
| 原始类型：string、number或boolean | 非Date对象                    | 将非Date对象转换为原始类型（优先尝试valueOf方法，再尝试toString方法） |
| 原始类型：string、number或boolean | 原始类型：string、number或boolean | 将原始类型转换为数字                               |

``` js
var date = new Date('2017/11/11');
console.log(date == '2017/11/11'); // false
console.log(date); // 2017-11-10T16:00:00.000Z
```
> 不等是因为Date对象呗转换成一种不同格式的字符串，而不是本例所采用的格式。

更好的策略是显式自定义应用程序转换的逻辑，并使用严格相等运算符

``` js
function toYMD(date) {
    var y = date.getYear() + 1900,
        m = date.getMonth() + 1,
        d = date.getDate();
    return y + '/' + (m < 10 ? '0' + m : m) + '/' + (d < 10 ? '0' + d : d);
}

var date = new Date('2017/11/11');
console.log(toYMD(date) === '2017/11/11'); // true
```



## 了解分号插入的局限

1、5个明确有问题的字符需要密切注意：`(、[、+、-`和`/`，每一个字符都能作为一个表达式运算符或者一条语句的前缀。如果下一行以这5个字符之一开始，那么本行不会自动插入分号。

``` js
var color = {}
var b = 3
var a = b
['r','g','b'].forEach(function(key) {
    color[key] = 255;
});

// 报错：
// 'r','g','b'].forEach(function(key) {
//              ^
// TypeError: Cannot read property 'forEach' of undefined
```

> 原因在于第4行以`[`开始，因此和第三行一起被解析为一条语句，等价于：

``` js
var a = b['r','g','b'].forEach(function(key) {
    color[key] = 255;
});
```

> 中括号中是逗号分隔表达式，从左到右一次执行，并返回最后一个表达式的值，对于本例，返回字符"b"

解决办法：在`(、[、+、-`和`/`字符的开始的前一行末尾加分号。最安全的是不要省略分号



## 视字符串为16位的代码单元序列

UFT-16的每个代码点编码需要一个或两个16位的代码单元，因此UTF-16是一种可变长度的编码。

JS已经采用16位的字符串元素。字符串属性和方法（如length、chatAt和charCodeAt）都是基于代码单元层级，而不是代码点层级。所以每当字符串包含辅助平面中的代码点时，JS将每个代码点表示为两个代码单元。

> Unicode标准从当时的2^16扩展到了超过2^20个代码点。新增加的范围被组织为17个大小为2^16个代码点。第一个子范围，称为*基本多文种平面（Basic Multilingual Plane，BMP）*，包含最初的2^16个代码点。余下的16个范围称为 *辅助平面（supplementary plane）*。


![一个包含来自辅助平面的代码点的JavaScript字符串](https://s2.ax1x.com/2019/10/13/uxgfpR.png)

``` js
var a = '𠮷a';
var b = '吉a';

console.log(a.length); // 3
console.log(b.length); // 2

console.log(a.charCodeAt(0)); // 55362
console.log(a.charCodeAt(1)); // 57271
console.log(a.charCodeAt(2)); // 97

console.log(a.charAt(1)); // 乱码
console.log(a.charAt(2)); // a
console.log(b.charAt(1)); // a

console.log(/^.$/.test('𠮷')); // false
console.log(/^..$/.test('𠮷')); // true
```

> 正则表达式也工作于代码单元层级，其单字符模式（“ . ”）匹配一个单一的代码单元。
>
> 上面代码中，汉字`𠮷`的码点是0x20BB7，UTF-16编码为0xD842 0xDFB7（十进制为55362 57271），需要4个字节储存。
>
> 对于这种4个字节的字符，JavaScript不能正确处理，字符串长度会误判为2，而且charAt方法无法读取整个字符，charCodeAt方法只能分别返回前两个字节和后两个字节的值。



## ES6 codePointAt方法

ES6提供了**codePointAt**方法，能够正确处理4个字节储存的字符，返回一个字符的码点。

``` js
let s = '𠮷a';

console.log(s.codePointAt(0)); // 134071
console.log(s.codePointAt(1)); // 57271
console.log(s.codePointAt(2)); // 97
```

> JavaScript将“𠮷a”视为三个字符，`codePointAt`方法在第一个字符上，正确地识别了“𠮷”，返回了它的十进制码点134071（即十六进制的20BB7）。在第二个字符（即“𠮷”的后两个字节）和第三个字符“a”上，`codePointAt`方法的结果与`charCodeAt`方法相同。
>
> `codePointAt`方法的参数，仍然是不正确的。比如，上面代码中，字符a在字符串s的正确位置序号应该是1，但是必须向codePointAt方法传入2。解决这个问题的一个办法是使用for...of循环，因为它会正确识别32位的UTF-16字符。

``` js
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}
// 20bb7
// 61
```

> 可以用`codePointAt`方法是测试一个字符由两个字节还是由四个字节组成

``` js
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit("𠮷")); // true
console.log(is32Bit("a")); // false
```



## ES6 fromCodePoint方法

ES6提供了**String.fromCodePoint**方法，可以识别大于`0xFFFF`的字符，弥补了`String.fromCharCode`方法的不足。在作用上，正好与`codePointAt`方法相反

```js
console.log(String.fromCodePoint(0x20BB7));
// "𠮷"
console.log(String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y');
// true
```

> 参考链接：http://es6.ruanyifeng.com/#docs/string



