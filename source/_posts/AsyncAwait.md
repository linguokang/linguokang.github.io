---
title: 异步解决方案-ES7的Async/Await
date: 2017-11-23 11:35:56
tags:
---

## async 和 await 在干什么
任意一个名称都是有意义的，先从字面意思来理解。async 是“异步”的简写，而 await 可以认为是 async wait 的简写。

所以应该很好理解 async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成。

另外还有一个很有意思的语法规定，await 只能出现在 async 函数中。

<!-- more -->

## 第一个例子
Async/Await应该是目前最简单的异步方案了，首先来看个例子。

这里我们要实现一个暂停功能，输入N毫秒，则停顿N毫秒后才继续往下执行。

``` bash
var sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            resolve();
        }, time);
    })
};

var start = async function () {
    // 在这里使用起来就像同步代码那样直观
    console.log('start');
    await sleep(3000);
    console.log('end');
};

start();
```

控制台先输出start，稍等3秒后，输出了end。

## 基本规则

1.async 表示这是一个async函数，await只能用在这个函数里面。

2.await 表示在这里等待promise返回结果了，再继续执行。

3.await 后面跟着的应该是一个promise对象（当然，其他返回值也没关系，只是会立即执行，不过那样就没有意义了…）

## 获得返回值

await等待的虽然是promise对象，但不必写.then(..)，直接可以得到返回值。

``` bash

var sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            // 返回 ‘ok’
            resolve('ok');
        }, time);
    })
};

var start = async function () {
    let result = await sleep(3000);
    console.log(result); // 收到 ‘ok’
};

```
## 捕捉错误

既然.then(..)不用写了，那么.catch(..)也不用写，可以直接用标准的try catch语法捕捉错误。

``` bash

var sleep = function (time) {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            // 模拟出错了，返回 ‘error’
            reject('error');
        }, time);
    })
};

var start = async function () {
    try {
        console.log('start');
        await sleep(3000); // 这里得到了一个返回错误
        
        // 所以以下代码不会被执行了
        console.log('end');
    } catch (err) {
        console.log(err); // 这里捕捉到错误 `error`
    }
};

```
