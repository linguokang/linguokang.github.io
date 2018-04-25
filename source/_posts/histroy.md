---
title: 框架的由来（转载）
date: 2018-03-05 17:46:05
tags:
---
<div><p>文章很长，麻烦大家耐心看，一定有收获。</p><p>我用React较多，我就说React。我问一个问题：<b>我们为什么需要React?</b></p>
<!-- more -->
<h2><b>一些废话</b></h2><p>或许在你看到这个话题的时候，脑海里一定会想</p><ul><li><b>组件化</b></li><li><b>性能好</b></li><li><b>大家都在用，好找工作</b></li></ul><p>在现在MVVM框架横行的年代里，大家学习前端基本上都是直奔三大框架而去，这是没错的，毕竟熟练框架的操作，确实能很快的找到一份像样的工作，但是很可能就不会知道「<b>为什么会出现这些框架</b>」了，这意味着，在使用框架之后的很长一段时间里，你都是知其然不知所以然。</p><p><br></p><p>这个带来的问题就是，每当一个框架「蹦」出来的时候，你总会非常非常的紧张，觉得「他妈的又有要学的东西了，蛋疼」。之前的 React 协议的问题，着实把很多还没入行的朋友吓的直哆嗦，很多粉丝也都在问我，React是否还有学习的必要...</p><p><br></p><h2><b>一）无框架的年代，我们如何开发的？</b></h2><figure><img src="https://pic3.zhimg.com/v2-0a2ec9a894dcc46b48659442b1ef136a_b.jpg" data-caption="" data-size="normal" data-rawwidth="102" data-rawheight="68" class="content_image" width="102"></figure><p>我们需要开发这样的一个功能，点击+号按钮，上面的数字就加1。HTML代码如下:

```bash
<div class="wrap">
        <div class="text">0</div>
        <button class='plus'>
        +
        </button>
</div>
```

再来一段Javascript的代码，我们的功能就能完备了：

```bash
<script>
//document.querySelector，其实就是DOM的api，选择一个元素

    const wrap = document.querySelector('.wrap')
    const plus = wrap.querySelector('.plus')
    const text = wrap.querySelector('.text')

    let number = 0
//dom api，给元素添加一个监听器
    plus.addEventListener('click', function () {
        number++
        text.innerHTML = number
    }, false)

</script>

```
上古时期的开发们大约就是这样去书写代码，因为功能非常的简单，所以代码量也很少，思路很清晰。随着页面的元素越来越多，你这个「点击加1按钮」的功能可能被复用，那么你现在用一个最快的办法对其进行复用：<b>复制HTML-&gt;复制JS代码-&gt;粘贴HTML和JS代码</b></p><p>这种方法是很挫的.....于是为了简单化我们的流程，我们想出了一个聪明的办法去复用</p><h2><b>1.1看似聪明的办法</b></h2><p><i>（前提：因为FB总部被炸了，正在修理，导致ES6出来了，React还没发布）

