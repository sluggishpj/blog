---
title: css-secrets-note
date: 2018-03-02 00:03:20
tags:
- css
- css3
categories:
- css
---

##  说明

内容参考自《CSS揭秘》，本博文只记录CSS的用途，不做详细解析，详细请看书

<!-- more -->

## 引言

CSS编码技巧

### 尽量减少代码重复

代码可维护性的最大要素是尽量减少改动时要编辑的地方

当某些值相互依赖时，应该把它们的相互关系用代码表达出来。例如，行高和字号的关系。把字号改为相对于行高而不是绝对高度，可以避免再修改一个值。同理字体尺寸的样式，如果改用百分比或 `em` 单位就好多了
```css
font-size: 20px;
line-height: 1.5;
```

需要重新审视到底哪些效果应该跟着按钮一起放大（使用em或rem或百分比），而哪些效果是保持不变的（使用px）。



### HSLA(H,S,L,A)

取值：

* H：Hue(色调)。0(或360)表示红色，120表示绿色，240表示蓝色，也可取其他数值来指定颜色。取值为：0 - 360
* S：Saturation(饱和度)。取值为：0.0% - 100.0%
* L：Lightness(亮度)。取值为：0.0% - 100.0%
* A：Alpha透明度。取值0~1之间。




### 尽量减少代码重复

### 代码易维护vs代码量少


我们要为一个元素添加一道 10px 宽的边框，但左侧不加边框。

```css
border-width: 10px 10px 10px 0;
/* 改成下面两条可读性和可维护性更好 */
border-width: 10px;
border-left-width: 0;
```



### currentColor：CSS有史以来第一个变量

假设我们想让所有的水平分割线（所有`<hr>` 元素）自动与文本的颜色保持一致。有了 currentColor 之后，我们只需要这样写：
```css
hr {
    height: .5em;
    background: currentColor;
}
```



### inherit 关键字

