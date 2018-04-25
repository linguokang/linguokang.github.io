---
title: (转载)基于webpack的前端工程化开发之多页站点
date: 2017-12-26 15:48:57
tags:
    - webpack
---
<article class="markdown-body entry-content" itemprop="text"><h2><a href="#基于webpack的前端工程化开发之多页站点篇一" aria-hidden="true" class="anchor" id="user-content-基于webpack的前端工程化开发之多页站点篇一"></a></h2>
<p>在最初接触webpack的较长一段时间里，我（也可能很多人）都觉得webpack是专为单页应用而量身打造的，比如webpack+react、webpack+vue等，都可以近乎完美的解决各种资源的依赖加载、打包的问题。甚至css都是打包在js里去动态添加到dom文档中去。</p>
<!-- more -->
<p>后来想想，这么好的工具这么好的方案为什么不能用在website（普通的web站点，姑且叫做website吧）中呢？</p>
<ul>
<li>
<p>首先对于普通的web站点，我们更倾向于将css独立出来，因为对于website来说，css还是要最先加载出来比较好。</p>
</li>
<li>
<p>再然后js我们也只想加载需要的部分，而不是一个大大的打包了所有js模块的包。</p>
</li>
</ul>
<p>在很多webpack入门级的demo里，无论是单入口的还是多入口的，都没有解决上面两个问题。入门毕竟是入门，要晋级还是只能靠自己。幸运的是，有很多优秀的工程师已为我们铺好了路，让我们在前端工程化的道路上少走很多的弯路。如果你也一样曾迷茫过，请不要走开，希望这里能为你答疑解惑。</p>
<p>好吧，以上通通是废话，接下来上干货。</p>
<hr>
<p>首先开始构建我们的项目目录结构。</p>
<h4><a href="#初始化项目安装依赖" aria-hidden="true" class="anchor" id="user-content-初始化项目安装依赖"><svg aria-hidden="true" class="octicon octicon-link" height="16" version="1.1" viewBox="0 0 16 16" width="16"><path fill-rule="evenodd" d="M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z"></path></svg></a>初始化项目、安装依赖</h4>
<p>使用<code>npm init</code>初始化项目就不多说了，生成package.json文件。</p>
<p>使用<code>npm install plugins --save-dev</code>安装项目所需依赖。最终package.json的依赖声明如下：</p>
<div class="highlight highlight-source-js"><pre><span class="pl-s"><span class="pl-pds">"</span>devDependencies<span class="pl-pds">"</span></span><span class="pl-k">:</span> {
  <span class="pl-s"><span class="pl-pds">"</span>css-loader<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^0.23.1<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>extract-text-webpack-plugin<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^1.0.1<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>file-loader<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^0.8.5<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>html-loader<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^0.4.3<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>html-webpack-plugin<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^2.9.0<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>jquery<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^1.12.0<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>less<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^2.6.0<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>less-loader<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^2.2.2<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>style-loader<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^0.13.0<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>url-loader<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^0.5.7<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>webpack<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^1.12.13<span class="pl-pds">"</span></span>,
  <span class="pl-s"><span class="pl-pds">"</span>webpack-dev-server<span class="pl-pds">"</span></span><span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>^1.14.1<span class="pl-pds">"</span></span>
}</pre></div>
<h4><a href="#主要目录结构" aria-hidden="true" class="anchor" id="user-content-主要目录结构"><svg aria-hidden="true" class="octicon octicon-link" height="16" version="1.1" viewBox="0 0 16 16" width="16"><path fill-rule="evenodd" d="M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z"></path></svg></a>主要目录结构</h4>
<pre><code>- website
  - src                #代码开发目录
    - css              #css目录，按照页面（模块）、通用、第三方三个级别进行组织
      + page
      + common
      + lib
    + img              #图片资源
    - js               #JS脚本，按照page、components进行组织
      + page
      + components
    + view             #HTML模板
  - dist               #webpack编译打包输出目录，无需建立目录可由webpack根据配置自动生成
    + css                
    + js
    + view
  + node_modules       #所使用的nodejs模块
  package.json         #项目配置
  webpack.config.js    #webpack配置
  README.md            #项目说明