```bash
<script>
    class Add {
        render() {
            return ` 
            <div class="text">0</div>
            <button class='plus'>
            +
            </button>`
        }
    }
    const wrap = document.querySelector('.wrap')

    const add = new Add()
    wrap.innerHTML = add.render()

    const plus = wrap.querySelector('.plus')
    const text = wrap.querySelector('.text')

    let number = 0
    plus.addEventListener('click', function () {
        number++
        text.innerHTML = number
    }, false)
</script>
```
代码稍微的一改，我们用一个class Add代替了HTML，当调用这个类的render函数的时候，就会返回一段字符串，这段字符串就是我们想要的组件。</p><p>由此，我们的组件复用方法变成了：<b>复制JS代码-&gt;粘贴JS代码</b></p><p>这么一来啊，我们就简化了我们组件复用的流程。但是我们可以看到，我们手动操作DOM的次数太多了，而且逻辑非常的重复，因此，我们把这部分逻辑抽象一下，使得我们组件复用更加简单。</p><h2><b>1.2 更加简单的组件复用</h2>
```bash
<script>
    class Add {
        createWrapper(string) {
            //document.createElement, DOM api，根据标签名字创建DOM元素
            const wrapper = document.createElement('div')
            wrapper.innerHTML = string
            return wrapper
        }

        render() {
            const domString = ` 
            <div class="text">0</div>
            <button class='plus'>
            +
            </button>`

            //生成DOM元素
            const wrapper = this.createWrapper(domString)
            //DOM中查找元素
            const plus = wrapper.querySelector('.plus')
            const text = wrapper.querySelector('.text')

            let number = 0
            plus.addEventListener('click', function () {
                number++
                text.innerHTML = number
            }, false)
            return wrapper
        }
    }
    const wrap = document.querySelector('.wrap')
    const add = new Add()
    wrap.appendChild(add.render())
</script>
```
现在我们的这个组件复用就非常的简单了，我们只需要将class Add 通过export的方法导出以后，你可以在任何地方使用几行代码，就可以使用他，我们复用我们的组件就变成了：<b>复制三行JS代码-&gt;粘贴三行JS代码</b></p><div class="highlight"><pre><code class="language-js"><span class="kr">const</span> <span class="nx">wrap</span> <span class="o">=</span> <span class="nb">document</span><span class="p">.</span><span class="nx">querySelector</span><span class="p">(</span><span class="s1">'.wrap'</span><span class="p">)</span>
<span class="kr">const</span> <span class="nx">add</span> <span class="o">=</span> <span class="k">new</span> <span class="nx">Add</span><span class="p">()</span>
<span class="nx">wrap</span><span class="p">.</span><span class="nx">appendChild</span><span class="p">(</span><span class="nx">add</span><span class="p">.</span><span class="nx">render</span><span class="p">())</span>
</code></pre></div><h2><b>二）数据驱动组件</b></h2><p>什么是数据驱动呢？简单的来说，就是「<b>数据是什么，我们就展示什么</b>」。回顾我们之前的组件内部</p>

```bash
//生成DOM元素
            const wrapper = this.createWrapper(domString)
            //DOM中查找元素
            const plus = wrapper.querySelector('.plus')
            const text = wrapper.querySelector('.text')

            let number = 0
            plus.addEventListener('click', function () {
                number++
                text.innerHTML = number
            }, false)

```
<p>我们在其中大量的使用了，querySelector这种api，使得我们需要各种手动的去操作DOM元素，如果我们的组件变成</p>

```bash
<div class="wrap">
        <div class="text">0</div>
        <div class="text_2">0</div>
        <div class="text_3">0</div>
        <button class='plus'>
        +
        </button>
</div>
```
<p>然后我需要点一下按钮，三个标签都得变化，那我们的代码很可能就需要对每一个text进行选择：</p>
```bash

//生成DOM元素
            const wrapper = this.createWrapper(domString)
            //DOM中查找元素
            const plus = wrapper.querySelector('.plus')
            const text = wrapper.querySelector('.text')
            const text_2 = wrapper.querySelector('.text_2')
            const text_3 = wrapper.querySelector('.text_3')

            let number = 0
            plus.addEventListener('click', function () {
                number++
                text.innerHTML = number
                text_2.innerHTML = number
                text_3.innerHTML = number
            }, false)
```
<p>这种做法，显然是不符合我们的预期的，因为这非常影响我们的开发效率，而我们也仅仅是简单的增添了一些新的标签以及展示，我们都非得增加一大堆的代码。</p><p><br></p><h2><b>2.1 引入组件的状态</b></h2><p><b>「数据是什么，我们就展示什么」,</b>这是我们最终的想要达到的成果，方法很多，但是最省代码的方式和最简单的方式就是：当我们数据发生改变的同时，我们根据数据的变化，重新渲染整个组件，那就简单多了。</p><p><br></p><p>话非常的绕，我们的代码这么去写：</p>

