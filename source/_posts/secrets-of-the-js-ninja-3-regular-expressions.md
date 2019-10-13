---
title: JavaScript忍者秘籍笔记03之正则表达式
date: 2017-10-22 23:57:49
tags:
- js
- 正则表达式
- js忍者秘籍
categories:
- js
- js忍者秘籍
---

## 正则表达式基础
### 正则表达式解释
#### 创建方法
<!-- more -->
正则表达式字面量或构造RegExp对象的实例
``` js
// 正则字面量
var reg = / pattern / flags ;
var test = /test/ig;
```
> 标志（flags），用以标明正则表达式的行为。正则表达式的匹配模式支持下列 3 个标志。
> * g ：表示全局（global）模式，即模式将被应用于所有字符串，而非在发现第一个匹配项时立即停止；
> * i ：表示不区分大小写（case-insensitive）模式，即在确定匹配项时忽略模式与字符串的大小写；
> * m ：表示多行（multiline）模式，即在到达一行文本末尾时还会继续查找下一行中是否存在与模式匹配的项。

``` js
// 构造RegExp对象实例
var reg = new RegExp(pattern,flags);
var test = new RegExp('test','ig');
```

### 术语与操作符
#### 匹配一类字符

* `[^abc]`: 除了'a','b','c'以外的任意字符
* `[a-m]`: 匹配'a'到'm'之间的所有字符


#### 转义
* 使用反斜杠\对任意字符进行转义
* 模式中使用的所有元字符都必须转义。正则表达式中的元字符包括：
`( [ { \ ^ $ | ) ? * + . ] }`

#### 匹配开始和结束
* 符号`^`如果作为正则表达式的**第一个字符**，则表示要从字符串的开头进行匹配。（注意，这只是`^`的一个重载，它还可以用于否定一个字符类集。
* 类似的，`$`表示该模式必须出现在字符串的结尾，同时使用`^`和`$`则表明指定的模式必须包含整个候选字符串。

#### 重复出现
* `?`：0次或1次
* `+`：1次或多次
* `*`：0次或多次
* `/a{4}/`表示匹配含有连续4个'a'的字符串。
* `/a{4, 10}/`表示匹配任何含有连续4个至10个'a'字符的字符串
* 次数区间的第二个值可选（但要保留逗号），其表示一个开区间。例如，`/a{4,}`表示匹配任何含有连续4个或多于4个'a'字符的字符串。

> * 默认情况下，它们是贪婪的：它们匹配所有的字符组合。在操作符后面加一个问号?字符（?字符的一个重载)，如`a+?`，可以让该表达式变为非贪婪的：进行最小限度的匹配。eg、
> * 对字符串'aaa'进行匹配，正则表达式`/a+/`匹配所有这三个字符，而非贪婪表达式`/a+?/`则只匹配一个a字符。

#### 预定义字符类
|预定义术语|匹配内容|预定义术语|匹配内容|
|:---:|:---:|:---:|:---:|
|\t|水平制表符|\b|空格|
|\v|垂直制表符|\f|换页符|
|\r|回车|\n|换行符|
|\cA:\cZ|控制符，例如：\cM匹配一个Control-M|\x0000:\xFFFF|十六进制的Unicode码|
|.|匹配除了换行（\n）之外的任意字符
|\d|匹配任意数字，等价于[0-9]|\D|匹配任意非数字，等价于[^0-9]|
|\w|匹配包含下划线的任意单词字符，等价于[A-Za-z0-9]|\W|匹配任意非单词字符|
|\s|匹配任何空白字符，包括空格，制表符，换页符等|\S|匹配任何非空白字符|
|\b|匹配单词边界|\B|匹配非单词边界|

#### 或操作符（OR）
* 用竖线（|）表示或者关系。eg、
* `/a|b/`匹配'a'或'b'字符。

#### 反向引用
* 这种术语表示法是在反斜杠后面加一个要引用的捕获数量，该数字从1开始，如\1，\2等
* `/^([dtn])a\1/`匹配任意一个以'd'或't'或'n'开头，且后面跟着一个a字符，且再后面跟着和**第一个捕获相同字符**的字符串
* 要匹配像'<strong>whatever</strong>'这样的简单元素，不使用反向引用，是无法做到的。因为无法知道关闭标签和开始标签是否匹配。`/<(\w+)>(.+)<\/\1>/`

