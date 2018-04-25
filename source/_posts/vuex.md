---
title: vuex总结
date: 2017-12-16 10:54:00
tags:
  - vuex
---

### vuex是干什么的？

一句话总结起来就是：解决不同组件之间通信的问题。

<!-- more -->

组件之间的作用域独立，而组件之间经常又需要传递数据。
A 为父组件，下面有子组件 B 和 C。
A 的数据可以通过 props 传递给 B 和 C。
A 可以通过 $broadcast 调用 B 和 C 的 events，从而操作 B 和 C 的数据。
B 和 C 可以通过 $dispatch 调用 A 的 events，从而操作 A 的数据。
当 B 需要操作 C 的数据就会比较麻烦，需要先 $dispatch 到 A，再 $broadcast 到 C。
如果项目比较小的话还好，越大的项目，涉及的组件通信就越多、越频繁，此时管理起来就会非常累，而且容易出错。
这就是 Vuex 的意义所在。它可以将数据置于单独的一层，并提供给外部操作内部数据的方法。粗俗一点，就这样理解吧。

### 安装

 直接下载 / CDN 引用

```bash
<script src="/path/to/vue.js"></script>
<script src="/path/to/vuex.js"></script>
```


 NPM
> npm install vuex --save

```bash
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)
```

### 核心概念

每一个 Vuex 应用的核心就是 store（仓库）。“store”基本上就是一个容器，它包含着你的应用中大部分的状态 (state)。Vuex 和单纯的全局对象有以下两点不同：

- Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。

- 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。
  
### State

单一状态树，每个应用将仅仅包含一个 store 实例。

那么怎么在在 Vue 组件中获得 Vuex 的状态呢？

Vuex 通过 store 选项，提供了一种机制将状态从根组件“注入”到每一个子组件中（需调用 Vue.use(Vuex)）：

通过在根实例中注册 store 选项，该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 this.$store 访问到。


### mapState 辅助函数

当一个组件需要获取多个状态时候，将这些状态都声明为计算属性会有些重复和冗余。使用mapState批量获取store.state的值


### Getter

getters从表面是获得的意思，可以把他看作在获取数据之前进行的一种再编辑,相当于对数据的一个过滤和加工。你可以把它看作store.js的计算属性。

Getter 会暴露为 store.getters 对象

### mapGetters 辅助函数

mapGetters 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性，和mapState类似。

### Mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。

要提交一个mutation，你需要以相应的 type 调用 store.commit 方法。

### 提交载荷（Payload）

你可以向 store.commit 传入额外的参数，即 mutation 的 载荷（payload）：

定义
```bash
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```
调用
```bash
store.commit('increment', {
  amount: 10
})
```

### actions异步修改状态

actions和之前讲的Mutations功能基本一样，不同点是，actions是异步的改变state状态，而Mutations是同步改变状态。Action 提交的是 mutation，而不是直接变更状态。

#### 分发 Action

Action 通过 store.dispatch 方法触发

>store.dispatch('increment')

Actions 支持同样的载荷方式和对象方式进行分发：

```bash
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})
```
在组件中使用 this.$store.dispatch('xxx') 分发 action，或者使用 mapActions 辅助函数将组件的 methods 映射为 store.dispatch 调用（需要先在根节点注入 store）

### 项目结构

├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块


官方文档：[https://vuex.vuejs.org/zh-cn/intro.html](https://vuex.vuejs.org/zh-cn/intro.html)