</code></pre>
<blockquote>
<p>假如你是一名纯粹的前端工程师，使用webpack构建website的目录结构大概就这样了，当然你也可以根据自己的喜好自由设计目录结构。</p>
<p>详细的代码全貌可以提前在这里“窥看”——<a href="https://github.com/vhtml/webpack-MultiPage-static">https://github.com/vhtml/webpack-MultiPage-static</a>。</p>
</blockquote>
<p>目录组织好，我们就可以开始撸代码了。</p>
<h4><a href="#开发页面" aria-hidden="true" class="anchor" id="user-content-开发页面"><svg aria-hidden="true" class="octicon octicon-link" height="16" version="1.1" viewBox="0 0 16 16" width="16"><path fill-rule="evenodd" d="M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z"></path></svg></a>开发页面</h4>
<p>在src/js/page目录下建立index.js文件，在src/view目录下建立index.html文件。入口js和模板文件名对应。</p>
<p>index.js内容如下：</p>
<div class="highlight highlight-source-js"><pre><span class="pl-c"><span class="pl-c">//</span>引入css</span>
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">"</span>../../css/lib/reset.css<span class="pl-pds">"</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">"</span>../../css/common/global.css<span class="pl-pds">"</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">"</span>../../css/common/grid.css<span class="pl-pds">"</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">"</span>../../css/page/index.less<span class="pl-pds">"</span></span>)
<span class="pl-en">$</span>(<span class="pl-s"><span class="pl-pds">'</span>.g-bd<span class="pl-pds">'</span></span>).<span class="pl-c1">append</span>(<span class="pl-s"><span class="pl-pds">'</span>&lt;p class="text"&gt;这是由js生成的一句话。&lt;/p&gt;<span class="pl-pds">'</span></span>)</pre></div>
<p>index.html 内容如下：</p>
<div class="highlight highlight-text-xml"><pre>&lt;!<span class="pl-k">DOCTYPE</span> <span class="pl-c1">html</span>
&lt;<span class="pl-ent">html</span> <span class="pl-e">lang</span>=<span class="pl-s"><span class="pl-pds">"</span>en<span class="pl-pds">"</span></span>&gt;
&lt;<span class="pl-ent">head</span>&gt;
  &lt;<span class="pl-ent">meta</span> <span class="pl-e">charset</span>=<span class="pl-s"><span class="pl-pds">"</span>UTF-8<span class="pl-pds">"</span></span>&gt;
  &lt;<span class="pl-ent">title</span>&gt;首页&lt;/<span class="pl-ent">title</span>&gt;
  &lt;<span class="pl-ent">meta</span> <span class="pl-e">name</span>=<span class="pl-s"><span class="pl-pds">"</span>description<span class="pl-pds">"</span></span> <span class="pl-e">content</span>=<span class="pl-s"><span class="pl-pds">"</span>基于webpack的前端工程化开发解决方案探索<span class="pl-pds">"</span></span>/&gt;
  <span class="pl-c"><span class="pl-c">&lt;!--</span></span>
<span class="pl-c">    描述：head中无需再引入css以及facicon，webpack将根据入口JS文件的要求自动实现按需加载或者生成style标签</span>
<span class="pl-c">  <span class="pl-c">--&gt;</span></span>
&lt;/<span class="pl-ent">head</span>&gt;
&lt;<span class="pl-ent">body</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-hd<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-bd<span class="pl-pds">"</span></span>&gt;
    &lt;<span class="pl-ent">input</span> <span class="pl-e">type</span>=<span class="pl-s"><span class="pl-pds">"</span>button<span class="pl-pds">"</span></span> <span class="pl-e">value</span>=<span class="pl-s"><span class="pl-pds">"</span>弹窗<span class="pl-pds">"</span></span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>btn<span class="pl-pds">"</span></span>&gt;
    &lt;<span class="pl-ent">p</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>img<span class="pl-pds">"</span></span>&gt;
      &lt;<span class="pl-ent">img</span> <span class="pl-e">src</span>=<span class="pl-s"><span class="pl-pds">"</span>../img/4.png<span class="pl-pds">"</span></span> <span class="pl-e">alt</span>=<span class="pl-s"><span class="pl-pds">"</span><span class="pl-pds">"</span></span>&gt;
    &lt;/<span class="pl-ent">p</span>&gt;
  &lt;/<span class="pl-ent">div</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-ft<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
  <span class="pl-c"><span class="pl-c">&lt;!--</span></span>
