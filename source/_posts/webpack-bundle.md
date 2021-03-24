---
title: webpack 打包后 JS 的分析
date: 2021-02-24 16:35:56
tags:
  - 阅读笔记
  - webpack
---

webpack 是我们现阶段要掌握的重要的打包工具之一，我们知道 webpack 会递归的构建依赖关系图，其中包含应用程序的每个模块，然后将这些模块打包成一个或者多个 bundle。

<!-- more -->

那么 webpack 打包后的代码是怎样的呢？是怎么将各个 bundle 连接在一起的？模块与模块之间的关系是怎么处理的？

## 简单配置

既然需要用到 webpack，还是需要简单配置一下的，这里就简单贴一下代码，首先是 webpack.config.js:

```javascript
const path = require('path');
const webpack = require('webpack');
//用于插入html模板
const HtmlWebpackPlugin = require('html-webpack-plugin');
//清除输出目录，免得每次手动删除
const CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
  entry: {
    index: path.join(__dirname, 'index.js')
  },
  output: {
    path: path.join(__dirname, '/dist'),
    filename: 'js/[name].[chunkhash:4].js'
  },
  module: {},
  plugins: [
    new CleanWebpackPlugin(['dist']),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html'
    }),
    //持久化moduleId，主要是为了之后研究加载代码好看一点。
    new webpack.HashedModuleIdsPlugin(),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest'
    })
  ]
};
```

之后是两个简单的 js 文件：

```javascript
// test.js
const str = 'test is loaded';
module.exports = str;

// index.js
const test = require('./src/js/test');
console.log(test);
```


## 从 index.js 开始看代码

先从打包后的 index.html 文件看看两个 JS 文件的加载顺序：

```javascript
<body>
	<script type="text/javascript" src="js/manifest.2730.js"></script>
	<script type="text/javascript" src="js/index.5f4f.js"></script>
</body>
```

可以看到，打包后 js 文件的加载顺序是先 manifest.js，之后才是 index.js，按理说应该先看 manifest.js 的内容的。然而这里先卖个关子，我们先看看 index.js 的内容是什么，这样可以带着问题去了解 manifest.js,也就是主流程的逻辑到底是怎样的，为何能做到模块化。


```javascript
// index.js
webpackJsonp([0], {
  "JkW7": (function(module, exports, __webpack_require__) {
    const test = __webpack_require__("zFrx");
    console.log(test);
  }),
  "zFrx": (function(module, exports) {
    const str = 'test is loaded';
    module.exports = str;
  })
}, ["JkW7"]);
```

删去各种奇怪的注释后剩下这么点内容，首先应该关注到的是 webpackJsonp 这个函数，可以看见是不在任何命名空间下的，也就是 manifest.js 应该定义了一个挂在 window 下的全局函数，index.js 往这个函数传入三个参数并调用。

第一个参数是数组，现在暂时还不清楚这个数组有什么作用。

第二个参数是一个对象，对象内都是方法，这些方法看起来至少接受两个参数（名为 zFrx 的方法只有两个形参）。看一眼这两个方法的内部，其实看见了十分熟悉的东西， module.exports，尽管看不见 require， 但有一个样子类似的 __webpack_require__，这两个应该是模块化的关键，先记下这两个函数。

第三个参数也是一个数组，也不清楚是有何作用的，但我们观察到它的值是 JkW7，与参数2中的某个方法的键是一致的，这可能存在某种逻辑关联。
至此，index.js 的内容算是过了一遍，接下来应当带着问题在 manifest.js 中寻找答案。


## manifest.js 代码阅读
```javascript
(function(modules) {
  window["webpackJsonp"] = function webpackJsonpCallback(chunkIds, moreModules, executeModules) {
    var moduleId, result;
    for (moduleId in moreModules) {
      if (Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
        modules[moduleId] = moreModules[moduleId];
      }
    }
    if (executeModules) {
      for (i = 0; i < executeModules.length; i++) {
        result = __webpack_require__(executeModules[i]);
      }
    }
    return result;
  };
  var installedModules = {};

  function __webpack_require__(moduleId) {
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = installedModules[moduleId] = {
      exports: {}
    };
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    return module.exports;
  }
})([]);
```

首先应该看到的是，manifest.js 内部是一个 IIFE，就是自执行函数咯，这个函数会接受一个空数组作为参数，该数组被命名为 modules。之后看到我们在 index.js 中的猜想，果然在 window 上挂了一个名为 webpackJsonp 的函数。它接受的三个参数，分别名为chunkIds, moreModules, executeModules。对应了 index.js 中调用 webpackJsonp 时传入的三个参数。而 webpackJsonp 内究竟是有怎样的逻辑呢？
<br>
先不管定义的参数，webpackJsonp 先是 for in 遍历了一次 moreModules,将 moreModules 内的所有方法都存在 modules， 也就是自执行函数执行时传入的数组。
之后是一个条件判断：


```javascript
if (executeModules) {
  for (i = 0; i < executeModules.length; i++) {
    result = __webpack_require__(executeModules[i]);
  }
}
```

判断 executeModules， 也就是第三个参数是否存在，如存在即执行 __webpack_require__ 方法。在 index.js 调用 webpackJsonp 方法时，这个参数当然是存在的，因而要看看 __webpack_require__ 方法是什么了。
__webpack_require__ 接受一个名为 moduleId 的参数。方法内部首先是一个条件判断，先不管。接下来看到赋值逻辑


```javascript
var module = installedModules[moduleId] = {
  exports: {}
};
```


结合刚才的条件判断，可以推测出 installedModules 是一个缓存的容器，那么前面的代码意思就是如果缓存中有对应的 moduleId,那么直接返回它的 exports，不然就定义并赋值一个吧。接着先偷看一下 __webpack_require__ 的最后的返回值，可以看到函数返回的是 module.exports，那么 module.exports 又是如何被赋值的呢？ 看看之后的代码：
```javascript
modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
```

刚才我们知道 modules[moduleId] 就是 moreModules 中的方法，此处就是将 this 指定为 module.exports，再把module, module.exports, __webpack_require__ 传入去作为参数调用。这三个参数是不是很熟悉？之前我们看 index.js 里面代码时，有一个疑问就是模块化是如何实现的。这里我们已经看出了眉目。

其实 webpack 就是将每一个 js 文件封装成一个函数，每个文件中的 require 方法对应的就是 __webpack_require__， __webpack_require__ 会根据传入的 moduleId 再去加载对应的代码。而当我们想导出 js 文件的值时，要么用 module.exports，要么用 exports，这就对应了module, module.exports两个参数。少接触这块的童鞋，应该就能理解为何导出值时，直接使用 exports = xxx 会导出失败了。

梳理一下打包后代码执行的流程，首先 minifest.js 会定义一个 webpackJsonp 方法，待其他打包后的文件（也可称为 chunk）调用。当调用 chunk 时，会先将该 chunk 中所有的 moreModules， 也就是每一个依赖的文件也可称为 module （如 test.js）存起来。之后通过 executeModules 判断这个文件是不是入口文件，决定是否执行第一次  __webpack_require__。而 __webpack_require__ 的作用，就是根据这个 module 所 require 的东西，不断递归调用 __webpack_require__，__webpack_require__函数返回值后供 require 使用。当然，模块是不会重复加载的，因为 installedModules 记录着 module 调用后的 exports 的值，只要命中缓存，就返回对应的值而不会再次调用 module。webpack 打包后的文件，就是通过一个个函数隔离 module 的作用域，以达到不互相污染的目的。

