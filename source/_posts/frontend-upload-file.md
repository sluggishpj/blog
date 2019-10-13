---
title: 前端文件上传
date: 2017-11-29 00:08:05
tags:
- 文件上传
- 图片上传预览
categories:
- 综合知识点
---

## 目的

简要介绍前端文件上传方式及上传接口

<!-- more -->

## 文件上传方式一：form表单提交

- 将`method`属性设置为`POST`，因为文件内容不能放入URL参数中。
- 将`enctype`的值设置为`multipart/form-data`，因为数据将被分成多个部分，每个文件分别对应一个文件以及表单正文中包含的文本数据(如果文本也输入到表单中)。
- 包含一个或多个File picker小部件，允许用户选择将要上传的文件。

```html
<form action="/testMultiple" method="post" enctype="multipart/form-data">
    <p>
        <input type="text" name="name">
    </p>
    <p>
        <input type="file" name="myfiles" multiple>
    </p>
    <p>
        <input type="submit" value="提交">
    </p>
</form>
```

> input 中的`multiple`表示可以同时上传多个文件，去掉则每次只能上传一个。
>
> 点击提交后页面发生跳转。可通过如下设置：设置form的target属性为页面某个隐藏iframe的name。

```html
<!-- 提交form表单不发生跳转 -->
<iframe width="0" height="0" border="0" name="dummyframe" id="myDummyframe"></iframe>

<form action="submitscript.php" target="dummyframe">
    <!-- form body here -->
</form>

<script type="text/javascript">
    // 获取后台返回的消息，IE测试不行。。。
    var myDummyframe = document.getElementById('myDummyframe');
    myDummyframe.onload = function() {
        console.log(myDummyframe.contentDocument.documentElement.innerText);
    }
</script>
```

> 参考链接1：[javascript - How to submit html form without redirection? - Stack Overflow](https://stackoverflow.com/questions/25983603/how-to-submit-html-form-without-redirection)
>
> 参考链接2：[Using files from web applications - Web API 接口 | MDN](https://developer.mozilla.org/zh-CN/docs/Learn/HTML/Forms/Sending_and_retrieving_form_data)
>



## 文件上传方法二：ajax提交FormData

```js
// 使用了jQuery

/**
 * @param  {FormData} myFormData formData格式，用于存放文件
 * @param  {Function} cb 回调函数
 * @return none
 */
function sendMultiple(myFormData, cb) {
    $.ajax({
        url: '/testMultipleFileInput',
        type: 'POST',
        data: myFormData,
        cache: false,
        dataType: 'json',
        processData: false, // 因为data值是FormData对象，不需要对数据做处理
        contentType: false, 
        success: function(res, textStatus, jqXHR) {
            typeof cb == 'function' && cb(res);
        },
        error: function(xhr, textStatus, errThrow) {
			//...
        }
    });
}

// 使用方法，此处是一旦选择了文件就上传，可根据需要修改
// $fileInput是type为'file'的input引用
var $filesInput = $('.filesInput');
$filesInput.on('change', function() {
    myFormData = new FormData();
    var fileList = this.files,
        myfile;
  
    for (var i = 0, len = fileList.length; i < len; i++) {
        myfile = fileList[i];
        console.log('文件名: ' + myfile.name + ' 大小:' + (myfile.size / 1024).toFixed(0) + 'KB 类型: ' + myfile.type );
        myFormData.append('myfiles', myfile); // 将文件都append进formData中
    }
	
  	// 上传
	sendMultiple(myFormData, function(res) {
   		console.log('返回结果', res);
	});
});
```

> 参考链接：[FormData - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/FormData)



## 使用自定义接口打开文件选择器

原因：file input元素有点丑

### 方法一 通过click()方法隐藏file input元素

```html
<input type="file" id="fileElem" multiple accept="image/*" style="display:none">
<a href="#" id="fileSelect">Select some files</a>

<script type="text/javascript">
var fileSelect = document.getElementById("fileSelect"),
    fileElem = document.getElementById("fileElem");

fileSelect.addEventListener("click", function(e) {
    if (fileElem) {
        fileElem.click();
    }
    e.preventDefault(); // prevent navigation to "#"
}, false);
</script>
```



### 方法二 使用label元素来触发一个隐藏的input元素对应的事件

```html
<input type="file" id="fileElem" multiple accept="image/*" style="display:none" onchange="handleFiles(this.files)">
<label for="fileElem">Select some files</label>
```



### 方法三 使用drag和drap来选择文件

```html
<div id="dropbox">拖拽文件到此处</div>
<script type="text/javascript">
var dropbox;
dropbox = document.getElementById("dropbox");
dropbox.addEventListener("dragenter", dragenter, false);
dropbox.addEventListener("dragover", dragover, false);
dropbox.addEventListener("drop", drop, false);

// 我们其实并不需要对dragenter and dragover 事件进行处理，所以这些函数都可以很简单。他们只需要包括禁止事件传播和阻止默认事件
function dragenter(e) {
    e.stopPropagation();
    e.preventDefault();
}

function dragover(e) {
    e.stopPropagation();
    e.preventDefault();
}

function drop(e) {
    e.stopPropagation();
    e.preventDefault();

    var dt = e.dataTransfer;
    var files = dt.files;

    handleFiles(files);
}
```



## 预览用户选择的图片

### 方法一 使用FileReader

```js
function handleFiles(files) {
  for (var i = 0; i < files.length; i++) {
    var file = files[i];
    var imageType = /^image\//;
    
    if (!imageType.test(file.type)) {
      continue;
    }
    
    var img = document.createElement("img");
    img.classList.add("obj");
    img.file = file;
    preview.appendChild(img); // 此处的preview是用于放图片的容器
    
    var reader = new FileReader();
    reader.onload = (function(aImg) { return function(e) { aImg.src = e.target.result; }; })(img);
    reader.readAsDataURL(file);
  }
}
```

> 为了在DOM树中更容易地找到他们，每个图片元素都被添加了一个名为obj的class
>
> 给每个图片添加了file属性使它具有File，这样做可以让我们拿到稍后需要实际上传的图片



### 方法二 使用 object URLs

```js
window.URL = window.URL || window.webkitURL;

function handleFiles(files) {
    if (!files.length) {
        return;
    } else {
        for (var i = 0; i < files.length; i++) {
            var img = document.createElement("img");
            img.src = window.URL.createObjectURL(files[i]); // 创建
            img.onload = function() {
                window.URL.revokeObjectURL(this.src); // 释放
            };
            preview.appendChild(img); // 此处的preview是用于放图片的容器
        }
    }
}
```

> 使用`window.URL.createObjectURL()`创建 blob URL
>
> 当图片load完后，obj URL不再需要，使用`window.URL.revokeObjectURL()` 方法释放掉

> 参考链接：https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications



