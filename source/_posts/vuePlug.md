---
title: Vue插件开发
date: 2017-12-01 11:55:03
tags:
  - vue.js
---
相对组件来说，Vue 的插件开发受到的关注要少一点。但是插件的功能是十分强大的，能够完成许多 Vue 框架本身不具备的功能。 

大家一般习惯直接调用现成的插件，比如官方推荐的 vue-router、vue-touch 等。 下面就看一下 Vue 的插件开发如何入门。

<!-- more -->

插件通常用于为 Vue 添加全局级别的功能。然而对于插件，并没有严格限定其使用范围 - 下面是常见的几种插件类型：

- 添加一些全局方法或属性。例如 vue-custom-element

- 添加一个或多个全局资源(asset)：指令(directives)/过滤器(filters)/过渡(transitions) 等。例如 vue-touch

- 通过全局混合，添加一些组件选项。例如 vue-router

- 添加一些 Vue 实例方法，通过把这些方法添加到 Vue.prototype 上实现。

- 一个可以提供 API 的库(library)，与此同时也是以上功能的组合。例如 vue-router

## 编写插件

Vue.js 插件应该暴露一个 install 方法。此方法在调用时，将 Vue 构造函数作为第一个参数传入，以及将一个可选的选项作为第二个参数传入：

```bash
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 一些逻辑……
  }
  // 2. 添加一个全局资源(asset)
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 一些逻辑……
    }
    ...
  })
  // 3. 注入一些组件选项
  Vue.mixin({
    created: function () {
      // 一些逻辑……
    }
    ...
  })
  // 4. 添加一个实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 一些逻辑……
  }
}
```


## 使用插件

插件在使用时有两种方式：

>第一种是如上述提到的，可以通过 Vue.use(yourPlugin) 全局方法进行调用。 进行 Vue 的大型项目开发时，如果用 vue-cli 生成项目目录结构，Vue.use() 方法一般在 main.js 中调用。


>第二种实际上是插件本身帮你完成了 Vue.use()的调用。 这种情况下，插件会去检测是否存在 Vue 全局变量，如果存在，就自动调用 Vue.use()。所以，如果你的项目中是使用 script 方式引入的 Vue.js（这种情况下 Vue 才会作为一个全局变量存在），在使用比如 vue-router 之类的插件时就可以直接引入然后使用，不用再调用 Vue.use()。 但是，在模块环境下应当始终显式调用该方法，以保证插件可以正常使用：

```bash
// 通过 Browserify 或 Webpack 使用 CommonJS 兼容模块
var Vue = require('vue')
var VueRouter = require('vue-router')
// 不要忘了调用此方法
Vue.use(VueRouter)

// 或者可以多传入一个选项对象
// Vue.use(VueRouter, { hashbang: true })
```

## 你的第一个插件

接下来要讲到的 vue-toast 插件则是通过添加实例方法实现的。我们先来看个小例子。先新建个js文件来编写插件：toast.js

```bash
/ toast.js
var Toast = {};
Toast.install = function (Vue, options) {
    Vue.prototype.$msg = 'Hello World';
}
module.exports = Toast;
```

在 main.js 中，需要导入 toast.js 并且通过全局方法 Vue.use() 来使用插件：

```bash
// main.js
import Vue from 'vue';
import Toast from './toast.js';
Vue.use(Toast);
```

然后，我们在组件中来获取该插件定义的 $msg 属性。

```bash
// App.vue
export default {
    mounted(){
        console.log(this.$msg);         // Hello World
    }
}
```
可以看到，控制台成功的打印出了 Hello World 。这种方法就是添加一个实例方法来实现的。


## 开发一个vue-loading插件

本次将直接使用vue简化版脚手架.

初始化项目
```bash
vue init webpack-simple vue-pay-keyboard
```
这里我选择了使用sass 因为后面我会用到 大家可以自行选择