## 编译正则表达式
1. 正则表达式编译发生在正则表达式第一次被创建的时候，而执行则是发生在我们使用编译过的正则表达式进行字符串匹配的时候。
2. 创建正则表达式的方式不一样，就算规则一样，也不相等。

``` js
var re1 = /test/i;
var re2 = new RegExp('test','i');
console.log(re1 == re2); // false
```

> 用构造器`（new RegExp(...))`创建正则表达式的使用，允许在运行时通过动态创建的字符串创建和编译一个正则表达式。

``` js
var regStr1 = '\s',
    regStr2 = '\\s';
console.log(regStr1); // s
console.log(regStr2); // \s

var reg1 = new RegExp(regStr1),
    reg2 = new RegExp(regStr2);

var testStr = 's';

console.log(reg1.test(testStr)); // true
console.log(reg2.test(testStr)); // false
```

> * 创建带有反斜杠的**字面量**正则表达式时，只需要提供一个反斜杠即可。但是，由于我们在**字符串**中写有反斜杠，所以需要双反斜杠进行转义。
> * 一旦正则表达式被编译了，就可以利用该表达式的test()方法进行验证是否匹配。


## 捕获匹配的片段
### 执行简单的捕获（match方法）
``` js
var filter = 'opacity:0.5;filter:alpha(opacity=50);'
console.log(filter.match(/opacity=([^)]+)/));
// 输出：
// [ 'opacity=50',
  // '50',
  // index: 25,
  // input: 'opacity:0.5;filter:alpha(opacity=50);' ]
```
> * match返回的数组的第一个索引的值总是改匹配的完整结果，然后是每个后续**捕获结果**
> * 注意：接下来的代码中输出的index和input被省略掉了

### 用全局表达式进行匹配
当使用全局正则表达式（添加一个g标记）时，返回的东西不一样。返回值依然是一个数组，但在全局正则表达式的情况下，匹配所有可能的匹配结果，而不仅仅是第一个匹配结果，返回的数组包含了全局匹配结果。每个匹配的捕获结果是不会返回的。
``` js
var html = "<div class='test'><b>Hello</b> <i>world!</i></div>";

// 局部正则匹配
var results = html.match(/<(\/?)(\w+)([^>]*?)>/);
console.log(results)
// 输出：
// [ '<div class=\'test\'>',
  // '',
  // 'div',
  // ' class=\'test\'',
// ]


// 全局正则匹配
var all = html.match(/<(\/?)(\w+)([^>]*?)>/g);
console.log(all);
// 输出：
// [ '<div class=\'test\'>', '<b>', '</b>', '<i>', '</i>', '</div>' ]
```
> * 可以看到，在进行局部正则匹配时，只有一个实例被匹配了，并且该匹配的捕获结果也返回了；但是在全局正则匹配时，返回的却是匹配结果的列表。
> * 可以使用正则表达式的`exec()`方法，在全局正则匹配时**恢复捕获功能**。该方法可以对一个正则表达式进行多次调用，每次调用都可以返回下一个匹配结果。例子如下：

``` js
var html = "<div class='test'><b>Hello</b> <i>world!</i></div>";
var tag = /<(\/?)(\w+)([^>]*?)>/g,
    match;
var num = 0;

while ((match = tag.exec(html)) !== null) {
    console.log(match);
    num++;
}
console.log(num == 6); // true

// 第一个输出：
// [ '<div class=\'test\'>',
//   '',
//   'div',
//   ' class=\'test\'',
// ]

// 第二个输出：
// [ '<b>',
//   '',
//   'b',
//   '',
// ]
//   接下来的4个输出同理
```
> 在本例中，反复调用了`exec()`方法，该方法保存了上一次调用的状态，这样每个后续调用就可以继续下去了，直到全局匹配。每个调用返回的都是下一个匹配及其**捕获内容**