<span class="pl-c">    描述：body中同样无需单独引入JS文件，webpack会根据入口JS文件自动实现按需加载或者生成script标签，还可以生成对应的hash值</span>
<span class="pl-c">  <span class="pl-c">--&gt;</span></span>
&lt;/<span class="pl-ent">body</span>&gt;
&lt;/<span class="pl-ent">html</span>&gt;</pre></div>
<p>就是这样一个简单的HTML模板，我们甚至没有引入任何CSS和JS，通过webpack打包就可以自动帮我们引入。</p>
<p>由于是做website，在此之前相信你对单页应用打包已经有过了解，我就不客气了，再来两个页面压压惊。</p>
<p>about.js:</p>
<div class="highlight highlight-source-js"><pre><span class="pl-c"><span class="pl-c">//</span>引入css</span>
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>../../css/lib/reset.css<span class="pl-pds">'</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>../../css/common/global.css<span class="pl-pds">'</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>../../css/common/grid.css<span class="pl-pds">'</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>../../css/page/about.less<span class="pl-pds">'</span></span>)
<span class="pl-en">$</span>(<span class="pl-s"><span class="pl-pds">'</span>#about<span class="pl-pds">'</span></span>).<span class="pl-en">html</span>(<span class="pl-s"><span class="pl-pds">'</span>这是一个关于webpack构建工程的栗子<span class="pl-pds">'</span></span>)</pre></div>
<p>about.html:</p>
<div class="highlight highlight-text-xml"><pre>&lt;!<span class="pl-k">DOCTYPE</span> <span class="pl-c1">html</span>&gt;
&lt;<span class="pl-ent">html</span> <span class="pl-e">lang</span>=<span class="pl-s"><span class="pl-pds">"</span>en<span class="pl-pds">"</span></span>&gt;
&lt;<span class="pl-ent">head</span>&gt;
  &lt;<span class="pl-ent">meta</span> <span class="pl-e">charset</span>=<span class="pl-s"><span class="pl-pds">"</span>UTF-8<span class="pl-pds">"</span></span>&gt;
  &lt;<span class="pl-ent">title</span>&gt;关于&lt;/<span class="pl-ent">title</span>&gt;
  &lt;<span class="pl-ent">meta</span> <span class="pl-e">name</span>=<span class="pl-s"><span class="pl-pds">"</span>description<span class="pl-pds">"</span></span> <span class="pl-e">content</span>=<span class="pl-s"><span class="pl-pds">"</span>基于webpack的前端工程化开发解决方案探索<span class="pl-pds">"</span></span>/&gt;
&lt;/<span class="pl-ent">head</span>&gt;
&lt;<span class="pl-ent">body</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-hd<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-bd<span class="pl-pds">"</span></span>&gt;
    &lt;<span class="pl-ent">div</span> <span class="pl-e">id</span>=<span class="pl-s"><span class="pl-pds">"</span>about<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
  &lt;/<span class="pl-ent">div</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-ft<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
&lt;/<span class="pl-ent">body</span>&gt;
&lt;/<span class="pl-ent">html</span>&gt;</pre></div>
<p>list.js:</p>
<div class="highlight highlight-source-js"><pre><span class="pl-c"><span class="pl-c">//</span>引入css</span>
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>../../css/lib/reset.css<span class="pl-pds">'</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>../../css/common/global.css<span class="pl-pds">'</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>../../css/common/grid.css<span class="pl-pds">'</span></span>)
<span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>../../css/page/list.less<span class="pl-pds">'</span></span>)
<span class="pl-k">var</span> html <span class="pl-k">=</span> <span class="pl-s"><span class="pl-pds">'</span><span class="pl-pds">'</span></span>
<span class="pl-k">for</span>(<span class="pl-k">var</span> i <span class="pl-k">=</span> <span class="pl-c1">0</span>; i <span class="pl-k">&lt;</span> <span class="pl-c1">5</span>; i<span class="pl-k">++</span>){
  html <span class="pl-k">+=</span> <span class="pl-s"><span class="pl-pds">'</span>&lt;li&gt;列表<span class="pl-pds">'</span></span> <span class="pl-k">+</span> (i <span class="pl-k">+</span> <span class="pl-c1">1</span>) <span class="pl-k">+</span> <span class="pl-s"><span class="pl-pds">'</span>&lt;/li&gt;<span class="pl-pds">'</span></span>
}
<span class="pl-en">$</span>(<span class="pl-s"><span class="pl-pds">'</span>#list<span class="pl-pds">'</span></span>).<span class="pl-en">html</span>(html);</pre></div>
<p>list.html:</p>
<div class="highlight highlight-text-xml"><pre>&lt;!<span class="pl-k">DOCTYPE</span> <span class="pl-c1">html</span>&gt;
&lt;<span class="pl-ent">html</span> <span class="pl-e">lang</span>=<span class="pl-s"><span class="pl-pds">"</span>en<span class="pl-pds">"</span></span>&gt;
&lt;<span class="pl-ent">head</span>&gt;
  &lt;<span class="pl-ent">meta</span> <span class="pl-e">charset</span>=<span class="pl-s"><span class="pl-pds">"</span>UTF-8<span class="pl-pds">"</span></span>&gt;
  &lt;<span class="pl-ent">title</span>&gt;列表页&lt;/<span class="pl-ent">title</span>&gt;
  &lt;<span class="pl-ent">meta</span> <span class="pl-e">name</span>=<span class="pl-s"><span class="pl-pds">"</span>description<span class="pl-pds">"</span></span> <span class="pl-e">content</span>=<span class="pl-s"><span class="pl-pds">"</span>基于webpack的前端工程化开发解决方案探索<span class="pl-pds">"</span></span>/&gt;
