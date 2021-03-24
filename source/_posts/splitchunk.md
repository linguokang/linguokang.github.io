---
title: SplitChunksPlugin
date: 2021-03-24 11:35:56
tags:
  - 阅读笔记
  - SplitChunks
---

webpack4 中支持了零配置的特性，同时对块打包也做了优化，CommonsChunkPlugin 已经被移除了，现在是使用 optimization.splitChunks 代替。

<!-- more -->

## 常用参数

- minSize(默认是 30000)：形成一个新代码块最小的体积
- minChunks（默认是 1）：在分割之前，这个代码块最小应该被引用的次数（译注：保证代码块复用性，默认配置的策略是不需要多次引用也可以被分割）
- maxInitialRequests（默认是 3）：一个入口最大的并行请求数
- maxAsyncRequests（默认是 5）：按需加载时候最大的并行请求数。
- chunks (默认是 async) ：initial、async 和 all
- test： 用于控制哪些模块被这个缓存组匹配到。原封不动传递出去的话，它默认会选择所有的模块。可以传递的值类型：RegExp、String 和 Function
- name(打包的 chunks 的名字)：字符串或者函数(函数可以根据条件自定义名字)
- priority ：缓存组打包的先后优先级。

下面这个配置对象代表 SplitChunksPlugin 的默认行为。
webpack.config.js

```javascript
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async', // 必须三选一： "initial" | "all"(推荐) | "async" (默认就是async)
      minSize: 20000, // 最小尺寸，30000
      minRemainingSize: 0,
      maxSize: 0,
      minChunks: 1, // 最小 chunk ，默认1
      maxAsyncRequests: 30, // 最大异步请求数， 默认5
      maxInitialRequests: 30, // 最大初始化请求书，默认3
      enforceSizeThreshold: 50000,
      cacheGroups: {
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/, //用于控制哪些模块被这个缓存组匹配到
          priority: -10, //缓存组打包的先后优先级
          reuseExistingChunk: true //如果当前代码块包含的模块已经有了，就不在产生一个新的代码块
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```

### 模块全部是同步引入

```javascript
// indexA.js
import React from 'react'
import ReactDOM from 'react-dom'
import _ from 'lodash'

console.log(_.join(['a', 'b'], '~'))

ReactDOM.render(
  <div>SplitChunk</div>,
  document.getElementById('root')
)

```
![Alt text](https://user-gold-cdn.xitu.io/2018/9/16/165e245b6a79a710?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
默认情况只会影响按需加载模块，所以所有内容全部被打包到一起了。

### 有模块动态导入
```javascript
// indexA.js
import React from 'react'
import ReactDOM from 'react-dom'
import _ from 'lodash'
import(/* webpackChunkName: "async-jquery" */ 'jquery').then(component => {
  console.log(component)
})

console.log(_.join(['a', 'b'], '~'))

ReactDOM.render(
  <div>SplitChunk</div>,
  document.getElementById('root')
)
```
![Alt text](https://user-gold-cdn.xitu.io/2018/9/16/165e24a3649144b4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
这里jquery使用动态导入，打包结果中可以看到jquery被单独打包了


### chunks

chunks的取值是有initial, async, all。默认情况下是async，在本文第一部分已经介绍了它的表现，所以现在来看下其它两个的表现。
initial, all模式会将所有来自node_modules的模块分配到一个叫vendors的缓存组；所有重复引用至少两次的代码，会被分配到default的缓存组。

```javascript
// indexA.js
import './Dashboard.jsx'

// indexB.js
import './Dashboard.jsx'

// Dashboard.jsx
import React from 'react'
```
```javascript
/ webpack.config.js
splitChunks: {
  chunks: 'initial'
}
```
![Alt text](https://user-gold-cdn.xitu.io/2018/9/16/165e258100bd82e0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
打包表现正如上面所述，产生了两个代码块vendors, default。

里面有提到initial模式下会分开优化打包异步和非异步模块。而all会把异步和非异步同时进行优化打包。也就是说moduleA在indexA中异步引入，indexB中同步引入，initial下moduleA会出现在两个打包块中，而all只会出现一个。

### 提取第三方库

上面已经提到了设置chunks: initial || all都可以提取出第三方库。但是它是把所有第三库提取出来，所以我们在只提取react和react-dom的情况下，需要自定义一个cacheGroup。
```javascript
// indexA.js
import React from 'react'
import ReactDOM from 'react-dom'
import _ from 'lodash'
import $ from 'jquery'

// webpack.config.js
entry: {
    indexA: path.join(__dirname, 'src/indexA.js')
},
optimization: {
    splitChunks: {
        chunks: 'all',
        cacheGroups: {
            vendors: {
                test: /react/,
                name: 'vendors'
            }
        }
    }
}
```

![Alt text](https://user-gold-cdn.xitu.io/2018/9/16/165e28007b2f44ca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
我们去重写了vendors打包块，只打包匹配react的模块，所以达到了之前CommonsChunkPlugin的功能。