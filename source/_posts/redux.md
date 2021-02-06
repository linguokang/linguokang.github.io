---
title: 【阅读笔记】深入理解redux
date: 2021-02-06 22:54:00
---
### REDUX是什么

下面是redux官方给出的解释：

> Redux is a predictable state container for JavaScript apps.
<!-- more -->

### 简介

1. redux是的诞生是为了给 React 应用提供「可预测化的状态管理」机制。
2. Redux会将整个应用状态(其实也就是数据)存储到到一个地方，称为store
3. 这个store里面保存一棵状态树(state tree)
4. 组件改变state的唯一方法是通过调用store的dispatch方法，触发一个action，这个action被对应的reducer处理，于是state完成更新
5. 组件可以派发(dispatch)行为(action)给store,而不是直接通知其它组件
6. 其它组件可以通过订阅store中的状态(state)来刷新自己的视图

### redux的工作流程

![Alt text](./16ad40d90c5d46fa.webp)


### 最小化实现REDUX

我们要实现的redux主要有如下几个功能：

- 获取应用state
- 发送action
- 监听state变化
- 让我们来看下redux store暴漏的api

```javascript
const store = {
  state: {}, // 全局唯一的state，内部变量，通过getState()获取
  listeners: [], // listeners，用来诸如视图更新的操作
  dispatch: () => {}, // 分发action
  subscribe: () => {}, // 用来订阅state变化
  getState: () => {}, // 获取state
}

```

我们来实现createStore，它返回store对象

```javascript
const createStore = (reducer, initialState) => {
  // internal variables
  const store = {};
  store.state = initialState;
  store.listeners = [];

  // api-subscribe
  store.subscribe = (listener) => {
    store.listeners.push(listener);
  };
  // api-dispatch
  store.dispatch = (action) => {
    store.state = reducer(store.state, action);
    store.listeners.forEach(listener => listener());
  };

  // api-getState
  store.getState = () => store.state;

  return store;
};


```
通过上面的20行左右的代码已经实现了redux的最基本功能了

### 使用

我们现在可以像使用redux一样使用了我们的"redux"了。

```javascript
// reducer
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

let store = createStore(counter)

store.subscribe(() =>
  console.log(store.getState())
)


store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1

```
