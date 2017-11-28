---
title: '使用hexo+github搭建个人博客'
date: 2017-11-21 16:15:59
tags:
     - 黑魔法
     - 奇淫技巧
---
## 1.前言
使用github pages服务搭建博客的好处有：
1. 全是静态文件，访问速度快；
2. 免费方便，不用花一分钱就可以搭建一个自由的个人博客，不需要服务器不需要后台；
3. 可以随意绑定自己的域名，不仔细看的话根本看不出来你的网站是基于github的；
4. 数据绝对安全，基于github的版本管理，想恢复到哪个历史版本都行；
5. 博客内容可以轻松打包、转移、发布到其它平台；

<!-- more -->

### 准备工作
在开始一切之前，你必须已经：
- 有一个github账号，没有的话去注册一个；
- 安装了node.js、npm，并了解相关基础知识；
- 安装了git for windows（或者其它git客户端）
 
---
## 2.使用hexo写博客
hexo简介
>Hexo是一个简单、快速、强大的基于 Github Pages 的博客发布工具，支持Markdown格式，有众多优秀插件和主题。

官网：[http://hexo.io](http://hexo.io)
github:[https://github.com/hexojs/hexo](https://github.com/hexojs/hexo)

### 安装
>$ npm install -g hexo

### 初始化
新建一个名为hexo的文件夹，比如我的是F:\Workspaces\hexo
>$ cd /f/Workspaces/hexo/

>$ hexo init 

hexo会自动下载一些文件到这个目录，包括node_modules，目录结构如下图：
![Alt text](./20160818_115922_773_1148.png)

>$ hexo g  #生成静态文件

执行以上命令之后，hexo就会在public文件夹生成相关html文件，这些文件将来都是要提交到github去的

>$ hexo s  #启动服务

hexo s是开启本地预览服务，打开浏览器访问 [http://localhost:4000](http://localhost:4000) 即可看到内容


## 3.部署到github
1.new repository
![Alt text](./1531909-8decffce7d3866b3.png)
![Alt text](./123.png)
 仓库名字和用户名保持一致
 然后点击
![Alt text](./1531909-fe29bad673d85f37.png)
 
2.在_config.yml进行配置
![Alt text](./456.png)

3.安装hexo-deployer-git自动部署发布工具
>npm instal lhexo-deployer-git  --save

4.发布到Github
>hexo clean && hexo g && hexo d


![Alt text](./LLPFAQ949D8DV9N$Y3S1ZO9.png)
发布完成

5.测试访问

在浏览器输入：https://linguokang.github.io/

![Alt text](./789.png)

