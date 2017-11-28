---
title: flex弹性布局
date: 2017-11-24 16:35:01
tags:
---

## 一、简要介绍

css3最喜欢的新属性之一便是flex布局属性，用六个字概括便是简单、方便、快速。

flex（ flexible box：弹性布局盒模型）,是2009年w3c提出的一种可以简洁、快速弹性布局的属性。
主要思想是给予容器控制内部元素高度和宽度的能力。目前已得到以下浏览器支持：

<!-- more -->

![Alt text](./2.png)

其中在webkit内核的浏览器中使用时，必须加上-webkit-前缀

下图为flex的相关概念的示意图

![Alt text](./1.png)

使用flex布局的容器（flex container），它内部的元素自动成为flex项目（flex item）。
容器拥有两根隐形的轴，水平的主轴（main axis），和竖直的交叉轴。
主轴开始的位置，即主轴与右边框的交点，称为main start；主轴结束的位置称为main end；
交叉轴开始的位置称为cross start；交叉轴结束的位置称为cross end。
item按主轴或交叉轴排列，item在主轴方向上占据的宽度称为main size，在交叉轴方向上占据的宽度称为cross size。

此外，需注意使用flex容器内元素，即flex item的float，clear、vertical-align属性将失效。

## 二、属性总结表

![Alt text](./3.png)

## 三、容器属性详述

### 1.flex-direction

决定主轴的方向，即项目排列的方向，有四个可能的值：row(默认)|row-reverse|column|column-reverse

- row:主轴为水平方向，项目沿主轴从左至右排列

- column：主轴为竖直方向，项目沿主轴从上至下排列

- row-reverse：主轴水平，项目从右至左排列，与row反向

- column-reverse：主轴竖直，项目从下至上排列，与column反向

![Alt text](./5.png)

![Alt text](./4.png)

### 2、flex-wrap

默认情况下，item排列在一条线上，即主轴上，flex-wrap决定当排列不下时是否换行以及换行的方式，可能的值nowrap(默认)|wrap|wrap-reverse

- nowrap：自动缩小项目，不换行

- wrap：换行，且第一行在上方

- wrap-reverse：换行，第一行在下面

![Alt text](./6.png)

![Alt text](./7.png)

### 3、flex-flow

是flex-direction和flex-wrap的简写形式，如：row wrap|column wrap-reverse等。默认值为row nowrap，即横向排列 不换行。

### 4、justify-content

决定item在主轴上的对齐方式，可能的值有flex-start（默认），flex-end，center，space-between，space-around。当主轴沿水平方向时，具体含义为

- flex-start：左对齐

- flex-end：右对齐

- center：居中对齐

- space- between：两端对齐

- space-around：沿轴线均匀分布

![Alt text](./8.png)

### 5、align-items

决定了item在交叉轴上的对齐方式，可能的值有flex-start|flex-end|center|baseline|stretch，当主轴水平时，其具体含义为

- flex-start：顶端对齐

- flex-end：底部对齐

- center：竖直方向上居中对齐

- baseline：item第一行文字的底部对齐

- stretch：当item未设置高度时，item将和容器等高对齐

![Alt text](./9.png)

### 6、align-content

该属性定义了当有多根主轴时，即item不止一行时，多行在交叉轴轴上的对齐方式。
注意当有多行时，定义了align-content后，align-items属性将失效。align-content可能值含义如下（假设主轴为水平方向）：

- flex-start：左对齐

- flex-end：右对齐

- center：居中对齐

- space- between：两端对齐

- space-around：沿轴线均匀分布

- stretch：各行将根据其flex-grow值伸展以充分占据剩余空间

![Alt text](./10.png)

![Alt text](./11.png)


## 四、flex item属性详述

item的属性在item的style中设置。item共有如下六种属性

### 1、order:order的值是整数，默认为0，整数越小，item排列越靠前，如下图所示代码如下

```bash
<div class="wrap">
    <div class="div" style="order:4"><h2>item 1</h2></div>
    <div class="div" style="order:2"><h2>item 2</h2></div>
    <div class="div" style="order:3"><h2>item 3</h2></div>
    <div class="div" style="order:1"><h2>item 4</h2></div>
</div>
```
![Alt text](./12.png)

### 2、flex-grow

定义了当flex容器有多余空间时，item是否放大。默认值为0，即当有多余空间时也不放大；可能的值为整数，表示不同item的放大比例，如

```bash
<div class="wrap">
    <div class="div" style="flex-grow:1"><h2>item 1</h2></div>
    <div class="div" style="flex-grow:2"><h2>item 2</h2></div>
    <div class="div" style="flex-grow:3"><h2>item 3</h2></div>
</div>
```
即当有多余空间时item1、item2、和item3以1：2：3的比例放大。

### 3、flex-shrink

定义了当容器空间不足时，item是否缩小。默认值为1，表示当空间不足时，item自动缩小，其可能的值为整数，表示不同item的缩小比例。flex-grow

### 4、flex-basis

表示项目在主轴上占据的空间，默认值为auto。如下代码

```bash
<div class="wrap">
    <div class="div" style="flex-basis:80px"><h2>item 1</h2></div>
    <div class="div" style="flex-basis:160px"><h2>item 2</h2></div>
    <div class="div" style="flex-basis:240px"><h2>item 3</h2></div>
</div>
```
![Alt text](./13.png)

### 5、flex

flex属性是flex-grow、flex-shrink和flex-basis三属性的简写总和。

### 6、align-self

align-self属性允许item有自己独特的在交叉轴上的对齐方式，它有六个可能的值。默认值为auto

- auto：和父元素align-self的值一致

- flex-start：顶端对齐

- flex-end：底部对齐

- center：竖直方向上居中对齐

- baseline：item第一行文字的底部对齐

- stretch：当item未设置高度时，item将和容器等高对齐

![Alt text](./14.png)

参考文档：[Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
&emsp;&emsp;&emsp;&emsp;&emsp;[Flex 布局教程：实例篇](http://www.ruanyifeng.com/blog/2015/07/flex-examples.html)



