---
title: 瀑布流布局-动画
date: 2017-10-19 00:04:51
tags:
- 布局
- css
- js
categories:
- 布局
---

## 前言
上一篇介绍了瀑布流布局的原理，这篇是在添加动画时的总结。
以下的`.box`是要进行布局的div的类名
<!-- more -->

## transform版本
1. 在页面的css中设置绝对定位和transition属性，设置运动时间
``` css
.box {
    position: absolute;
    transition: transform 0.6s linear;
}
```
> 绝对定位是为了方便控制div的运动位置，但也为后面埋下一点坑

2. 在触发页面瀑布流布局时用transform属性代替left和top定位
``` js
/**
 * 瀑布流函数
 * @param  {[type]} oBoxs     图片box数组
 * @param  {[type]} oBoxWidth 图片box宽度
 * @param  {[type]} columnNum 图片列数
 */
function waterfall(oBoxs, oBoxWidth, columnNum) {
    var columnHeightList = [];
    var targetIndex = 0;
    for (var i = 0; i < oBoxs.length; i++) {
        var box = oBoxs[i];
        if (i < columnNum) { // 第一行图片
            columnHeightList.push(box.offsetHeight)
            box.style.transform = 'translate(' + oBoxWidth * i + 'px,0px)';
        } else { // 其余图片
            // 获取高度最低的那一列序号
            targetIndex = columnHeightList.indexOf(Math.min.apply(null, columnHeightList));

            // 改动的地方
            box.style.transform = 'translate(' + targetIndex * oBoxWidth + 'px,' + columnHeightList[targetIndex] + 'px)';

            // 最低列高度加上该div高度
            columnHeightList[targetIndex] += box.offsetHeight;
        }
    }
}
```
> 动画预览地址：http://sluggish.gitee.io/staticpages/waterfall/js-animate-ver/index2.html

## 改善transform版本
1. 从预览demo知，当图片加载越多时，图片下降速度变得越来越快。原因在于每次加载进去的div都是从`left:0;top:0`运动到目标位置，由于运动时间不变，垂直距离变长，垂直速度也随之加快。

2. 解决方法：修改每次加载进来的图片的top值，同时在waterfall函数中的translate垂直距离应随着top的改变作出对应改变。

### 加载进来的图片进行处理

``` js
// 将图片包装成box后，加载到mainDiv中【maindiv是所有box所在的div】。cb是图片加载完的回调
function loadImg(mainDiv, cb) {
    // 模拟后台数据
    var imgData = { data: [{ 'src': '1.jpg' }, { 'src': '2.jpg' }, { 'src': '3.jpg' },{ 'src': '4.jpg' }, { 'src': '5.jpg' }, { 'src': '6.jpg' }] };
    var prefix = '../../source/image/';
    var imgLen = imgData.data.length;

    imgData.data.forEach(function(val, index) {
        var oPic = document.createElement('div');
        oPic.className = 'pic';
        var oBox = document.createElement('div');
        oBox.className = 'box';
        var img = document.createElement('img');
        img.src = prefix + val.src;

        oPic.appendChild(img);
        oBox.appendChild(oPic);
        mainDiv.appendChild(oBox);


        var scrollTop = document.documentElement.scrollTop || document.body.scrollTop; //注意解决兼容性
        var bodyHeight = document.documentElement.clientHeight; //可视区高度

        // 解决图片加载过长时下降速度过快
        oBox.style.left = '0';
        oBox.style.top = 20+'px';
        oBox.style.top = scrollTop-bodyHeight+'px';

        // 让所有图片加载完后执行回调，避免新加进来的图片高度获取不到，产生叠加
        img.onload = function() {
            if (!--imgLen) {
                console.log('执行图片回调')
                cb();
            }
        };
    });
}
```

