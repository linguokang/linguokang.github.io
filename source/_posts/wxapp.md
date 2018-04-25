---
title: 微信小程序框架——wepy
date: 2017-12-23 11:26:09
tags:
    - wepy
---
## 一、小程序开发有哪些痛点

<!-- more -->

1） 频繁调用 setData及 setData过程中页面跳闪

2） 组件化支持能力太弱(几乎没有)

3） 不能使用 less、jade 等

4） 无法使用 NPM 包及 ES 高级语法

5） request 并发次数限制

6） 一个页面对应4个文件，看的眼花缭乱

## 二、什么是以及为什么使用 wepy

如果可以，请先看一下[官方文档](https://tencent.github.io/wepy/)。因为关于wepy的一切，文档肯定比我说的清楚。

现在，我们这样写小程序，是不是很嗨皮呢

```bash
import wepy from 'wepy';
        
// 通过继承自wepy.page的类创建页面逻辑
export default class Index extends wepy.page {
    //可用于页面模板绑定的数据
    data = {
      motto: 'Hello World',
      userInfo: {}
    };
     
    //事件处理函数(集中保存在methods对象中)
    methods = {
      bindViewTap () {
        console.log('button clicked');
      }
    };
        
    //页面的生命周期函数
    onLoad () {
      console.log('onLoad');
    };
}
```

## 三、wepy 使用注意事项

由于类Vue的开发风格，使得开发效率变高了，一个页面对应一个文件，后期维护变得更简单了。

3.1 数据管理问题

组件间的数据可以使用框架提供的 $emit $broast等方法，但页面之间的数据，就需要我们手动管理，非常麻烦且易于出错。
就目前来看，主要有两种方案可以选择：

- Gloabal 对象或Storage
  使用微信提供的getApp()方法，可以在页面间随意访问和改写这个对象。

- Store
  创建一个存储类来管理数据，本质上和Storage类似，但不能直接对数据进行修改而通过action触发数据变更。

## 四、针对原生API进行优化

对小程序原生API进行promise处理，同时修复了一些原生API的缺陷，比如：wx.request的并发问题等。

原生代码：

```bash
onLoad = function () {
    var self = this;
    wx.login({
        success: function (data) {
            wx.getUserInfo({
                success: function (userinfo) {
                    self.setData({userInfo: userinfo});
                }
            });
        }
    });
}
```
基于WePY的代码：
```bash
import wepy from 'wepy';

async onLoad() {
    await wepy.login();
    this.userInfo = await wepy.getUserInfo();
}
```

## 五、项目实战

基于微信使用wepy开发的仿「ONE · 一个」的微信小程序

[wechat-app-one](https://github.com/linguokang/wechat-app-one)
