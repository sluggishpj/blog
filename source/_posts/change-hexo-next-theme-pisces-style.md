---
title: 修改NexT Pisces主题样式
date: 2017-10-14 12:48:10
tags:
  - NexT主题
categories: 
  - tools
  - hexo
---

## 修改nexT Pisces主题内容区宽度
### 问题
默认的宽度觉得有点窄，想改宽一点

<!-- more -->
### 方法
在[官网](http://theme-next.iissnan.com/faqs.html)的指引下找到了[方案](https://github.com/iissnan/hexo-theme-next/issues/759#issuecomment-202242848)
在测试后发现布局乱了，蓝瘦。。。
> 测试版本为，hexo:v3.3.8，hexo-theme-next:v5.1.2
> 上面的方案如下：

``` css
/*对于 Pisces Scheme，需要同时修改 header 的宽度、.main-inner 的宽度以及 .content-wrap 的宽度。
例如，使用百分比（Pisces 的布局定义在 source/css/_schemes/Picses/_layout.styl 中）：*/

header{ width: 90%; }
.container .main-inner { width: 90%; }
.content-wrap { width: calc(100% - 260px); }

```

### 改进方法
没办法，手动修改样式。。。
以下代码受上面方案启发，经过试验，在`source/css/_schemes/Picses/_layout.styl`文件末尾添加如下代码。

``` styl
// 以下为新增代码！！
// 白色区域的最大宽度
$white_max_width = 1200px

header{ 
  width: 90% !important;
  max-width: $white_max_width;
}
header.post-header {
  width: auto !important;
}
.container .main-inner {
  width: 90%;
  max-width: $white_max_width;
}
.content-wrap { width: calc(100% - 260px); }


.header {
  +tablet() {
    width: auto !important;
  }
  +mobile() {
    width: auto !important;
  }
}


.container .main-inner {
  +tablet() {
    width: auto !important;
  }
  +mobile() {
    width: auto !important;
  }
}

.content-wrap {
  +tablet() {
    width: 100% !important;
  }
  +mobile() {
    // 为了在手机访问时，内边距不至于太大
    padding: 0 !important;
    width: 100% !important;
  }
}
```
> 修改样式后，需要 hexo clean 后再部署才会生效



## 结束
{% cq %} Till I reach the end, then I'll start again 
《Try Everything》
{% endcq %}