### waterfall函数改造
``` js
/**
 * 瀑布流函数
 * @param  {[type]} oBoxs     图片box数组
 * @param  {[type]} oBoxWidth 图片box宽度
 * @param  {[type]} columnNum 图片列数
 */
function waterfall(oBoxs, oBoxWidth, columnNum) {
    var columnHeightList = [];
    var targetIndex = 0;
    var y = 0; // 保存要移动的垂直高度

    for (var i = 0; i < oBoxs.length; i++) {
        var box = oBoxs[i];
        if (i < columnNum) { // 第一行图片
            columnHeightList.push(box.offsetHeight)
            box.style.transform = 'translate(' + oBoxWidth * i + 'px,0px)';
        } else { // 其余图片
            // 获取高度最低的那一列序号
            targetIndex = columnHeightList.indexOf(Math.min.apply(null, columnHeightList));

            // 获取要移动的垂直高度
            var y = columnHeightList[targetIndex]-box.style.top.replace('px','');
            box.style.transform = 'translate(' + targetIndex * oBoxWidth + 'px,' + y + 'px)';

            // box.style.transform = 'translate(' + targetIndex * oBoxWidth + 'px,' + columnHeightList[targetIndex] + 'px)';

            columnHeightList[targetIndex] += box.offsetHeight;
        }
    }
}
```

> 动画预览地址：http://sluggish.gitee.io/staticpages/waterfall/js-animate-ver/index.html

## jq版本
jq版本是使用jq的animate函数进行left和top定位。直接贴waterfall函数代码
``` js
/**
 * 瀑布流函数
 * @param  {[type]} $oBoxs     图片box数组
 * @param  {[type]} oBoxWidth 图片box宽度
 * @param  {[type]} columnNum 图片列数
 * @param  {[type]} i         第i张图片起调整成瀑布流布局
 * @param  {[type]} columnHeightList 存放列高度的数组
 */
function waterfall($oBoxs, oBoxWidth, columnNum, i, columnHeightList) {
    var targetIndex = 0,
        targetTop = 0;
    for (; i < $oBoxs.length; i++) {
        var $box = $($oBoxs[i]);
        var boxHeight = Math.floor($box[0].offsetHeight);
        if (i < columnNum) { // 第一行
            columnHeightList.push(boxHeight)
            $box.css({ 'position': 'absolute' });
            $box.animate({
                    'left': Math.floor(i * oBoxWidth) + 'px',
                    'top': '0px'
                },500,function() {
                    console.log('动画动作完成1！');
                });
        } else {
            targetIndex = columnHeightList.indexOf(Math.min.apply(null, columnHeightList));
            targetTop = columnHeightList[targetIndex];
            // 注意加单位啊！！！
            $box.css('position', 'absolute')

            $box.animate({
                    'left': Math.floor(targetIndex * oBoxWidth) + 'px',
                    'top': Math.floor(targetTop) + 'px'
                },
                500,function() {
                    console.log('动画动作完成2！');
                });

            // 将所在的列高度加上图片高度
            columnHeightList[targetIndex] += boxHeight;
        }
    }
}
```
> 新增参数i是为了在垂直滚动时，只让新加入的图片进行定位，其他图片定位不变

> 预览地址http://sluggish.gitee.io/staticpages/waterfall/jq-animate-ver/index.html

## 注意事项
1. 加载图片是要在加载完后执行回调，避免新加进来的图片高度没获取到，产生叠加
2. 使用translate后，则div距离页面顶部的高度通过offsetTop获取不到！
需要自己封装获取translateX和translateY的值
``` js
// 获取元素的translateX和translateY数值，传入一个dom元素，
// 返回一个对象，含属性transX和transY
function getTransform(dom) {
    var transform = window.getComputedStyle(dom).transform;
    if(transform == 'none') { // 没使用transform的情况下
        return {
            'tranX':0,
            'tranY':0
        };
    }
    // transform值是这种字符串 "matrix(1, 0, 0, 1, 408, 190)"
    var tempArr = transform.split(',');
    return {
        'transX': parseInt(tempArr[4]),
        'transY': parseInt(tempArr[5].substring(0, tempArr[5].length - 1))
    }
}
```
> 关于matrix，可以参考这篇文章:http://www.zhangxinxu.com/wordpress/2012/06/css3-transform-matrix-%E7%9F%A9%E9%98%B5/

## 总结
> 源码地址（源码都做了注释）：https://gitee.com/sluggish/staticpages/tree/master/waterfall
