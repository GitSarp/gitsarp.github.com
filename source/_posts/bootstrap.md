---
title: bootstrap
date: 2018-07-01 21:31:22
tags: Bootstrap
categories: 前端
---
#### 1.栅格系统
总共12列，col-xs-6 col-sm-3 col-md（超小屏幕，小，中等）占的列宽
```
<div class="page-header">
    <h1>案例:Responsive column resets</h1>
</div>
<div class="row">
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3(通过调整浏览器的宽度或在手机上即可查看这些案例的实际效果。)</div>
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
  <div class="clearfix visible-xs"></div>
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
  <div class="col-xs-6 col-sm-3">.col-xs-6 .col-sm-3</div>
</div>
```
超小效果：
![image.png](http://upload-images.jianshu.io/upload_images/3341325-c3fdc0bc769e437a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
不加<div class="clearfix visible-xs"></div>，超小效果：
![image.png](http://upload-images.jianshu.io/upload_images/3341325-24143326d1bc1cd7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 2.列偏移
```
<div class="page-header">
    <h1>案例:列偏移</h1>
</div>
<div class="row">
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4 col-md-offset-4">.col-md-4 .col-md-offset-4</div>
</div>
<div class="row">
  <div class="col-md-3 col-md-offset-3">.col-md-3 .col-md-offset-3</div>
  <div class="col-md-3 col-md-offset-3">.col-md-3 .col-md-offset-3</div>
</div>
<div class="row">
  <div class="col-md-6 col-md-offset-3">.col-md-6 .col-md-offset-3</div>
</div>
```
效果：
![image.png](http://upload-images.jianshu.io/upload_images/3341325-7448798212e0e9dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
