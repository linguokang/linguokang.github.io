---
title: axios cancelToken原理
date: 2021-02-22 00:44:10
tags:
     - axios
---

### 前言

最近看了axios源码，里面的cancelToken有意思，以免忘记，写一篇博客记录一下。

<!-- more -->

### 用法

1.使用 CancelToken.source 工厂方法创建 cancel token

```javascript
// CancelToken是一个构造函数，用于创建一个cancelToken实例对象
// cancelToken实例对象包含了一个promise属性，值为可以触发取消请求的一个promise
const CancelToken = axios.CancelToken;
// 执行source()得到的是一个包含了cancelToken对象和一个取消函数cancel()的对象
// 即 source = {token: cancelToken对象, cancel: 取消函数}
const source = CancelToken.source();

// 在请求的配置中配置cancelToken，那么这个请求就有了可以取消请求的功能
axios.get('/data', {
  cancelToken: source.token
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
     // 处理错误
  }
});

```

2.通过传递一个 executor 函数到 CancelToken 的构造函数来创建 cancel token

```javascript
const CancelToken = axios.CancelToken;
let cancel;

axios.get('/data', {
  cancelToken: new CancelToken(function executor(c) {
    // executor 函数接收一个 cancel 函数作为参数
    // 把cancel函数传递给外面，使得外面能控制执行取消请求
    cancel = c;
  })
});

// cancel the request
cancel();

```

### 源码


```javascript
// axios/lib/cancel/CancelToken.js

'use strict';

var Cancel = require('./Cancel');

function CancelToken(executor) {
    if (typeof executor !== 'function') {
        throw new TypeError('executor must be a function.');
    }
    /**
    * 定义一个将来能执行取消请求的promise对象，当这个promise的状态为完成时(fullfilled),
    * 就会触发取消请求的操作（执行then函数）。而执行resolve就能将promise的状态置为完成状态。
    * 这里把resolve赋值给resolvePromise，就是为了在这个promise外能执行resolve而改变这个promise的状态
    * 注意这个promise对象被赋值给CancelToken实例的属性promise，将来定义then函数就是通过这个属性得到promise
    */
    var resolvePromise;
    this.promise = new Promise(function promiseExecutor(resolve) {
        resolvePromise = resolve;
    });
    /**
    * 将CancelToken实例赋值给token
    * 执行executor函数，将cancel方法传入executor，
    * cancel方法可调用resolvePromise方法，即触发取消请求的操作
    */
    var token = this;
    executor(function cancel(message) {
        if (token.reason) {
            // 取消已响应 返回
            return;
        }
        token.reason = new Cancel(message);
        // 这里执行的就是promise的resolve方法，改变状态
        resolvePromise(token.reason);
  });
}

CancelToken.prototype.throwIfRequested = function throwIfRequested() {
    if (this.reason) {
        throw this.reason;
    }
};

// 这里可以看清楚source函数的真面目
CancelToken.source = function source() {
    var cancel;
    var token = new CancelToken(function executor(c) {
        // c 就是CancelToken中给executor传入的cancel方法
        cancel = c;
    });
    return {
        token: token,
        cancel: cancel
    };
};

module.exports = CancelToken;

```

### CancelToken

CancelToken是一个构造函数，通过new CancelToken()得到的是一个实例对象，它只有一个属性promise, 它的值是一个能触发取消请求的Promise对象。

```javascript
token = new CancelToken(executor function) ===> { promise: Promise对象 }

```
执行CancelToken函数做了两件事：

创建一个Promise对象，且将这个对象赋值给promise属性，其resolve参数被暴露出来以备外部引用。 执行executor函数，将内部定义的cancel函数作为参数传递给executor


```javascript
var token = this;
var cancel = function (message) {
    if (token.reason) {
        // 取消已响应 返回
        return;
    }
    token.reason = new Cancel(message);
    // 这里执行的就是promise的resolve方法，改变状态
    resolvePromise(token.reason);
}
executor(cancel);

```
当执行

```javascript
let cancel
token = new CancelToken(function executor(c) {
    cancel = c;
});

```
token值为{promise: Promise对象} cancel就是CanelToken内部的那个cancel函数。

### CancelToken.source

CancelToken.source是一个函数，通过源码可以看到，执行const source = CancelToken.source(),得到的是一个对象：
```javascript
return {
    token: token,
    cancel: cancel
};

```
包含一个token对象，即CancelToken实例对象，和一个cancel函数。因此CancelToken.source()函数的作用是生成token对象和取得cancel函数。token对象是用于配置给axios请求中的cancelToken属性，cancel函数是将来触发取消请求的函数。

### 如何执行取消请求


```javascript
// axios/lib/adapters/xhr.js

// 创建XHR对象
var request = new XMLHttpRequest()
// 模拟当前ajax请求
request.open(config.method.toUpperCase(), buildURL(config.url, config.params, config.paramsSerializer), true)

// 定义取消请求promise对象的then函数
if (config.cancelToken) { // 如果配置了cancelToken属性
    // 当promise为完成态时，这个then函数执行，即执行取消请求
    config.cancelToken.promise.then(function onCanceled(cancel) {
        if (!request) {
            return;
        }
        // 取消ajax请求
        request.abort();
        reject(cancel);
        // Clean up request
        request = null;
    });
}

```
这是最终执行ajax请求的地方。
我们前面配置了cancelToken = new CancelToken(executor),得到对象{promise: 取消请求的Promise对象}，因此上面cancelToken.promise就是这个promise。
当我们执行了内部暴露出的cancel方法，promise被reslove转为fullfilled状态，进入then回调中，
调用xhr的abort()取消请求。