```bash
<script>
    class Add {
        constructor() {//构造函数，设置state
            this.state = {
                number: 0
            }
        }
        //重写setState方法，使得每次调用setState，就会重新渲染，更新组件
        setState(NewState) {
            const oldElement = this.wrapper
            this.state = {...NewState}//ES6语法，展开操作符
            this.wrapper = this.render()//重新渲染

            //拿到新的组件，更新原来的组件
            if (this.update) this.update(oldElement, this.wrapper)
        }
        createWrapper(string) {
            const wrapper = document.createElement('div')
            wrapper.innerHTML = string
            return wrapper
        }
        onClick() {//事件调用
            let NewState = this.state.number + 1
            this.setState({
                number: NewState
            })
        }
        render() {
            const domString = ` 
            <div class="text">${this.state.number}</div>
            <div class="text_2">${this.state.number}</div>
            <div class="text_3">${this.state.number}</div>
            <button class='plus'>
            +
            </button>`

            this.wrapper = this.createWrapper(domString)
            const plus = this.wrapper.querySelector('.plus')

            plus.addEventListener('click', this.onClick.bind(this), false)
            return this.wrapper
        }
    }

    const wrap = document.querySelector('.wrap')
    const add = new Add()
    wrap.appendChild(add.render())
    add.update = (old, next) => {//给组件定义一个方法
        wrap.insertBefore(next, old)//插入新的组件
        wrap.removeChild(old)//去掉旧的组件
    }
</script>

```
<p>代码稍微有些长了，但是其实很好理解。我们最终达到的目的就是「当我们数据发生改变的同时，我们根据数据的变化，重新渲染整个组件」</p><figure><img src="https://pic1.zhimg.com/v2-b927b14df5ecdbd7265996c60c8b99b8_b.jpg" data-caption="" data-size="normal" data-rawwidth="297" data-rawheight="132" class="content_image" width="297"></figure><h2><b>2.2 抽象：让所有组件都拥有自动更新的能力</b></h2><p>我们现在已经自己开发出一个非常容易去复用的组件了，但是当我们要开发另外一个组件的时候或许就要重新写一套这样的「复用」逻辑。在我们的思想里，所有的组件都应该拥有「<b>自动更新的能力」</b>，因此我们将组件根据状态，自动更新的能力抽取出来，那我们以后就非常方便了。</p>

```bash

class ButtonComponent {
        constructor() {
        }
        createWrapper(string) {
            //document.createElement, DOM api，根据标签名字创建DOM元素
            const wrapper = document.createElement('div')
            wrapper.innerHTML = string
            return wrapper
        }

        setState(newState) {
            const oldElement = this.wrapper
            this.state = {...newState}
            this.wrapper = this.renderElement()//渲染出元素
            if (this.update) this.update(oldElement, this.wrapper)
        }

        renderElement() {
            this.wrapper = this.createWrapper(this.render())
            const plus = this.wrapper.querySelector('.plus')
            if (this.onClick) {
                plus.addEventListener('click', this.onClick.bind(this), false)
            }
            return this.wrapper
        }

        render() { }//玩家需要重写的函数
    }
```
<p>好了，逻辑抽象完成。还记得我们的React有一个ReactDOM.render方法吗？我们改一下名字直接叫做：</p>
```bash
const renderToDOM = (component, DOMElement) => {
        DOMElement.appendChild(component.renderElement())
        component.update = (old, next) => {//给组件定义一个方法
            DOMElement.insertBefore(next, old)//插入新的组件
            DOMElement.removeChild(old)//去掉旧的组件
        }
    }

```
<p>最后呢，我们刚刚的组件，可以写成这样～</p>
```bash
class Add extends ButtonComponent {
        constructor() {
            super()
            this.state = {
                number:0
            }
        }
        onClick() {//事件调用
            let NewState = this.state.number + 1
            this.setState({
                number: NewState
            })
        }
        render() {
            return ` 
            <div class="text">${this.state.number}</div>
            <div class="text_2">${this.state.number}</div>
            <div class="text_3">${this.state.number}</div>
            <button class='plus'>
            +
            </button>`
        }
    }

    renderToDOM(new Add, document.getElementById('wrap'))

```
<p>自此，我们的组件化就完成了。你可以看到，我们的组件其实外貌上已经非常像React，并且已经拥有了一个良好的组件复用能力。</p><p><br></p><h2><b>最后：我们为什么需要React?</b></h2><p>通过上述的一点一点分析，实现我们可以得知，实际上React的出现，已经完完全全的颠覆了传统的开发模式。</p><p><br></p><p>我们无须再去：</p><ul><li>使用DOM API，一个一个的元素进行更新，操作</li><li>组件复用非常的简单，几乎每次复用之前写的组件，就是1-2行代码就搞定</li><li>组件内部状态，自己管理自己，跟外界完全无关</li></ul><p><br></p><p>实际上，<b>所有的框架出现都是为了解决「开发者困难」的问题</b>。试想一下，一个10万行的项目，整天都是一个一个的元素进行更新，操作，那请10个程序员都维护不过来。有了这些框架的帮助，我们就能够更好的开发。</p><p><br></p><p>当我们站在这个角度去看前端框架的时候，我们就会发现，其实三大框架也不过就是：<b>react牌锤子，vue牌锤子，ng牌锤子，他们都是锤子，以后就算有框架出来，也还是锤子....</b></p></div>