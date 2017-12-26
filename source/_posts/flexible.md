---
title: 使用Flexible实现手淘H5页面的移动端适配
date: 2017-12-26 14:35:36
tags:
---

由于移动端特殊性，本文讲的是如何使用rem实现自适应，或叫rem响应式布局，通过使用一个脚本就可以rem自适应，不用再为各种设备宽度不同而烦恼如何实现自适应的问题。
<!-- more -->
## 开发流程

- 选择一种尺寸作为设计和开发基准

- 定义一套适配规则，自动适配剩下的两种尺寸(其实不仅这两种，你懂的)

- 特殊适配效果给出设计效果

设计师和前端开发协作过程中：设计师常选择iPhone6作为基准设计尺寸，交付给前端的设计尺寸是按750px * 1334px为准(高度会随着内容多少而改变)。前端开发人员通过一套适配规则自动适配到其他的尺寸。


## CSS单位rem

简单的理解，rem就是相对于根元素<html>的font-size来做计算。而我们的方案中使用rem单位，是能轻易的根据<html>的font-size计算出元素的盒模型大小。而这个特色对我们来说是特别的有益处。

## 了解我们使用的库flexible.js

- 手机淘宝从2014年中开始，全面推行flexible设计。什么叫flexible呢？其实flexible就是responsive的低端形态和基础。

- 手机淘宝用JS来动态写meta标签，手机淘宝的flexible方案是综合运用rem和px两种单位+js设置scale和html字体。

[GitHub地址](https://github.com/amfe/lib-flexible)

实现方法：通过js来调整html的字体大小，而在页面中的制作稿则统一使用rem这个单位来制作。

页面生成的html，注意flexible帮我们的页面添加了几处修改
![Alt text](./1.png)

假设我们的设计稿宽是640的，则html的字体大小则被设为64px.则相当于1rem=64px。


假如一个元素的宽是160px,在平时，我们可以采用百分比可以做到自适应，假如使用响应式的话，可能需要设置多个，比如在320px，输出80px，而在640px输出160px等。

而采用以上rem的方法，则只需要输出2.5rem就能实现统一，如下表格：

| 设备宽度        | 320px           | 360px  |414px  |640px  |
|:-------------:|:-------------:|:-----:|:-----:|:-----:|
|Html字体大小|	32px|	36px|	41.4px|	64px|
|实际输出|	1rem|	1rem|	1rem|	1rem|
|设计稿缩放大小|	80px|	90px|	103.5px|	160px|
|实际输出|	2.5rem|	2.5rem|	2.5rem|	2.5rem|

目前Flexible会将视觉稿分成100份（主要为了以后能更好的兼容vh和vw），而每一份被称为一个单位a。同时1rem单位被认定为10a。针对我们这份视觉稿可以计算出：

```bash
1a   = 7.5px
1rem = 75px 
```
那么我们这个示例的稿子就分成了10a，也就是整个宽度为10rem，<html>对应的font-size为75px

这样一来，对于视觉稿上的元素尺寸换算，只需要原始的px值除以rem基准值即可。例如此例视觉稿中的图片，其尺寸是176px * 176px,转换成为2.346667rem * 2.346667rem。

## 如何快速计算转换

使用Sass的同学，可以使用Sass的函数、混合宏这些功能来实现
```bash
@function px2em($px, $base-font-size: 16px) { 
    @if (unitless($px)) { 
        @warn "Assuming #{$px} to be in pixels, attempting to convert it into pixels for you"; 
        @return px2em($px + 0px); // That may fail. 
    } @else if (unit($px) == em) { 
        @return $px; 
    } 
    @return ($px / $base-font-size) * 1em; 
}

```

## 字号不使用rem

不希望文本在Retina屏幕下变小，另外，我们希望在大屏手机上看到更多文本，考虑文本还是使用px作为单位。

## 实例

[175star微信公众号](http://wx.user.175star.com)
[175fly官网](http://fb.175fly.com)  PC端用手机模式查看

参考文档：[https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html](https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html)


