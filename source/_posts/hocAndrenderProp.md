---
title: Render Props and HOC
date: 2019-11-08 11:13:10
tags:
     - react.js
---


文章目录：

*   什么是 Render Props
*   Render Props 的应用
*   什么是 React Hooks
*   React Hooks 的应用
*   总结.

<!-- more -->

什么是 Render Props
----------------

简而言之，只要一个组件中某个属性的值是函数，那么就可以说改组件使用了 Render Props 这种技术。听起来好像就那么回事儿，那到底 Render Props 有哪些应用场景呢，让我们还是从简单的例子讲起，假如我们要实现一个打招呼的组件，一开始可能会这么实现：
```yaml
    const Greeting = props => (
        <div>
            <h1>{props.text}</h1>
        </div>
    );
    
    // 然后这么使用
    <Greeting text="Hello 🌰！" />
```

但是如果在打招呼的时候同时还需要发送一个表情呢，然后可能会这么实现：
```yaml
    const Greeting = props => (
        <div>
            <h1>{props.text}</h1>
            <p>{props.emoji}</p>
        </div>
    );
    // how to use
    <Greeting text="Hello 🌰！" emoji="😳" />
```

然后如果还要加上链接呢，又要在 `Greeting` 组件的内部实现发送链接的逻辑，很明显这种方式违背了软件开发六大原则之一的 **开闭原则**，即每次修改都要到组件内部需修改。

> 开闭原则：对修改关闭，对拓展开放。

那有什么方法可以避免这种方式的修改呢，当然有，也就是接下来要讲的 **Render Props**，不过在此之前，我们先来看一个非常简单的求和函数：
```yaml
    const sumOf = array => {
        const sum = array.reduce((prev, current) => {
            prev += current;
            return prev;
        }, 0);
        console.log(sum);
    }
```

这个函数的功能非常简单，对数组求和并打印它。但是如果需要把 `sum` 通过 `alert` 显示出来，是不是又要到 `sumOf` 内部去修改呢，和上面的 `Greeting` 类似，是的，这两个函数存在相同的问题，就是当需求有变是，都需要要函数内部去修改。

对于第二个函数，你可能很快就能想出用 **回调函数** 去解决：
```yaml
    const sumOf = (array, done) => {
        const sum = array.reduce((prev, current) => {
            prev += current;
            return prev;
        }, 0);
        done(sum);
    }
    
    sumOf([1, 2, 3], sum => {
        console.log(sum);
        // or
        alert(sum);
    })
```

会发现回调函数很完美的解决了之前存在的问题，每次修改，我们只需要在 `sumOf` 函数的回调函数中去修改，而不需要到 `sumOf` 内部去修改。

反观 React 组件 `Greeting`，要解决前面遇到的问题，其实和 `sumOf` 的回调函数一样：
```yaml
    const Greeting = props => {
        return props.render(props);
    };
    // how to use
    <Greeting
        text="Hello 🌰！"
        emoji="😳"
        link="link here"
        render={(props) => (
        <div>
            <h1>{props.text}</h1>
            <p>{props.emoji}</p>
            <a href={props.link}></a>
        </div>
    )}></Greeting>
```

类比之前的 `sumOf` 是不是非常的相似，简直就是一毛一样：

*   `sumOf` 中通过执行回调函数 `done` 并把 `sum` 传入其中，此时只要在 `sumOf` 函数的第二个参数中传入一个函数即可获得 `sum` 的值，进而做一写定制化的需求
*   `Greeting` 中通过执行回调函数 `props.render` 并把 `props` 传入其中，此时只要在 `Greeting` 组件的 `render` 属性中传入一个函数即可获得 `props` 的值并返回你所需要的 UI

