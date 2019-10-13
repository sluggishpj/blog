---
title: JavaScript忍者秘籍笔记05之跨浏览器策略
date: 2017-10-28 08:52:36
tags:
- js
- 跨浏览器
- js忍者秘籍
categories:
- js
- js忍者秘籍
---

## 浏览器bug修复
<!-- more -->
``` js
<form id="form" action="/conceal">
    <input type="text" id="action" />
    <input type="submit" id="submit" />
</form>
<script>
document.getElementById('form').submit();
// TypeError: document.getElementById(...).submit is not a function
</script>
```
> * 浏览器将表单里引用的每个input元素都作为`<form>`元素的属性了。当所添加属性的名称是input属性的`id`值时就可能会出现问题了。如果id值恰好是form元素已经存在的属性，这些原始属性就会被新属性取而代之，如`action`或`submit`。
> * 因此，在创建`input#submit`元素之前，`form.action`引用指向的是`<form>`的`action`特性的值，但在创建元素之后，指向的就是`input#submit`元素了。

### 特征仿真
```js
window.findByTagWorksAsExpected = (function() {
    var div = document.createElement('div');
    div.appendChild(document.create('test'));
    return div.getElementsByTagName('*').length === 0;
})();
```

## 元素特性和属性
特性（attribute）是DOM构建的一个组成部分，而属性（property）是元素保持运行时信息的主要手段，并且通过属性可以访问这些运行时信息。
``` js
var image = document.getElementsByTagName('img')[0];
var newSrc = 'images/kong.png';

image.src = newSrc;

console.log(image.src); // file:///F:/Web/sublimeProject/test/images/kong.png
console.log(image.getAttribute('src')); // images/kong.png
```

### 自定义特性的行为
如果不确定一个特性的属性是否存在，可以对其进行测试，如果不存在的话再使用DOM方法。eg、
``` js
var value = element.someValue? element.someValue: element.getAttribute('someValue');
```

### URL规范化
在访问一个引用了URL的属性（eg，href，src或action），该URL值会自动将原始值转换成完整规范的URL。

### 获取计算样式
1. Window.getComputedStyle() 方法给出应用活动样式表后的元素的所有CSS属性的值，并解析这些值可能包含的任何基本计算。

2. 语法

``` js
let style = window.getComputedStyle(element, [pseudoElt]);
```

* `element`：用于获取计算样式的Elemen
* `pseudoElt`：可选。指定一个要匹配的伪元素的字符串。必须对普通元素省略（或null）。
* 返回的样式是一个实时的 CSSStyleDeclaration 对象，当元素的样式更改时，它会自动更新本身。该对象提供一个名为`getPropertyValue()`的方法，接受CSS属性名称（如font-size和background-color），而不是“驼峰式”格式的名称。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>测试getComputedStyle方法</title>
    <style>
    #box {
        font-size: 16px;
    }
    </style>
</head>

<body>
    <div id="box">我是box</div>

    <script>
    var oBox = document.getElementById('box');
    var compSty = window.getComputedStyle(oBox);

    console.log(compSty.getPropertyValue('font-size')); // 16px

    oBox.style.fontSize = '14px';
    console.log(compSty.getPropertyValue('font-size')); // 14px
    </script>
</body>

</html>
```

## 绑定和解绑事件处理程序
1. IE9之前版本：`attachEvent()`和`detachEvent()`方法
2. 现代浏览器：`addEventListener()`和`removeEventListener()`方法

## 冒泡与委托
一个元素被单击的时候可以通过event.target获得该元素的引用。
``` js
// 将事件处理委托给表格元素+
var table = document.getElementById('someId');
table.addEventListener('click',function(event) {
    if(event.target.tagName.toLowerCase() == 'td') {
        event.target.style.backgroundColor = 'yellow';
    }
});
```

## 向DOM中注入HTML
有些HTML元素在被注入之前，必须存放在特定的容器元素中，例如，`<option>`元素必须放在`<select>`中。需要使用特定容器元素进行包装的问题元素有7个：
* `<option>` and `< optgroup>` need to be contained in a `<select multiple="multiple">...</select>`
* `<legend>` needs to be contained in a `<fieldset>...</fieldset>`
* `<thead>` , `< tbody>` , `< tfoot>` , `< colgroup>` , and `< caption>` need to be contained in a`<table>...</table>`
* `<tr>` needs to be in a `<table><thead>...</thead></table>` , `<table><tbody>...</tbody></table>` , or `<table><tfoot>...</tfoot></table>`
* `<td>` and `<th>` need to be in a `<table><tbody><tr>...</tr></tbody></table>`
* `<col>` must be in a `<table><tbody></tbody><colgroup>...</colgroup></table>`
* `<link>` and `<script>` need to be in a `<div></div><div>...</div>`
