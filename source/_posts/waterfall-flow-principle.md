---
title: 瀑布流布局-原理
date: 2017-10-16 23:14:46
tags:
- 瀑布流布局
- css
- js
categories:
- 布局
---

## 效果
各div宽度一样，高度由内容撑开。所有div紧密排列在一起，效果如下：
![图片瀑布流布局](https://s2.ax1x.com/2019/10/13/uxckVg.png)

<!-- more -->

[点击在线预览](http://sluggish.gitee.io/staticpages/waterfall/js-static-ver/index.html)

## 参考
学习了慕课网上的瀑布流布局课程，参考实现了一下，做个总结。
> 课程地址：http://www.imooc.com/learn/101

## 步骤
1. 先获取**第一行**中各div高度，保留在数组中，作为各列初始高度。假设一排有`n`个div
2. 获取各列高度的最小项（数组最小项），将第`n+1`个div移动到高度最低的列的下方
3. 高度最低的列的高度（数组对应的项）加上第`n+1`个div的自身高度，n++
4. 重复步骤2，直到所有div排序完毕

![图片瀑布流布局](https://s2.ax1x.com/2019/10/13/uxcDde.png)
图片中1,2,3...是对应div元素在html文档中的排列顺序。

## 核心代码
``` js
/**
 * 瀑布流函数
 * @param  {[type]} oDivs     页面所有div构成的dom数组
 * @param  {[type]} divWidth  每个div的宽度
 * @param  {[type]} columnNum 列数
 */
function waterfall(oDivs,divWidth,columnNum) {
    var columnHeightList = []; // 保存各列高度
    var targetIndex = 0; // 列高度最低所在的列序号
    for (i = 0; i < oDivs.length; i++) {
        var mydiv = oDivs[i];
        if (i < columnNum) {
            columnHeightList.push(mydiv.offsetHeight); // 保存各列初始高度
            mydiv.style.cssText = 'position:relative;float:left;';
        } else {
            // 获取高度最低所在的列序号（从0开始）
            targetIndex = columnHeightList.indexOf(Math.min.apply(null, columnHeightList));

            // 将div移动到最低列的下方
            mydiv.style.cssText = 'position:absolute';
            mydiv.style.left = targetIndex * divWidth + 'px';
            mydiv.style.top = columnHeightList[targetIndex] + 'px';

            // 将所在的列高度加上div高度
            columnHeightList[targetIndex] += mydiv.offsetHeight;
        }
    }
}
```

## 图片无限加载
思路是检测当前`屏幕滚动的高度`和`浏览器窗口高度`之和 跟 `最低图片的具页面顶部高度`比较，满足一定条件便加载图片。
![图片无线加载](https://s2.ax1x.com/2019/10/13/uxcfL8.png)

``` js
// 页面滚动时触发
window.onscroll = function() {
    var lastDiv = oDivs[oDivs.length - 1];
    var lastDivTop = lastDiv.offsetTop; // 距离页面顶部的高度
    var lastDivHeight = lastDiv.offsetHeight; // 自身高度
    var scrollTop = document.documentElement.scrollTop || document.body.scrollTop; //注意解决兼容性
    var bodyHeight = document.documentElement.clientHeight; //可视区高度

    // 加载图片的条件
    if ((scrollTop + bodyHeight) > (lastDivTop + lastDivHeight / 2)) {
      // 加载图片代码...
    }
}
```

## 小结
1. 在js中设置元素样式记得加单位
2. 后续会实现JS动画版和CSS3版本