值得一提的是，并不是只有在 `render` 属性中传入函数才能叫 Render Props，实际上任何属性只要它的值是函数，都可称之为 Render Props，比如上面这个例子把 `render` 属性名改成 `children` 的话使用上其实更为简便：
```yaml
    const Greeting = props => {
        return props.children(props);
    };
    // how to use
    <Greeting text="Hello 🌰！" emoji="😳" link="link here">
    {(props) => (
        <div>
            <h1>{props.text}</h1>
            <p>{props.emoji}</p>
            <a href={props.link}></a>
        </div>
    )}
    </Greeting>
```

这样就可以直接在 `Greeting` 标签内写函数了，比起之前在 `render` 中更为直观。

所以，**React 中的 Render Props 你可以把它理解成 JavaScript 中的回调函数**。

Render Props 的应用
----------------

上面简单介绍了什么是 Render Props，那么在实际开发中 Render Props 具体有什么实际应用呢，简单来讲 Render Props 所解决的问题和 **高阶组件** 所解决的问题类似，都是为了 **解决代码复用的问题**。


简单实现一个「开关」功能的组件：
```yaml
    class Switch extends React.Component {
        constructor(props) {
            super(props);
            this.state = {
                on: props.initialState || false,
            };
        }
        toggle() {
            this.setState({
                on: !this.state.on,
            });
        }
        render() {
            return (
                <div>{this.props.children({
                    on,
                    toggle: this.toggle,
                })}</div>
            );
        }
    }
    // how to use
    const App = () => (
        <Switch initialState={false}>{({on, toggle}) => {
            <Button onClick={toggle}>Show Modal</Button>
            <Modal visible={on} onSure={toggle}></Modal>
        }}</Switch>
    );
```

这是一个简单的 **复用显隐模态弹窗逻辑** 的组件，比如要显示 `OtherModal` 就直接替换 `Modal` 就行了，达到复用「开关」逻辑代码的目的。

Render Props 更像是 **控制反转（IoC）**，它只负责定义接口或数据并通过函数参数传递给你，具体怎么使用这些接口或者数据完全取决于你。


### Render Props VS HOC

前面提到过 Render Props 所解决的问题和 **高阶组件** 所解决的问题类似，都是为了 **解决代码复用的问题**，那它们有什么区别呢，让我们来简单分析一下它们各自的特点：

#### HOC

**缺点：**

*   由于可能会多次嵌套高阶组件，而我们又很难确保每个高阶组件中的属性名是不同的，所以 **属性容易被覆盖**。
*   当在使用高阶组件的时候，高阶组件相当于一个 **黑盒**，我们必须去看如何实现才能去使用它：
*   不够直接。我们不必再担心 state 或者 props 来自哪里。我们可以看到通过 render prop 的参数列表看到有哪些 state 或者 props 可供使用。

**优点：**

*   可以使用 `compose` 方法合并多个高阶组件然后在使用
```yaml
    // 不要这么使用
    const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))；
    // 可以使用一个 compose 函数组合这些高阶组件
    // lodash, redux, ramda 等第三方库都提供了类似 `compose` 功能的函数
    const enhance = compose(withRouter, connect(commentSelector))；
    const EnhancedComponent = enhance(WrappedComponent)；
```

*   调用方便（ES6 + 装饰器语法）
```yaml
    @withData   
    class App extends React.Component {}
```

#### Render Props

*   缺点
    *   嵌套过深也会形成 **地狱回调**
*   优点
    *   解决了 HOC 的缺点

Render Props 和 HOC 并不是非此即彼的关系，明白它们各自的优缺点之后，我们就可以在合适的场景下使用适合的方式去实现了。



总结
--

Render Props 的一个核心思想就是在组件内部通过调用 `this.props.children()` （当然可以是其他任意值是函数属性名）把组件内部的一些状态传递出去（参考回调函数），然后在组件外部对应属性的函数中通过函数的参数来获取组件内部的状态，进而在该函数中处理相应的 UI 或逻辑。

目前为止，介绍了 React 代码复用的两种方式：

*   **Render Props**
*   **HOC**


**相关阅读：**

*   **[Render Props](https://zh-hans.reactjs.org/docs/render-props.html)**
*   **[高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html)**