### 捕获的引用
两种方法可以引用捕获到的匹配结果：一是**自身匹配**，二是**替换字符串**。
#### 使用反向引用匹配HTML标签内容
``` js
var html = "<b class='hello'>Hello</b> <i>world!</i>";
var pattern = /<(\w+)([^>]*)>(.*?)<\/\1>/g;

var result = pattern.exec(html);

console.log(result);
// 输出：
// [ '<b class=\'hello\'>Hello</b>',
//   'b',
//   ' class=\'hello\'',
//   'Hello',
// ]


result = pattern.exec(html);
console.log(result);
// 输出：
// [ '<i>world!</i>',
//   'i',
//   '',
//   'world!',
// ]
```

#### replace方法获得捕获的引用
使用`$1`、`$2`、`$3`语法表示每个捕获的数字。eg、
``` js
console.log("fontFamily".replace(/([A-Z])/g, "-$1").toLowerCase()); // font-family
```
> 由于捕获和表达式分组都使用了小括号，正则表达式无法区分，所以会将小括号既视为分组，又视为捕获。

### 没有捕获的分组
1. 当正则表达式有一部分是用括号进行分组时，它具有双重责任，同时也创建捕获（capture）。
2. 要让一组括号不进行结果捕获，正则表达式的语法允许我们在开始括号后加一个`?:`标记。这就是所谓的*被动子表达式（passive subexpression）*。

``` js
var pattern1 = /((ninja-)+)sword/;
var pattern2 = /((?:ninja-)+)sword/;

var res1 = "ninja-ninja-sword".match(pattern1);
var res2 = "ninja-ninja-sword".match(pattern2);

console.log(res1);
// 输出，（有两个捕获）
// [
//     'ninja-ninja-sword',
//     'ninja-ninja-',
//     'ninja-'
// ]

console.log(res2);
// 输出，（只有一个捕获）
// [ 'ninja-ninja-sword', 'ninja-ninja-']
```

## 利用函数进行替换
### replace方法
1. 将正则表达式作为第一个参数时，会导致在该模式的匹配元素（全局匹配的话，就是多个匹配元素）上进行替换，而不是在固定字符串上进行替换。
2. 当替换值（第二个参数）是一个函数时，每个匹配都会调用该函数（记住，全局搜索会在源字符串中匹配所有的模式实例）并带有一串参数列表。函数的返回值是即将要替换的值。参数列表如下：
 * 匹配的完整文本
 * 匹配的捕获，一个捕获对应一个参数
 * 匹配字符串在源字符串中的索引
 * 源字符串

``` js
// 将横线字符串转换成驼峰拼写法
function upper(all,letter) {
    return letter.toUpperCase();
}

console.log('border-bottom-width'.replace(/-(\w)/g,upper)); // borderBottomWidth
```

``` js
// 压缩查询字符串的技术
function compress(source) {
    var keys = {};
    source.replace(/([^=&]+)=([^&]*)/g, function(full, key, value) {
        keys[key] = (keys[key] ? keys[key] + ',' : '') + value;
        return '';
    });

    var result = [];
    for(var key in keys) {
        result.push(key+'='+keys[key]);
    }
    return result.join('&');
}

console.log(compress("foo=1&foo=2&blah=a&blah=b&foo=3"));
// foo=1,2,3&blah=a,b
```

### 匹配换行符
``` js
var html = "<b>Hello</b>\n<i>world!</i>";
console.log(/.*/.exec(html)[0] === "<b>Hello</b>"); // true，说明换行符没有被匹配到
console.log(/[\S\s]*/.exec(html)[0] ===
    "<b>Hello</b>\n<i>world!</i>"); // true，最佳方案
console.log(/(?:.|\s)*/.exec(html)[0] ===
    "<b>Hello</b>\n<i>world!</i>"); // true
```

### RegExp 实例属性（来自JS高级程序设计）
RegExp 的每个实例都具有下列属性，通过这些属性可以取得有关模式的各种信息。

* global ：布尔值，表示是否设置了 g 标志。
* ignoreCase ：布尔值，表示是否设置了 i 标志。
* lastIndex ：整数，表示开始搜索下一个匹配项的字符位置，从 0 算起。
* multiline ：布尔值，表示是否设置了 m 标志。
* source ：正则表达式的字符串表示，按照字面量形式而非传入构造函数中的字符串模式返回。

``` js
var pattern1 = /\[bc\]at/i;
console.log(pattern1.global); //false
```
