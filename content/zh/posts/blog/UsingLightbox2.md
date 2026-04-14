---
title: 五分钟搭建个人博客：使用lighbox预览图片
date: 2024-06-03T06:47:40.000Z
draft: false
description: '如何使用lightbox2为基于Hugo和PaperMod主题的个人博客添加图片预览功能。'
---


前阵子来了趟西北环线旅行，写了几天游记，并附上几张图，但图片大小不一，也不能占太多空间，就想着要是能有那种点击图片放大查看的功能就好了，找了一圈没找到hugo PaperMod有类似的设置。

今天就趁着有点儿时间，就把这个能力补充上。查询了下类似的脚本库，决定使用lightbox2，感觉一下子回到了刚接触前端写jQuery的时候，也用了这个库，那时候还没有第二版呢哈哈。

闲话不多说，接下来看看怎么接入这个功能吧。

### 接入lightbox2
打开[qs](https://lokeshdhakar.com/projects/lightbox2/#getting-started)。
1. [下载](https://github.com/lokesh/lightbox2/releases) 资源文件。
1. 让打开/example

```html
  <link rel="stylesheet" href="../dist/css/lightbox.min.css">
	<!-- ... -->
  <script src="../dist/js/lightbox-plus-jquery.min.js"></script>

  <!-- <script src="../bower_components/jquery/dist/jquery.js"></script>
  <script src="../src/js/lightbox.js"></script> -->
```
3. 按照example引入css和js文件
4. 额外步骤：之前在[使用本地图片](/posts/blog/moreusage/#%E4%BD%BF%E7%94%A8%E6%9C%AC%E5%9C%B0%E5%9B%BE%E7%89%87)处理过静态资源，这次直接把解压的`dist`移动到`static`下。
5. 额外步骤：之前还有[自定义header](/posts/blog/supportdropdownsubmenu/)的时候，刚好把css和js在这里引入了。
6. 确保jQuery也被引入，咱们使用了`lightbox-plus-jquery`这个文件，就可以，不然会报错类似$ undefined之类的。因为这个依赖Dom元素，需要解析完成再执行，刚好([script中的defer和async属性](/posts/job/loadjsdeferandasync/))添加defer属性。
7. 确保`dist/images`被引入，整个dist都过来了，也就确保了。
8. 最后就是lightbox的使用方式了。

**当然如果你dist被ignore了，别忘记改个名字！**

#### lightbox使用方式
##### 单图片
```html
<a href="images/image-1.jpg" data-lightbox="image-1" data-title="My caption">Image #1</a>
```
##### 图片组
```html
<a href="images/image-2.jpg" data-lightbox="roadtrip">Image #2</a>
<a href="images/image-3.jpg" data-lightbox="roadtrip">Image #3</a>
<a href="images/image-4.jpg" data-lightbox="roadtrip">Image #4</a>
```
##### [options 配置](https://lokeshdhakar.com/projects/lightbox2/#options)
配置可以写到js文件里面，同样使用defer引入。
```html
<script>
    lightbox.option({
      'resizeDuration': 200,
      'wrapAround': true
    })
</script>
```

### 最后
更多的lightbox使用可以查看文档，这里就不做展开了。今天用简短的时间，给博客接入了图片预览功能，这个接入流程，希望对您有帮助/doge。

### example
<a class="example-image-link" href="http://lokeshdhakar.com/projects/lightbox2/images/image-6.jpg" data-lightbox="example-set" data-title="Click anywhere outside the image or the X to the right to close."><img class="example-image" src="http://lokeshdhakar.com/projects/lightbox2/images/thumb-6.jpg" alt=""></a>
