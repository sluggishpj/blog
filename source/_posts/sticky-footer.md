---
title: sticky-footer
date: 2018-04-21 16:24:05
tags:
- sticky-footer布局
- css
categories:
- 布局
---



## 目的

实现sticky-footer布局

<!-- more -->

## 布局

![sticky-footer.png](https://gitee.com/sluggish/music/raw/master/sticky-footer.png)



## absolute方案

```css
html,
body {
    padding: 0;
    margin: 0;
}

.wrapper {
    position: relative;
    min-height: 100vh;
}

.content {
    /*100px是footer高度*/
    padding-bottom: 100px;
}


.footer {
    position: absolute;
    bottom: 0;
    width: 100%;
    height: 100px;
}
```



## margin方案

```css
html,
body {
    padding: 0;
    margin: 0;
}

.content {
    min-height: 100vh;
    box-sizing: border-box;
    /*100px是footer高度*/
    padding-bottom: 100px;
}

.footer {
    height: 100px;
    /*-100px是-footer高度*/
    margin-top: -100px;
}
```



## flex方案

```css
html,
body {
    padding: 0;
    margin: 0;
}

.wrapper {
    min-height: 100vh;
    display: flex;
    flex-flow: column;
}

.content {
    flex: 1;
}

.footer {
    height: 100px;
}
```



## calc方案

```css
html,
body {
    padding: 0;
    margin: 0;
}

.content {
    /*100px 是footer高度*/
    min-height: calc(100vh - 100px);
}

.footer {
    height: 100px;
}
```



## grid方案

```css
html, body {
    margin: 0;
    padding: 0;
}

.wrapper {
    min-height: 100vh;
    display: grid;
    /* 100px 为footer高度 */
    grid-template-rows: 1fr 100px;
}
```