&lt;/<span class="pl-ent">head</span>&gt;
&lt;<span class="pl-ent">body</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-hd<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-bd<span class="pl-pds">"</span></span>&gt;
    &lt;<span class="pl-ent">ul</span> <span class="pl-e">id</span>=<span class="pl-s"><span class="pl-pds">"</span>list<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">ul</span>&gt;
  &lt;/<span class="pl-ent">div</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-ft<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
&lt;/<span class="pl-ent">body</span>&gt;
&lt;/<span class="pl-ent">html</span>&gt;</pre></div>
<p>OK，太棒了！！！</p>
<h4><a href="#webpack配置" aria-hidden="true" class="anchor" id="user-content-webpack配置"><svg aria-hidden="true" class="octicon octicon-link" height="16" version="1.1" viewBox="0 0 16 16" width="16"><path fill-rule="evenodd" d="M4 9h1v1H4c-1.5 0-3-1.69-3-3.5S2.55 3 4 3h4c1.45 0 3 1.69 3 3.5 0 1.41-.91 2.72-2 3.25V8.59c.58-.45 1-1.27 1-2.09C10 5.22 8.98 4 8 4H4c-.98 0-2 1.22-2 2.5S3 9 4 9zm9-3h-1v1h1c1 0 2 1.22 2 2.5S13.98 12 13 12H9c-.98 0-2-1.22-2-2.5 0-.83.42-1.64 1-2.09V6.25c-1.09.53-2 1.84-2 3.25C6 11.31 7.55 13 9 13h4c1.45 0 3-1.69 3-3.5S14.5 6 13 6z"></path></svg></a>webpack配置</h4>
<p>这里是关键，在webpack.config.js里，我们将进行一些配置，来完成我们的需求，一开始或许有点难理解，但等你真的掌握了，你便会惊呼它的神奇。配置中我写了详细的注释，要想彻底理解，还需多实践，多查阅文档，必要时看看源码，呜呼，学习之路漫漫兮。</p>
<div class="highlight highlight-source-js"><pre><span class="pl-k">var</span> path <span class="pl-k">=</span> <span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>path<span class="pl-pds">'</span></span>)
<span class="pl-k">var</span> webpack <span class="pl-k">=</span> <span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>webpack<span class="pl-pds">'</span></span>)
<span class="pl-c"><span class="pl-c">/*</span></span>
<span class="pl-c">extract-text-webpack-plugin插件，</span>
<span class="pl-c">有了它就可以将你的样式提取到单独的css文件里，</span>
<span class="pl-c">妈妈再也不用担心样式会被打包到js文件里了。</span>
<span class="pl-c"> <span class="pl-c">*/</span></span>
<span class="pl-k">var</span> ExtractTextPlugin <span class="pl-k">=</span> <span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>extract-text-webpack-plugin<span class="pl-pds">'</span></span>);
<span class="pl-c"><span class="pl-c">/*</span></span>
<span class="pl-c">html-webpack-plugin插件，重中之重，webpack中生成HTML的插件，</span>
<span class="pl-c">具体可以去这里查看https://www.npmjs.com/package/html-webpack-plugin</span>
<span class="pl-c"> <span class="pl-c">*/</span></span>
<span class="pl-k">var</span> HtmlWebpackPlugin <span class="pl-k">=</span> <span class="pl-c1">require</span>(<span class="pl-s"><span class="pl-pds">'</span>html-webpack-plugin<span class="pl-pds">'</span></span>);
<span class="pl-c1">module</span>.<span class="pl-smi">exports</span> <span class="pl-k">=</span> {
  entry<span class="pl-k">:</span> { <span class="pl-c"><span class="pl-c">//</span>配置入口文件，有几个写几个</span>
    index<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/js/page/index.js<span class="pl-pds">'</span></span>,
    list<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/js/page/list.js<span class="pl-pds">'</span></span>,
    about<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/js/page/about.js<span class="pl-pds">'</span></span>,
  },
  output<span class="pl-k">:</span> { 
    path<span class="pl-k">:</span> <span class="pl-smi">path</span>.<span class="pl-c1">join</span>(<span class="pl-c1">__dirname</span>, <span class="pl-s"><span class="pl-pds">'</span>dist<span class="pl-pds">'</span></span>), <span class="pl-c"><span class="pl-c">//</span>输出目录的配置，模板、样式、脚本、图片等资源的路径配置都相对于它</span>
    publicPath<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>/dist/<span class="pl-pds">'</span></span>,       <span class="pl-c"><span class="pl-c">//</span>模板、样式、脚本、图片等资源对应的server上的路径</span>
    filename<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>js/[name].js<span class="pl-pds">'</span></span>,     <span class="pl-c"><span class="pl-c">//</span>每个页面对应的主js的生成配置</span>
    chunkFilename<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>js/[id].chunk.js<span class="pl-pds">'</span></span>   <span class="pl-c"><span class="pl-c">//</span>chunk生成的配置</span>
  },
  module<span class="pl-k">:</span> {
    loaders<span class="pl-k">:</span> [ <span class="pl-c"><span class="pl-c">//</span>加载器，关于各个加载器的参数配置，可自行搜索之。</span>
      {
        test<span class="pl-k">:</span><span class="pl-sr"> <span class="pl-pds">/</span><span class="pl-cce">\.</span>css<span class="pl-k">$</span><span class="pl-pds">/</span></span>,
        <span class="pl-c"><span class="pl-c">//</span>配置css的抽取器、加载器。'-loader'可以省去</span>
        loader<span class="pl-k">:</span> <span class="pl-smi">ExtractTextPlugin</span>.<span class="pl-en">extract</span>(<span class="pl-s"><span class="pl-pds">'</span>style-loader<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>css-loader<span class="pl-pds">'</span></span>) 
      }, {
        test<span class="pl-k">:</span><span class="pl-sr"> <span class="pl-pds">/</span><span class="pl-cce">\.</span>less<span class="pl-k">$</span><span class="pl-pds">/</span></span>,
        <span class="pl-c"><span class="pl-c">//</span>配置less的抽取器、加载器。中间!有必要解释一下，</span>
        <span class="pl-c"><span class="pl-c">//</span>根据从右到左的顺序依次调用less、css加载器，前一个的输出是后一个的输入</span>
        <span class="pl-c"><span class="pl-c">//</span>你也可以开发自己的loader哟。有关loader的写法可自行谷歌之。</span>
        loader<span class="pl-k">:</span> <span class="pl-smi">ExtractTextPlugin</span>.<span class="pl-en">extract</span>(<span class="pl-s"><span class="pl-pds">'</span>css!less<span class="pl-pds">'</span></span>)
      }, {
        <span class="pl-c"><span class="pl-c">//</span>html模板加载器，可以处理引用的静态资源，默认配置参数attrs=img:src，处理图片的src引用的资源</span>
        <span class="pl-c"><span class="pl-c">//</span>比如你配置，attrs=img:src img:data-src就可以一并处理data-src引用的资源了，就像下面这样</span>
        test<span class="pl-k">:</span><span class="pl-sr"> <span class="pl-pds">/</span><span class="pl-cce">\.</span>html<span class="pl-k">$</span><span class="pl-pds">/</span></span>,
        loader<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">"</span>html?attrs=img:src img:data-src<span class="pl-pds">"</span></span>
      }, {
        <span class="pl-c"><span class="pl-c">//</span>文件加载器，处理文件静态资源</span>
        test<span class="pl-k">:</span><span class="pl-sr"> <span class="pl-pds">/</span><span class="pl-cce">\.</span>(woff<span class="pl-k">|</span>woff2<span class="pl-k">|</span>ttf<span class="pl-k">|</span>eot<span class="pl-k">|</span>svg)(<span class="pl-cce">\?</span>v=<span class="pl-c1">[<span class="pl-c1">0-9</span>]</span><span class="pl-cce">\.</span><span class="pl-c1">[<span class="pl-c1">0-9</span>]</span><span class="pl-cce">\.</span><span class="pl-c1">[<span class="pl-c1">0-9</span>]</span>)<span class="pl-k">?</span><span class="pl-k">$</span><span class="pl-pds">/</span></span>,
        loader<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>file-loader?name=./fonts/[name].[ext]<span class="pl-pds">'</span></span>
      }, {
        <span class="pl-c"><span class="pl-c">//</span>图片加载器，雷同file-loader，更适合图片，可以将较小的图片转成base64，减少http请求</span>
        <span class="pl-c"><span class="pl-c">//</span>如下配置，将小于8192byte的图片转成base64码</span>
        test<span class="pl-k">:</span><span class="pl-sr"> <span class="pl-pds">/</span><span class="pl-cce">\.</span>(png<span class="pl-k">|</span>jpg<span class="pl-k">|</span>gif)<span class="pl-k">$</span><span class="pl-pds">/</span></span>,
        loader<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>url-loader?limit=8192&amp;name=./img/[hash].[ext]<span class="pl-pds">'</span></span>
      }
    ]
  },
  plugins<span class="pl-k">:</span> [
    <span class="pl-k">new</span> <span class="pl-en">webpack.ProvidePlugin</span>({ <span class="pl-c"><span class="pl-c">//</span>加载jq</span>
      $<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>jquery<span class="pl-pds">'</span></span>
    }),
    <span class="pl-k">new</span> <span class="pl-en">webpack.optimize.CommonsChunkPlugin</span>({
      name<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>vendors<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span> 将公共模块提取，生成名为`vendors`的chunk</span>
      chunks<span class="pl-k">:</span> [<span class="pl-s"><span class="pl-pds">'</span>index<span class="pl-pds">'</span></span>,<span class="pl-s"><span class="pl-pds">'</span>list<span class="pl-pds">'</span></span>,<span class="pl-s"><span class="pl-pds">'</span>about<span class="pl-pds">'</span></span>], <span class="pl-c"><span class="pl-c">//</span>提取哪些模块共有的部分</span>
      minChunks<span class="pl-k">:</span> <span class="pl-c1">3</span> <span class="pl-c"><span class="pl-c">//</span> 提取至少3个模块共有的部分</span>
    }),
    <span class="pl-k">new</span> <span class="pl-en">ExtractTextPlugin</span>(<span class="pl-s"><span class="pl-pds">'</span>css/[name].css<span class="pl-pds">'</span></span>), <span class="pl-c"><span class="pl-c">//</span>单独使用link标签加载css并设置路径，相对于output配置中的publickPath</span>
    <span class="pl-c"><span class="pl-c">//</span>HtmlWebpackPlugin，模板生成相关的配置，每个对于一个页面的配置，有几个写几个</span>
    <span class="pl-k">new</span> <span class="pl-en">HtmlWebpackPlugin</span>({ <span class="pl-c"><span class="pl-c">//</span>根据模板插入css/js等生成最终HTML</span>
      favicon<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/img/favicon.ico<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>favicon路径，通过webpack引入同时可以生成hash值</span>
      filename<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./view/index.html<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>生成的html存放路径，相对于path</span>
      template<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/view/index.html<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>html模板路径</span>
      inject<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>body<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>js插入的位置，true/'head'/'body'/false</span>
      hash<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>为静态资源生成hash值</span>
      chunks<span class="pl-k">:</span> [<span class="pl-s"><span class="pl-pds">'</span>vendors<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>index<span class="pl-pds">'</span></span>],<span class="pl-c"><span class="pl-c">//</span>需要引入的chunk，不配置就会引入所有页面的资源</span>
      minify<span class="pl-k">:</span> { <span class="pl-c"><span class="pl-c">//</span>压缩HTML文件  </span>
        removeComments<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>移除HTML中的注释</span>
        collapseWhitespace<span class="pl-k">:</span> <span class="pl-c1">false</span> <span class="pl-c"><span class="pl-c">//</span>删除空白符与换行符</span>
      }
    }),
    <span class="pl-k">new</span> <span class="pl-en">HtmlWebpackPlugin</span>({ <span class="pl-c"><span class="pl-c">//</span>根据模板插入css/js等生成最终HTML</span>
      favicon<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/img/favicon.ico<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>favicon路径，通过webpack引入同时可以生成hash值</span>
      filename<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./view/list.html<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>生成的html存放路径，相对于path</span>
      template<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/view/list.html<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>html模板路径</span>
      inject<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>js插入的位置，true/'head'/'body'/false</span>
      hash<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>为静态资源生成hash值</span>
      chunks<span class="pl-k">:</span> [<span class="pl-s"><span class="pl-pds">'</span>vendors<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>list<span class="pl-pds">'</span></span>],<span class="pl-c"><span class="pl-c">//</span>需要引入的chunk，不配置就会引入所有页面的资源</span>
      minify<span class="pl-k">:</span> { <span class="pl-c"><span class="pl-c">//</span>压缩HTML文件  </span>
        removeComments<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>移除HTML中的注释</span>
        collapseWhitespace<span class="pl-k">:</span> <span class="pl-c1">false</span> <span class="pl-c"><span class="pl-c">//</span>删除空白符与换行符</span>
      }
    }),
    <span class="pl-k">new</span> <span class="pl-en">HtmlWebpackPlugin</span>({ <span class="pl-c"><span class="pl-c">//</span>根据模板插入css/js等生成最终HTML</span>
      favicon<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/img/favicon.ico<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>favicon路径，通过webpack引入同时可以生成hash值</span>
      filename<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./view/about.html<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>生成的html存放路径，相对于path</span>
      template<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./src/view/about.html<span class="pl-pds">'</span></span>, <span class="pl-c"><span class="pl-c">//</span>html模板路径</span>
      inject<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>js插入的位置，true/'head'/'body'/false</span>
      hash<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>为静态资源生成hash值</span>
      chunks<span class="pl-k">:</span> [<span class="pl-s"><span class="pl-pds">'</span>vendors<span class="pl-pds">'</span></span>, <span class="pl-s"><span class="pl-pds">'</span>about<span class="pl-pds">'</span></span>],<span class="pl-c"><span class="pl-c">//</span>需要引入的chunk，不配置就会引入所有页面的资源</span>
      minify<span class="pl-k">:</span> { <span class="pl-c"><span class="pl-c">//</span>压缩HTML文件  </span>
        removeComments<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>移除HTML中的注释</span>
        collapseWhitespace<span class="pl-k">:</span> <span class="pl-c1">false</span> <span class="pl-c"><span class="pl-c">//</span>删除空白符与换行符</span>
      }
    }),
    <span class="pl-k">new</span> <span class="pl-en">webpack.HotModuleReplacementPlugin</span>() <span class="pl-c"><span class="pl-c">//</span>热加载</span>
  ],
  <span class="pl-c"><span class="pl-c">//</span>使用webpack-dev-server，提高开发效率</span>
  devServer<span class="pl-k">:</span> {
    contentBase<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>./<span class="pl-pds">'</span></span>,
    host<span class="pl-k">:</span> <span class="pl-s"><span class="pl-pds">'</span>localhost<span class="pl-pds">'</span></span>,
    port<span class="pl-k">:</span> <span class="pl-c1">9090</span>, <span class="pl-c"><span class="pl-c">//</span>默认8080</span>
    inline<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>可以监控js变化</span>
    hot<span class="pl-k">:</span> <span class="pl-c1">true</span>, <span class="pl-c"><span class="pl-c">//</span>热启动</span>
  }
};</pre></div>
<p>好了，完成以上的这些配置之后，执行<code>webpack</code>打包命令完成项目打包。</p>
<p>此时，前往/dist/view目录下查看生成的list.html文件，如下：</p>
<div class="highlight highlight-text-xml"><pre>&lt;!<span class="pl-k">DOCTYPE</span> <span class="pl-c1">html</span>&gt;
&lt;<span class="pl-ent">html</span> <span class="pl-e">lang</span>=<span class="pl-s"><span class="pl-pds">"</span>en<span class="pl-pds">"</span></span>&gt;
&lt;<span class="pl-ent">head</span>&gt;
  &lt;<span class="pl-ent">meta</span> <span class="pl-e">charset</span>=<span class="pl-s"><span class="pl-pds">"</span>UTF-8<span class="pl-pds">"</span></span>&gt;
  &lt;<span class="pl-ent">title</span>&gt;列表页&lt;/<span class="pl-ent">title</span>&gt;
  &lt;<span class="pl-ent">meta</span> <span class="pl-e">name</span>=<span class="pl-s"><span class="pl-pds">"</span>description<span class="pl-pds">"</span></span> <span class="pl-e">content</span>=<span class="pl-s"><span class="pl-pds">"</span>基于webpack的前端工程化开发解决方案探索<span class="pl-pds">"</span></span>&gt;