对于背景色同样非常有用。举个例子，在创建提示框的时候，你可能希望它的小箭头能够自动继承背景和边框的样式：
![css-secrets-inherit.png](https://gitee.com/sluggish/music/raw/master/css-secrets-inherit.png)

```css
.callout {
    position: relative;
}

.callout::before {
    content: "";
    position: absolute;
    top: -.4em;
    left: 1em;
    padding: .35em;
    background: inherit;
    border: inherit;
    border-right: 0;
    border-bottom: 0;
    transform: rotate(45deg);
}
```



### 关于响应式网页设计

下面还有一些建议，可能会帮你避免不必要的媒体查询

* 使用百分比长度来取代固定长度。如果实在做不到这一点，也应该尝试使用与**视口**相关的单位（ vw 、 vh 、 vmin 和 vmax ），它们的值解析为视口宽度或高度的百分比。
* 当你需要在较大分辨率下得到固定宽度时，使用 max-width 而不是width ，因为它可以适应较小的分辨率，而无需使用媒体查询。
* 不要忘记为替换元素（比如 img 、 object 、 video 、 iframe 等）设置一个 max-width ，值为 100% 。
* 假如背景图片需要完整地铺满一个容器，不管容器的尺寸如何变化，background-size: cover 这个属性都可以做到。但是，我们也要时刻牢记——带宽并不是无限的，因此在移动网页中通过 CSS 把一张大图缩小显示往往是不太明智的。
* 当图片（或其他元素）以行列式进行布局时，让视口的宽度来决定列的数量。弹性盒布局（即 Flexbox）或者 display: inline-block加上常规的文本折行行为，都可以实现这一点。
* 在使用多列文本时，指定 column-width （列宽）而不是指定column-count （列数），这样它就可以在较小的屏幕上自动显示为单列布局。

总的来说，我们的思路是尽最大努力实现弹性可伸缩的布局，并在媒体查询的各个断点区间内指定相应的尺寸。



### 合理使用简写

```css
background: rebeccapurple;
background-color: rebeccapurple;
```
>  合理使用简写是一种良好的防卫性编码方式，可以抵御未来的风险。当然，如果我们要明确地去覆盖某个具体的展开式属性并保留其他相关样式，那就需要用展开式属性



## 背景与边框

### 半透明边框

默认状态下，背景会延伸到边框的区域下层。 如果不希望背景侵入边框所在的范围，我们要做的就是把它的`background-clip`设为 `padding-box`



### 多重边框

#### box-shadow方案

一个正值的扩张半径加上两个为零的偏移量以及为零的模糊值，得到的“投影”其实就像一道实线边框 box-shadow 的好处在于，它支持逗号分隔语法，我们 可以创建任意数量的投影

```css
.mydiv {
    box-shadow: 0 0 0 10px #655, 0 0 0 15px deeppink;
}
```

> * box-shadow 是层层叠加的，第一层投影位于最顶 层，依次类推。因此，你需要按此规律调整扩张半径
>
>
> * 投影的行为跟边框不完全一致，因为它不会影响布局，而且也不会 受到 box-sizing 属性的影响。不过，你还是可以通过内边距或外边 距（这取决于投影是内嵌和还是外扩的）来额外模拟出边框所需要 占据的空间
> * 上述方法所创建出的假“边框”出现在元素的外圈。它们并不会响 应鼠标事件，比如悬停或点击。如果这一点非常重要，你可以给 box-shadow 属性加上 inset 关键字，来使投影绘制在元素的内圈。 请注意，此时你需要增加额外的内边距来腾出足够的空隙



#### outline方案

* box-shadow只能模拟实线边框
* outline可以模拟虚线边框
  * outline-offset控制元素与元素边缘的间距
  * 只适用于双层“边框”场景，不支持逗号分隔的多个值



### 灵活的背景定位

目标：灵活控制背景图片位置

#### background-position扩张语法

允许我们指定**背景图片距离任意角的偏移量**，只要我们在偏移量前面指定关键字。举例来说，让背景图片跟右边缘保持 20px 的偏移量，同时跟底边保持 10px 的偏移量，可以这样做

``` css
background: url(code-pirate.svg)
            no-repeat bottom right #58a; /* bottom right提供回退*/
background-position: right 20px bottom 10px;
```


#### background-origin方案

* 目的：设置背景图片偏移量与容器的内边距一致

`background-position` 默认是以 `padding box` 为准的，因此， top left 默认指的是 padding box 的**左上角**

`background-origin` ，可以用它来**改变**这种行为，默认为`padding-box`，可以将其设置为`content-box`，则`background-position`就会以**内容区**的边缘为标准

```css
padding: 10px;
background: url("code-pirate.svg") no-repeat #58a bottom right; /* 或 100% 100% */
background-origin: content-box;
```



#### calc()方案

可以在`background-position`中使用

```css
background: url("code-pirate.svg") no-repeat;
background-position: calc(100% - 20px) cal(100% - 10px)
```

> 注意：calc() 函数内部的 - 和 + 运算符的两侧要加 一个**空白符**，否则会产生解析错 误！这个规则如此怪异，是为了向前兼容：未来，在 calc() 内部 可能会允许使用关键字，而这些 关键字可能会包含连字符（即减号）



### 边框内圆角

目标：如下图

![css-secrect-inner-border-circle.png](https://gitee.com/sluggish/music/raw/master/css-secrect-inner-border-circle.png)

解决方案：

* 两个元素（不展开）
* 一个元素

描边（outline）不会跟着元素的圆角走，box-shadow会。

```css
background: tan;
border-radius: .8em;
padding: 1em;
box-shadow: 0 0 0 .6em #655;
outline: .6em solid #655;
```

> 扩张半径需要比描边的宽度值小，但它同时又要比` ( √2-1)*border-radius` 大，可以直接设置为其一半



### 条纹背景

```css
background: linear-gradient(to bottom, red 25%, orange 0, orange 50%, yellow 0, yellow 75%, blue 75%);
```

<script async src="//jsfiddle.net/sluggishpj/xodcp2t6/embed/html,css,result/dark/"></script>

> 产生4色块
>
> `red 25%`表示结束位置是在25%，因为是第一个颜色，所以开始位置默认是0%
>
> 如果某个色标的位置比整个列表中它之前的色标的位置都要小，则该色标的位置会被设置为它前面所有色标位置值的最大值。所以上面的`orange 0`相当于`orange 25%`
>
> `orange 0`此处表示开始位置为25%
>
> `orange 50%`表示结束为值为50%
>
> `yellow 0` 此处表示开始位置为50%
>
> `yellow 75%`表示结束为值为75%
>
> `blue 75%`表示开始位置为75%，又因为是最后一个，其结束位置为100%



#### 垂直条纹

#### 斜向条纹

#### 灵活的同色系条纹



### 复杂的背景图案

#### 网格

#### 波点和棋盘

通过`background-position`调整

```css
background: #655;
background-image: radial-gradient(tan 30%, transparent 0), radial-gradient(tan 30%, transparent 0);
background-size: 30px 30px;
background-position: 0 0, 15px 15px;
```

处于可读性考虑，需要把一句CSS代码打断为多行，只需要用反斜杠(\\)来转义每行末尾的换行就可以了

```css
background: #eee url('data:image/svg+xml,\
        <svg xmlns="http://www.w3.org/2000/svg" \
        width="100" height="100" \
        fill-opacity=".25">\
        <rect x="50" width="50" height="50" /> \
        <rect y="50" width="50" height="50" /> \
</svg>');
```

### 伪随机背景

### 连续的图像边框

* 两元素实现（看书）
* 单元素实现

思路：在图片之上，再叠加一层纯白的实色背景，给两层背景指定不同的`background-clip`，注意的是，我们只能在多重背景的最底层设置背景色，因此需要用一道从白色过渡到白色的CSS渐变来模拟纯白色背景

```css
padding: 1em;
border: 1em solid transparent;
background: linear-gradient(white, white), url(stone-art.jpg);
background-size: cover;
background-clip: padding-box, border-box;
background-origin: border-box;
```



## 形状

### 自适应椭圆

> 当任意两个相邻圆角的半径之和超过border box的尺寸时，用户代理必须按比例减小各个边框半径所使用的值，直到它们不会相互重叠

解决方案

* border-radius可以单独指定水平和垂直半径，只要用一个斜杠(/)分隔这两个值即可。因此可以创建椭圆圆角
* 创建一个自适应的椭圆

```css
border-radius: 50%/50%;
```



#### 半椭圆

border-radius是一个简写属性，可以展开如下：

* border-top-left-radius
* border-top-right-radius
* border-bottom-right-radius
* border-bottom-left-radius

可以给border-radius设置四个值，以空格隔开，分别从左上角开始以顺时针应用到各个角。如果忽略设置某个值，会以CSS的常规方式重复，第一个值和第三个值相同，第二个字和第四个值相同

可以为所有四个角提供完全不同的水平和垂直半径，方法是在斜杠前指定1~4个值（水平），在斜杠后指定另外1~4个值（垂直）。这两组值是单独展开为四个值的。

```css
border-radius: 10px / 5px 20px;
/* 等同于 */
border-radius: 10px 10px 10px 10px / 5px 20px 5px 20px;
```



#### 四分之一椭圆



### 平行四边形

#### 嵌套元素解决方案

容器使用skew()，对内容再应用一次反向的skew()，从而抵消到容器的变形

```css
.container { transform: skewX(-45deg); }
.container > div { transform: skewX(45deg); }
```



#### 伪元素解决方案

把所有样式（背景，边框等）应用到伪元素上，然后再对伪元素进行变形

```css
.container {
    position: relative;
}
.container::before {
    content: ''; /* 用伪元素来生成一个矩形 */
    position: absolute;
    top: 0; right: 0; bottom: 0; left: 0;
    z-index: -1; /* 宿主元素之后 */
    background: #58a;
    transform: skew(45deg);
}
```



### 菱形图片

#### 基于变形的方案

* 通过rotate和scale

```css
.picture {
    width: 400px;
    transform: rotate(45deg);
    overflow: hidden;
}
.picture {
    max-width: 100%;
    transform: rotate(-45deg) scale(1.42); /* 1.42 > √2*/
}
```



#### 裁切路径方案

* 使用clip-path属性，还可以参与动画

```css
img {
    transition: all 2s;
    clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
}
img:hover {
    clip-path: polygon(0 0, 100% 0, 100% 100%, 0 100%);
}
```



### 切角效果

#### 渐变解决

使用预处理器sass

```scss
@mixin beveled-corners($bg, $tl:0, $tr:$t1, $br:$t1, $bl:$tr) {
    background: $bg;
    background: 
        linear-gradient(135deg, transparent $t1, $bg 0) top left,
        linear-gradient(225deg, transparent $tr, $bg 0) top left,
        linear-gradient(-45deg, transparent $br, $bg 0) top left,
        linear-gradient(45deg, transparent $b1, $bg 0) top left,
}

@include beveled-corners(#58, 15px, 5px);
```

> tl：上左角(top left)，tr：上右角，br：下右角，bl：下左角



##### 弧形切角



#### 内联SVG与border-image方案

#### 裁切路径方案

```css
background: #58a;
clip-path: polygon(20px 0, calc(100% - 20px) 0, 100% 20px, 100% calc(100% - 20px), calc(100% - 20px) 100%, 20px 100%, 0 calc(100% - 20px), 0 20px);
```

> 好处：可以使用任意类型的背景。
>
> 缺点：当内边距不够宽时，它会裁切掉文本



#### CSS4 corner-shape



### 梯形标签页

* 对伪元素使用3D变形（如果对自身元素使用3D变形，其内部元素不可逆转）

```css
.tab {
    position: relative;
    display: inline-block;
    padding: .5em 1em .35em;
    color: white;
}
.tab::before {
    content:''; /* 用伪元素生成一个矩形 */
    position: absolute;
    top: 0; right: 0; bottom: 0; left: 0;
    z-index: -1;
    background: #58a;
    /* 旋转后高度会变小，需适当放大 */
    transform: scaleY(1.3) perspective(.5em) rotateX(5deg);
    transform-origin: bottom; /* 旋转时底边固定住 */
}
```

> 缺点：斜边的角度依赖于元素的宽度。适用于宽度基本一致的元素



### 简单的饼图

#### 基于伪元素transform的解决方案

#### SVG解决方案



## 视觉效果

### 单侧投影

* 使用box-shadow的第四个长度参数，这个参数会根据你指定的值去扩大或（当指定负值时）缩小投影的尺寸。eg、一个-5px的扩展半径会把投影的宽度和高度各减少10px（每边各5px）
* 如果应用一个负的扩张半径，而它的值刚好等于模糊半径，那么投影的尺寸就会与投影所属元素的尺寸完全一致。除非用偏移量（前两个参数）来移动它，
  实现单侧投影如下：


```css
box-shadow: 0 5px 4px -4px black;
```



#### 邻边投影

#### 双侧投影

* 把单侧投影的技巧运用两次



### 不规则投影

问题：border-radius会忽略透明的部分。这类情况包括：

* 半透明图像、背景图像、或者border-image
* 点状、虚线或半透明的边框，但没有背景（或者当backgroud-clip不是border-box时）
* 对话气泡
* 切角形状
* 折角效果
* 菱形图片

解决方案：使用filter属性，指定drop-shadow滤镜

```css
filter: drop-shadow(2px 2px 10px rgba(0,0,0,.5));
```

> 可参考：https://developer.mozilla.org/en-US/docs/Web/CSS/filter



### 染色效果

#### 基于滤镜的方案

通过filter属性，参考上面链接



#### 基于混合模式的方案



### 毛玻璃效果

```css
body, main::before {
    background: url("trigger.jpg") 0 / cover fixed;
}
main {
    position: relative;
    background: hsla(0, 0%, 100%, .3);
    overflow: hidden;
}
main::before {
    content: '';
    position: absolute;
    top: 0; right: 0; bottom: 0; left: 0;
    filter: blur(20px);
    margin: -30px; /* 修复边缘模糊消退的问题 */
}
```



### 折角效果



## 字体排印

### 连字符断行

使用新属性hyphens，接收参数：none | manual | auto

* manual：默认值，可以手工插入软连字符，来实现断词折行的效果
* none：禁用手工插入软连字符
* auto：自动帮你插入连字符，仍可以手工插入软连字符（`&shy;` )来辅助浏览器断词。需要在HTML标签的lang属性中指定合适的语言



### 插入断行

```html
<dl>
    <dt>Name:</dt>
    <dd>Lea Verou</dd>
    
    <dt>Email:</dt>
    <dd>lea@verou.me</dd>
    
    <dt>Location：</dt>
    <dd>Earc</dd>
</dl>
```

```css
dd + dt::before {
    content: '\A', /* 相当于换行符 */
    white-space: pre;
}
dd + dd::before {
    content: ',',
    font-weight: normal;
}
```



### 文本行的斑马条纹

* 传统做法：使用:nth-child()实现
* 可以对整个元素设置统一的背景图像，一次性加上所有的斑马条纹

```css
padding: .5em;
line-height: 1.5;
background: beige; /* 米色 */
background-size: auto 3em;
background-origin: content-box;
background-image: linear-gradient(rgba(0,0,0,.2) 50% transparent 0);
```



### 调整tab的宽度

通常使用`<pre>`和`<code>`元素来显示代码。通过`tab-size`属性设置tab宽度



### 连字

原有的font-varient被升级成了一个简写属性，由很多新的展开式属性组合而成。其中之一叫做`font-variant-ligatures`，用来控制连字符的开启和关闭。如果要启用所有可能的连字，需要同时制定这三个标识符

```css
font-variant-ligatures: common-ligatures discretionary-ligatures historical-ligatures;
```



### 华丽的&字符



### 自定义下划线

通过CSS渐变生成所需的图像



### 现实中的文字效果

#### 凸版印刷效果

#### 空心字效果

#### 文字外发光效果

几层重叠的text-shadow即可，不需要考虑偏移量，颜色也只需跟文字保持一致



#### 文字凸起效果

给文字添加一系列逐渐加深的`text-shadow`



### 环形文字



## 用户体验

### 选用合适的鼠标

* 提示禁用状态

  ```css
  :disable, [disable], [aria-disable="true"] {
      cursor: not-allowed;
  }
  ```

  ​

* 隐藏鼠标光标

  ```css
  cursor: url('transparent.gif'); /* 回退方案 */
  cursor: none;
  ```




### 扩大可点击区域

* 目标：扩张点击区域，但要**透明**的。可以为其设置一圈**透明边框**，因为鼠标对元素边框的交互也会触发鼠标事件

```css
border: 10px solid transparent;
background-clip: padding-box; /* 避免背景色扩张到边框 */
box-shadow: 0 0 0 1px rgba(0,0,0,.3) inset; /* 使用box-shadow模拟实线边框 */
```

如果要再设置按钮**外部投影**，可以使用**伪元素**，伪元素同样可以代替其宿主元素来响应鼠标交互。

```css
button {
    position: relative;
    /* [其余样式] */
}
button::before {
    content: '';
    position: absolute;
    top: -10px; right:-10px; bottom:-10px; left:-10px; /* 减去边框厚度 */
}
```



### 自定义复选框

没有多少样式能够对复选框起作用，不过可以基于复选框的勾选状态借助组合选择符来给其他元素设置样式。

可以借助label元素，当label元素与复选框关联之后，也可以起到触发开关的作用。可以为它添加生成性内容（伪元素），并基于复选框的状态为其设置样式，然后就可以把真正的复选框隐藏起来（并不能把它从tab键切换焦点的队列中完全删除），然后把生成性内容美化一番，用来顶替原来的复选框！

```html
<input type="checkbox" id="awesome">
<label for="awesome">Awesome!</label>
```

```css
/* 隐藏原来的复选框 */
input[type="checkbox"] { 
    position: absolute;
    clip: rect(0,0,0,0);
}

/* 未选中样式 */
input[type="checkbox"] + label::before { 
    content:'\a0'; /* 不换行空格 */
    display: inline-block;
    vertical-align: .2em;
    width: .8em;
    height: .8em;
    margin-right: .2em;
    border-radius: .2em;
    background: silver;
    text-indent: .15em;
   	line-height: .65;
}

/* 选中样式 */
input[type="checkbox"]:checked + label::before { 
    content:'\2713'; /* 对勾 */
    background: yellowgreen;
}

/* 聚焦样式 */
input[type="checkbox"]:focus + label::before {
    /* style content */
}

/* 禁用样式 */
input[type="checkbox"]:disabled + label::before {
    /* style content */
}
```

> 隐藏原始复选框不能使用`display:none`是因为那样会把它从键盘tab键切换焦点的队列中完全删除



#### 开关式按钮



### 通过阴影弱化背景

#### 传统方案

两个元素，遮罩层和目标元素。设置不同的z-index



#### 伪元素方案

把遮罩层交给元素自己的::before伪元素来实现，并设置不同的z-index。

缺点：伪元素无法绑定独立的JavaScript事件处理函数



#### box-shadow方案

```css
box-shadow: 0 0 0 50vmax rgba(0,0,0,.8);
```

> 无法分开指定水平和垂直方向上的扩张半径，所以此处最合适的视口单位是vmax。1vmax相当于1vw和1vh两者中的较大值。100vw等于整个视口的宽度，100vh就是视口的高度

缺点1：当滚动页面时，遮罩层的边缘就露出来了，除非给它加上`position:fixed`。

缺点2：遮罩层无法阻止用户的鼠标与页面的其他部分发生交互



#### backdrop方案

如果你想引导用户关注的元素就是一个模态的`<dialog>`元素（`<dialog>`元素可以由它的showModal()方法显示出来），那么根据浏览器的默认样式，它会自带一个遮罩层。借助`::backdrop`伪元素，这个原生的遮罩层也是可以设置样式的：

```css
dialog::backdrop {
    background: rgba(0,0,0,.8);
}
```

> 缺点：浏览器支持极为有限（18.3.1）



### 通过模糊弱化背景

通过滤镜

```html
<main>this is the main content</main>
<dialog>This is dialog</dialog>
<style>
    main {
        transition: .6s filter;
    }
    main.de-emphasized {
        filter: blur(5px) contrast(.8) brightness(.8);
    }
</style>
```

> 一旦滤镜不被支持，将没有任何回退方案。因此不妨使用前一篇中的box-shadow来实现阴影。
>
> 页面背景还可以通过scale()变形属性来产生缩小效果，从而进一步增强景深效果



### 滚动提示

效果类似如下：

![css-secrect-scroll.png](https://gitee.com/sluggish/music/raw/master/css-secrect-scroll.png)

原理：两层背景：一层用来生成那条阴影，另一层基本上就是**一个用来遮挡阴影的白色矩形**，其作用类似于遮罩层。生成阴影的那层背景将具有默认的background-attachment值（scroll），因为它总是保持在原位。把遮罩背景的`background-attachment`属性设置为local，这样它就会在我们滚动到最顶部时盖住阴影，在向下滚动时跟着滚动，从而露出阴影。

> 背景跟着内容滚动，将`background-attachment`设置为`local`

```css
background: linear-gradient(white 30%, transparent),
			radial-gradient(at 50% 0, rgba(0,0,0,.2), transparent 70%);
background-repeat: no-repeat;
background-size: 100% 50px, 100% 15px;
background-attachment: local, scroll;
```

> 上面只是一边，完整请参考：http://dabblet.com/gist/20205b5fcdd834461e80



### 交互式的图片对比控件



## 结构与布局

### 自适应内部元素

* 目标：希望width自适应其内容的宽度
* 方法：

```css
figure {
    width: min-content;
    margin: auto;
}
```

> `min-content`这个关键字将解析为这个容器内部最大的不可断行元素的宽度（即最宽的单词、图片或具有固定宽度的盒元素）



### 精确控制表格列宽

来自CSS2.1中的属性，叫做`table-layout`，它的默认值是auto，其行为模式被称为自动表格布局算法。不过它可以接受另一个值`fixed`，这个值的行为要明显可控一些。它把更多的控制权交给了网页开发者。

我们设置的（宽度）样式会直接起作用，而不仅仅作为一种提示；同时溢出行为（包括text-overflow）与其他元素行为也是一样的，因此表格的内容将只能影响表格行的高度了。



### 根据兄弟元素的数量来设置样式

* 问题1：在列表项的总数为4时，选中每一项

```css
li:first-child:nth-last-child(4),
li:first-child:nth-last-child(4)~li {
    /* 当列表项正好包含四项时，命中所有列表项 */
}
```

* 问题2：在列表项的总数是4或更多时选中所有列表项

```css
li:first-child:nth-last-child(n+4),
li:first-child:nth-last-child(n+4)~li {
    /* 当列表至少包含四项时，命中所有列表项 */
}
```

* 问题3：但列表中有4个或更少的列表项时，选中所有的列表项

```css
li:first-child:nth-last-child(-n+4),
li:first-child:nth-last-child(-n+4) ~ li {
    /* 当列表最多包含四项时，命中所有列表项 */
}
```



### 满幅的背景，定宽的内容

#### 传统做法

为每个区块准备两层元素：外层用来实现满幅的背景，内容用来实现定宽的内容



#### 单元素做法

```css
footer {
    padding: 1em; /* 向后兼容 */
    padding: 1em calc(50% - 450px); /* 此处说明内容宽度最大为900px */
    background: #333;
}
```

> 如果屏幕的宽度比内容的宽度还要窄，这个解决方案所产生的效果就是没有内边距。可以用媒体查询修复



### 垂直居中

#### 基于绝对定位的解决方案

1. 使用margin

```css
main {
    position: absolute;
    top: 50%;
    left: 50%;
    margin-top: -3em;
    margin-left: -9em;
    width: 18em;
    height: 6em;
}
```

2. 使用calc()函数

```css
main {
    position: absolute;
    top: calc(50% - 3em);
    left: calc(50% - 9em);
    width: 18em;
    height: 6em;
}
```

3. 使用translate

```css
main {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

>  缺点：
>
>  * 有时不能使用绝对定位，它对整个布局的影响太过强烈
>  * 如果需要居中的元素已经在高度上超过了视口，那它的顶部会被视口裁切掉
>  * 在某些浏览器，这个方法可能会导致元素的显示有一些模糊，因为元素可能被放置在半个像素上。这个问题可以用`transform-style: perserve-3d`来修复



#### 基于视口单位的解决方案

* margin的百分比是以父元素的宽度作为解析基准的。即使对于margin-top和margin-bottom也是如此

```css
main {
    width: 18em;
    padding: 1em 1.5em;
    margin: 50vh auto 0;
    transform: translateY(-50%);
}
```

> 只适用于在视口中居中的场景



#### 基于Flex的解决方案

```css
body {
    display: flex;
    min-height: 100vh;
    margin: 0;
}
main {
    margin: auto; /* main自身水平垂直居中 */
}
```

可以设置如下，让main中的文本也水平垂直居中

```css
main {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 18em;
    height:10em;
}
```



### 紧贴底部的页脚(sticky-footer)

#### 固定高度的解决方案

```css
/* 把header，main包在wrapper容器中，
	只需考虑页脚的高度 */
#wrapper {
    min-height: calc(100vh - 7em); /* 页脚高度为7em */
}
```

缺点：不仅要求我们确保页脚内的文本永远不会折行，而且每当我们改变页脚的尺寸时，都需要跟着跳转min-height值



#### flex解决方案

```css
body {
    display: flex;
    flex-flow: column;
    min-height: 100vh;
}

/* 页头和页脚的高度由其内部因素决定，而内容main的高度
	应该可以自动伸展并占满所有的可用空间 */
main {
    flex: 1;
}
```



## 过渡与动画

### 缓动效果



### 逐帧动画

* 把所有帧拼合到一张图片中。使用setp()调速函数，而不是基于贝塞尔曲线的调速函数。step()会根据你指定吧步进数量，把整个动画切分为多帧，而且整个动画会在帧与帧之间硬切，不会做任何处理

```css
@keyframes loader {
    to {background-position: -800px 0;}
}
.loader {
    width: 100px;height: 100px;
    background: url(img/loader.png) 0 0;
    animation: loader 1s infinite steps(8); /* 8帧 */
    
    /* 把文本隐藏起来 */
    text-indent: 200%;
    white-space: nowrap;
    overflow: hidden;
}
```

> steps()还可以接收可选的第二个参数，其值可以是start或end（默认值）。这个参数用于指定动画在每个循环周期的什么位置发生帧的切换动作



### 闪烁效果



### 打字动画

* 核心思路：让容器的宽度成为动画的主体：把所有文本包裹在这个容器中，然后让它的宽度从0开始以步进的方式、一个字一个字地扩张到它应有的宽度


```html
<h1>CSS is awesome!</h1>
<style>
    @keyframes typing {
        from: {width: 0}
    }
    @keyframes caret { /* 光标 */
        50% {border-color: currentColor;}
    }
    h1 {
        width: 15ch; /* 文本的宽度 */
        overflow: hidden;
        white-space: nowrap;
        animation: typing 6s steps(15),
            	   caret 1s steps(1) inifinite;
    }
</style>
```

> `ch`单位是CSS3引入的新单位，表示“0”字形的宽度。在等宽字体中，所有字形的宽度是一样的，因此取值就是字符的数量（这里是15）





### 状态平滑的动画

```css
@keyframes panoramic {
	to { background-position: 100% 0; }
}

.panoramic {
	width: 150px; height: 150px;
	background: url('http://c3.staticflickr.com/3/2671/3904743709_74bc76d5ac_b.jpg');
	background-size: auto 100%;	
	animation: panoramic 10s linear infinite alternate;
	animation-play-state: paused;
}

.panoramic:hover, .panoramic:focus {
	animation-play-state: running;
}
```



### 沿环形路径平移的动画
