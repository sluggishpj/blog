---
title: 标签页间通信
date: 2017-11-22 21:55:44
tags:
- js
- 标签页
- 窗口
- 通信
categories:
- 综合知识点
---

## 目的

实现浏览器内多个标签页之间的通信，有4种方法

<!-- more -->

## 使用window.opener【不支持跨域】

适用于新标签页是主标签页使用window.open(...)打开的情况。window.opener返回打开当前窗口的那个窗口的引用。

```html
<!-- main.html -->
<h1>主窗口</h1>
<h3>通过window.open(...)打开另一个隔壁窗口：</h3>
<button id="openNew">打开隔壁窗口</button>
<h3>通过window.open(...)的返回值改变隔壁窗口背景</h3>
<button id="turnRed">变红</button>
<button id="turnOrange">变橙</button>
<button id="turnYellow">变黄</button>

<script>
var $openNew = $('#openNew');
var anotherWindow = null;
$openNew.on('click', function() {
    anotherWindow = window.open('otherWindow.html'); // 打开新隔壁标签页
});
$('#turnRed').on('click', function() {
    anotherWindow.document.bgColor = 'red';
});
$('#turnOrange').on('click', function() {
    anotherWindow.document.bgColor = 'orange';
});
$('#turnYellow').on('click', function() {
    anotherWindow.document.bgColor = 'yellow';
});
</script>
```

```html
<!-- otherWindow.html -->
<h1>隔壁窗口</h1>
<h3>通过window.opener改变主窗口颜色</h3>
<button id="turnGreen">变绿</button>
<button id="turnBlue">变蓝</button>
<button id="turnPurple">变紫</button>

<script type="text/javascript">
$('#turnGreen').on('click', function() {
    window.opener.document.bgColor = 'green';
});
$('#turnBlue').on('click', function() {
    window.opener.document.bgColor = 'blue';
});
$('#turnPurple').on('click', function() {
    window.opener.document.bgColor = 'purple';
});
</script>
```

> 在线演示：http://sluggish.gitee.io/staticpages/tab-communication/window-opener/mainWindow.html
>
> 参考链接：http://www.javascriptkit.com/javatutors/remote2.shtml

​

## 使用window.postMessage()【支持跨域】

方法：`otherWindow.postMessage(message, targetOrigin, [transfer])`

​        `window.addEventListener("message", receiveMessage, false);`

- otherWindow

  ​	其他窗口的一个引用，比如`iframe的`contentWindow`属性、执行`window.open`返回的窗口对象、或者是命名过或数值索引的`window.frames`。

- message

  ​	将要发送到其他 window的数据。它将会被结构化克隆算法序列化。这意味着你可以不受什么限制的将数据对象安全的传送给目标窗口而无需自己序列化。

- targetOrigin

  ​	通过窗口的origin属性来指定哪些窗口能接收到消息事件，其值可以是字符串"\*"（表示无限制）或者一个URI。在发送消息的时候，如果目标窗口的协议、主机地址或端口这三者的任意一项不匹配targetOrigin提供的值，那么消息就不会被发送；只有三者完全匹配，消息才会被发送。这个机制用来控制消息可以发送到哪些窗口；例如，当用`postMessage`传送密码时，这个参数就显得尤为重要，必须保证它的值与这条包含密码的信息的预期接受者的orign属性完全一致，来防止密码被恶意的第三方截获。**如果你明确的知道消息应该发送到哪个窗口，那么请始终提供一个有确切值的targetOrigin，而不是\*。不提供确切的目标将导致数据泄露到任何对数据感兴趣的恶意站点。**

- `transfer` 可选

  ​	是一串和message 同时传递的 Transferable对象. 这些对象的所有权将被转移给消息的接收方，而发送一方将不再保有所有权。

![主窗口](https://s2.ax1x.com/2019/10/13/uxy77Q.png)
![另一个窗口](https://s2.ax1x.com/2019/10/13/uxyqts.png)


```js
// 主页面
var $openNew = $('#openNew'); // 打开新窗口的按钮
var $sendText = $('.send-text'); // 发送input内容的按钮
var $content = $('.content'); // 输入文字内容的input
var popup = null; // 目标窗口的引用

