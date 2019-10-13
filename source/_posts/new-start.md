---
title: new start
date: 2017-10-13 20:02:34
tags: 
- 第一篇 
- 测试
categories: 
- 日志
---

# 常用功能及写法

## 代码块
```js
console.log('HELLO WORLD');
```
## 使用more进行截断，以上显示在主页上
<!-- more --> 
 
``` html
<!-- more --> 
```

## 图片
``` md
{% asset_img yuki.jpg 大萌神%}
```
> 或直接使用markdown格式的图片

## 链接
[主页](https://sluggishpj.github.io/)

``` md
[主页](https://sluggishpj.github.io/)
```

## 文本居中的引用
{% cq %} blah blah blah {% endcq %}

``` md
{% cq %} blah blah blah {% endcq %}
```

## 引用块
>everything will be better!

```md
>everything will be better!
```

## 彩色引用块

{% note default %} Content (md partial supported) {% endnote %}

{% note primary %} Content (md partial supported) {% endnote %}


{% note success %} Content (md partial supported) {% endnote %}

{% note info %} Content (md partial supported) {% endnote %}

{% note warning %} Content (md partial supported) {% endnote %}

{% note danger %} Content (md partial supported) {% endnote %}

```md
{% note default %} Content (md partial supported) {% endnote %}
{% note primary %} Content (md partial supported) {% endnote %}
{% note success %} Content (md partial supported) {% endnote %}
{% note info %} Content (md partial supported) {% endnote %}
{% note warning %} Content (md partial supported) {% endnote %}
{% note danger %} Content (md partial supported) {% endnote %}
```

> 在两个彩色引用块之间插入md标签无效。。。原因未知。。。

## 引用书籍文章
{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}

```md
{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}
```

## 引用网络上的文章
{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}

``` md
{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}
```

# 常用命令

## 新建文章
``` bash
hexo new [layout] <title> 
```

## 启动服务器
``` bash
$ hexo server
```

## 生成静态文件
``` bash
$ hexo g
```

## 发布
``` bash
$ hexo publish [layout] <filename>
```

## 部署网站
``` bash
$ hexo d
```

## 生成静态文件并部署
``` bash
$ hexo g -d
```

> 建个博客真不容易，希望能坚持写下去。。。