OK 一切先用默认的,等会我们再改, 启动项目之后，我们把自带的多余的东西全删掉，然后在src下面新建一个 lib文件 用于存放我们的源码
并新建 index.js 用于作为我们的插件入口
新建一个 vue-pay-keyboard.vue 存放我们的开发的组件
至此 文件目录如下：
```bash
.
├── src                            // 源码目录
│   ├── lib                        // 源码
│   │   ├── index.js               // 插件入口
│   │   ├── vue-loading.vue   // 组件
│   ├── App.vue                    // 页面入口文件
│   ├── main.js                    // 程序入口文件，加载各种公共组件
├── index.html                     // 入口html文件
.

```
然后开始写我们的index.js
我们需要导入.vue组件，并按照vue插件规范开发并且导出,这里我们用全局组件的方式作为插件
```bash
import vueLoading from './vue-loading.vue' // 导入组件
const Loading = {
  install(Vue, options) {
    Vue.component(vueLoading.name, vueLoading) // vuePayKeyboard.name 组件的name属性
    // 类似通过 this.$xxx 方式调用插件的 其实只是挂载到原型上而已
    // Vue.prototype.$xxx  // 最终可以在任何地方通过 this.$xxx 调用
    // 虽然没有明确规定用$开头  但是大家都默认遵守这个规定
  }
}

//如果检测到 Vue 是可访问的全局变量，插件会自动调用 Vue.use()
if (typeof window !== 'undefined' && window.Vue) {
  window.Vue.use(Loading);
}

export default Loading // 导出..
```
ok 可以说基本的开发环境我们已经搭建完成了,接下来我们写弹窗的业务逻辑
因为太长就不贴出来了，详见点击[链接](https://github.com/linguokang/vue-loading/blob/master/src/lib/vue-loading.vue)

### 使用vue-loading
在main.js中导入vue-loading插件
```bash
import Loading from 'vue-loading-easy'
Vue.use(Loading)
```

在app.vue中使用插件
```bash
  <vue-loading :isLoading='isLoading' type="loading2" text="嘿嘿嘿" color='lightgreen' scale='1'></vue-loading>
```

### 演示

[demo](https://linguokang.github.io/vue-loading/)
[github地址](https://github.com/linguokang/vue-loading)

### Props

| name          | Description  | type     | default  |
|:-------------: |:------------:|:-----: |:-----:|
| isLoading     | 是否显示      | Boolean  |false
| type          | 加载样式      | String   |loading1
| text          | 文字显示      | String   |加载中...
| color         | 图标颜色      | String   |lightgreen
| scale         | 弹窗缩放      | String   |1

## 打包

至此 我们所有开发都完成了 前面都是大家熟悉的 现在进入打包以及发布 也都是几分钟的事情.
修改 weppack.config.js 部分配置

```bash
module.exports = {
 // entry: './src/main.js',  // 项目入口 我们通过npm run dev 就是从这里进去的 我们通过run build 打包编译也是
  // 因为我们要打包的插件在lib里面 所以稍稍改一下
   entry: './src/lib/index.js', // 注释掉原有的
  output: {
    path: path.resolve(__dirname, './dist'), 
    publicPath: '/dist/',
   // filename: 'build.js' // 打包后输出的文件名
    filename: 'vue-loading.js' // 我们可不想打包后叫build.js 多low啊 起一个与项目相对应的
    library:'vueLoading',// library指定的就是你使用require时的模块名，这里便是require("vueLoading")
    libraryTarget: 'umd', //libraryTarget会生成不同umd的代码,可以只是commonjs标准的，也可以是指amd标准的，也可以只是通过script标签引入的。
    umdNamedDefine: true // 会对 UMD 的构建过程中的 AMD 模块进行命名。否则就使用匿名的 define。
  },

```
ok 通过 npm run build 打包一次
此时可以看到生成了一个dist文件里面并且有两个文件 一个vue-loading.js与一个map文件