// 点击打开新窗口
$openNew.on('click', function() {
    popup = window.open('https://sluggishpj.github.io/staticPages/index.html'); // 新窗口的引用，稍后发消息要用到
    setTimeout(function() {
        popup.postMessage('连接上了', 'https://sluggishpj.github.io'); // 过2s发送第一条消息
    }, 2000);
});

// 点击发送按钮发送
$('.send').click(function() {
    var text = $sendText.val();
    popup.postMessage(text, 'https://sluggishpj.github.io'); // 发送到目标窗口
    $('<p>').text('我：' + text).appendTo($content);
    $sendText.val('');
});

// 处理别的窗口发送过来的消息
function receiveMsg(event) {
    if (event.origin !== 'https://sluggishpj.github.io') { // 判断来源是否符合要求
        return;
    }
    $('<p>').text('对方（' + event.origin + '）: ' + event.data).appendTo($content);
}

window.addEventListener('message', receiveMsg, false); // 监听别的窗口发送过来的消息事件
```

```js
// 从主页面中打开的页面
var eventSource = null; // 目标窗口的引用
var eventOrigin = null; // 目标窗口的协议，主机地址和端口

// 点击发送input内文字
$('.send').click(function() {
    var text = $sendText.val();
    eventSource.postMessage(text, eventOrigin);
    $('<p>').text('我：' + text).appendTo($content);
    $sendText.val('');
});

window.addEventListener('message', receiveMsg, false);

function receiveMsg(event) {
    eventOrigin = event.origin; 
    eventSource = event.source; // 保存发消息给本页面的窗口的引用

    if (eventOrigin !== 'http://sluggish.gitee.io') {
        return;
    }
    $('<p>').text('对方（' + eventOrigin + '）: ' + event.data).appendTo($content);
}
```

> 在线演示：http://sluggish.gitee.io/staticpages/tab-communication/postMessage/
>
> 参考链接：https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage
>
> 主页面源码：https://gitee.com/sluggish/staticpages/blob/master/tab-communication/postMessage/index.html
>
> 次页面源码：https://github.com/sluggishpj/staticPages/blob/gh-pages/index.html
>
> MDN上的例子：https://mdn.github.io/dom-examples/web-storage/index.html

​

## 使用cookies【不支持跨域】

原理：多个页面共享同样的cookie

```js
var CookieUtil = {
    get: function(name) {
        var cookieName = encodeURIComponent(name) + '=',
            cookieStart = document.cookie.indexOf(cookieName),
            cookieValue = null;
        if (cookieStart > -1) {
            var cookieEnd = document.cookie.indexOf(';', cookieStart);
            if (cookieEnd == -1) {
                cookieEnd = document.cookie.length;
            }
            cookieValue = decodeURIComponent(document.cookie.substring(cookieStart + cookieName.length, cookieEnd));
        }
        return cookieValue;
    },

    // expires传入的是过期那个日期 或者 距离过期的天数
    set: function(name, value, expires, path, domain, secure) {
        var cookieText = encodeURIComponent(name) + '=' + encodeURIComponent(value),
            exp;

        if (expires instanceof Date) { // 具体日期
            cookieText += '; expires=' + expires.toUTCString();
        } else if (typeof expires === 'number') { // 过期天数
            exp = new Date();
            exp.setTime(exp.getTime() + expires * 1000 * 3600 * 24);
            cookieText += '; expires=' + exp.toUTCString();
        }

        if (path) {
            cookieText += '; path=' + path;
        }
        if (domain) {
            cookieText += '; domain=' + domain;
        }
        if (secure) {
            cookieText += '; secure';
        }
        document.cookie = cookieText;
    },

    remove: function(name, path, domain, secure) {
        this.set(name, '', new Date(0), path, domain, secure);
    }
};
```

> 在线演示：http://sluggish.gitee.io/staticpages/tab-communication/cookies/
>
> 参考链接：https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cookie
>
> 参考书籍：《JavaScript高级程序设计》

​

## 使用localStorage【不支持跨域】

方法：

- 存储数据：localStorage.setItem(key, value)
- 获取数据：localStorage.getItem(key)
- 删除特定数据：localStorage.removeItem(key)
- 清空域名对应的整个存储对象：localStorage.clear()
- 没有过期时间

> 在线演示：http://sluggish.gitee.io/staticpages/tab-communication/localStorage/
>
> 参考链接：https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage