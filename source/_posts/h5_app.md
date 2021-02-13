---
title: 【阅读笔记】h5 与原生 app 交互的原理
date: 2021-02-13 16:14:10
tags:
     - 原生交互
     - 阅读笔记
---

h5 与原生 app 的交互，本质上说，就是两种调用：
1. app 调用 h5 的代码
2. h5 调用 app 的代码

<!-- more -->

### app 调用 h5 的代码

因为 app 是宿主，可以直接访问 h5，所以这种调用比较简单，就是在 h5 中曝露一些全局对象（包括方法），然后在原生 app 中调用这些对象。

![Alt text](./h5_app1.png)

### h5 调用 app 的代码

因为 h5 不能直接访问宿主 app，所以这种调用就相对复杂一点。

有两种方式：

#### 由 app 向 h5 注入一个全局 js 对象，然后在 h5 直接访问这个对象

这种方式沟通机制简单，比较好理解，并且对于 h5 来说，没有新的东西，所以是比较推荐的一种方式。

![Alt text](./h5_app2.png)


#### 由 h5 发起一个自定义协议请求，app 拦截这个请求后，再由 app 调用 h5 中的回调函数

这种方式要稍复杂一点，因为需要自定义协议，这对很多前端开发者来说是比较新的东西

大致需要以下几个步骤：
1. 由 app 自定义协议，比如 sdk://action?params
2. 在 h5 定义好回调函数，比如 window.bridge = {getDouble: value => {}, getTriple: value => {}}
3. 由 h5 发起一个自定义协议请求，比如 location.href = 'sdk://double?value=10'
4. app 拦截这个请求后，进行相应的操作，获取返回值
5. 由 app 调用 h5 中的回调函数，比如 window.bridge.getDouble(20);

![Alt text](./h5_app3.png)