&lt;<span class="pl-ent">link</span> <span class="pl-e">rel</span>=<span class="pl-s"><span class="pl-pds">"</span>shortcut icon<span class="pl-pds">"</span></span> <span class="pl-e">href</span>=<span class="pl-s"><span class="pl-pds">"</span>/dist/favicon.ico<span class="pl-pds">"</span></span>&gt;&lt;<span class="pl-ent">link</span> <span class="pl-e">href</span>=<span class="pl-s"><span class="pl-pds">"</span>/dist/css/vendors.css?02568e631b7717b7149a<span class="pl-pds">"</span></span> <span class="pl-e">rel</span>=<span class="pl-s"><span class="pl-pds">"</span>stylesheet<span class="pl-pds">"</span></span>&gt;&lt;<span class="pl-ent">link</span> <span class="pl-e">href</span>=<span class="pl-s"><span class="pl-pds">"</span>/dist/css/list.css?02568e631b7717b7149a<span class="pl-pds">"</span></span> <span class="pl-e">rel</span>=<span class="pl-s"><span class="pl-pds">"</span>stylesheet<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">head</span>&gt;
&lt;<span class="pl-ent">body</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-hd<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-bd<span class="pl-pds">"</span></span>&gt;
    &lt;<span class="pl-ent">ul</span> <span class="pl-e">id</span>=<span class="pl-s"><span class="pl-pds">"</span>list<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">ul</span>&gt;
  &lt;/<span class="pl-ent">div</span>&gt;
  &lt;<span class="pl-ent">div</span> <span class="pl-e">class</span>=<span class="pl-s"><span class="pl-pds">"</span>g-ft<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">div</span>&gt;
&lt;<span class="pl-ent">script</span> <span class="pl-e">src</span>=<span class="pl-s"><span class="pl-pds">"</span>/dist/js/vendors.js?02568e631b7717b7149a<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">script</span>&gt;&lt;<span class="pl-ent">script</span> <span class="pl-e">src</span>=<span class="pl-s"><span class="pl-pds">"</span>/dist/js/list.js?02568e631b7717b7149a<span class="pl-pds">"</span></span>&gt;&lt;/<span class="pl-ent">script</span>&gt;&lt;/<span class="pl-ent">body</span>&gt;
&lt;/<span class="pl-ent">html</span>&gt;</pre></div>
<p>可以看到生成的文件除了保留原模板中的内容以外，还根据入口文件list.js的定义，自动添加需要引入CSS与JS文件，以及favicon，同时还添加了相应的hash值。</p>
<p>执行<code>webpack-dev-server</code>启动devServer，打开<a href="http://localhost:9090/dist/view/index.html%E5%B0%B1%E5%8F%AF%E4%BB%A5%E8%BF%9B%E8%A1%8C%E6%AD%A3%E5%B8%B8%E7%9A%84%E9%A1%B5%E9%9D%A2%E9%A2%84%E8%A7%88%E4%BA%86%E3%80%82%E8%AF%B4%E6%98%8E%E6%88%91%E4%BB%AC%E7%9A%84%E8%B5%84%E6%BA%90%E8%B7%AF%E5%BE%84%E7%94%9F%E6%88%90%E4%B9%9F%E6%98%AF%E6%B2%A1%E6%9C%89%E9%97%AE%E9%A2%98%E7%9A%84%E3%80%82" rel="nofollow">http://localhost:9090/dist/view/index.html就可以进行正常的页面预览了。说明我们的资源路径生成也是没有问题的。</a></p>
<p>好了，纯静态的webpack前端构建过程就是这样了。然而你可能还有疑问。</p>
<ul>
<li>假如你是个懒人，可能会觉得目前的配置不够智能，每增加一个页面，就得再手动添加入口文件及模板配置。</li>
<li>假如你是个全栈工程师或者以nodejs做中间层的开发者，你的模板不是纯粹的html，但是又需要像html模板那样能自动根据需要添加css与js文件。</li>
</ul>
<p>还等什么，快戳这里吧——<a href="https://github.com/vhtml/webpack-MultiplePage">基于webpack的前端工程化开发之多页站点篇（二）</a></p>
</article>
