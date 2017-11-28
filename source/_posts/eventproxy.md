---
title: eventproxy控制并发
date: 2017-11-23 14:35:28
tags:
---

### EventProxy是一个通过控制事件触发顺序来控制业务流程的工具。

1. 利用事件机制解耦复杂业务逻辑
2. 移除被广为诟病的深度callback嵌套问题
3. 将串行等待变成并行等待，提升多异步协作场景下的执行效率
4. 友好的Error handling

node.js的亮点是回调函数，node.js流程控制、传参都是通过回调函数来实现的。
开发中经常会遇到回调嵌套的场景，尤其是在业务复杂的场景下，会嵌套n层回调函数，这样做的原因是为了控制代码执行的流程。

<!-- more -->

下面是一个需要同步读取文件的例子

```bash
var str = '';
fs.readFile('file1.txt', function(err, data1) {
    str += data1;
    fs.readFile('file2.txt', function(err, data2) {
        str += data2;
        fs.readFile('file3.txt', function(err, data3) {
            str += data3;
        });
    });
});
```
上面代码有3层回调嵌套，用嵌套来保障文件的顺序读取。这样代码可读性就比较差了，而开发中用到的嵌套往往有很多层，代码很难读。

### 一、安装eventproxy

>eventproxy在github:[https://github.com/JacksonTian/eventproxy](https://github.com/JacksonTian/eventproxy)

```bash
npm install eventproxy --save
```
### 二、使用eventproxy的步骤

```bash
var eventproxy=require("eventproxy");
```
引入eventproxy

```bash
var ep=new eventproxy();
```
创建eventproxy对象

```bash
ep.all('event_name',function(result){

});
```
告诉它需要监听哪些事件，并提供回调函数

```bash
ep.emit('event_name',data);
```
在适当的时候

### 三、eventproxy实现并发的原理

#### 1.不使用evenproxy的时候如何实现并发控制

比较经典的做法就是使用计数器控制并发，比如一共有3个并发线程，希望3个并发线程都完成之后才进行下一步操作，那么可以这么做

```bash
var count=0;
var result={};
$.get("http://example.com/1.html",function(data){
    result.data1=data;
    count++;
    handler();
});
$.get("http://example.com/2.html",function(data){
    result.data2=data;
    count++;
    handler();
});
$.get("http://example.com/3.html",function(data){
    result.data3=data;
    count++;
    handler();
});
function handler(){
    if(count==3){
        var html=getHtml(result);
        return html;
    }
}
```

#### 2.如果使用eventproxy该如何做

```bash
var ep=new eventproxy();
    ep.all('event1','event2','event3',function(data1,data2,data3){
    var html=getHtml(data1,data2,data3);
});
$.get("http://example.com/1.html",function(data){
    ep.emit('event1',data);
});
$.get("http://example.com/2.html",function(data){
    ep.emit('event2',data);
});
$.get("http://example.com/3.html",function(data){
    ep.emit('event3',data);
});

```



## 四、API

- on/addListener，绑定事件监听器 
- removeListener，移除事件的监听器 
- removeAllListeners，移除单个事件或者所有事件的监听器 
- emit，触发事件 
- once，绑定只执行一次的事件监听器 
- immediate，绑定事件，并立即触发它，别名asap 
- all，绑定一些事件,当所有事件执行后，将执行回调方法，别名assign 
- fail，error事件处理 
- tail，与all方法比较类似，都是注册到事件组合上。不同在于，指定事件都触发之后，如果事件依旧持续触发，将会在每次触发时调用handler，别名assignAll/assignAlways 
- after，事件执行N次后，在执行回调函数 
- any，事件执行完成后，只执行一次回调函数 
- not，当事件名称与绑定的绑定事件名称不符时，才执行回调函数 
- done，error事件处理 
- create，创建EventProxy

## 五、目标程序实现

1.重复异步协作

此处以读取目录下的所有文件为例，在异步操作中，我们需要在所有异步调用结束后，执行某些操作。

```bash
var ep = new EventProxy();
ep.after('got_file', files.length, function (list) {
  // 在所有文件的异步执行结束后将被执行
  // 所有文件的内容都存在list数组中
});
for (var i = 0; i < files.length; i++) {
  fs.readFile(files[i], 'utf-8', function (err, content) {
    // 触发结果事件
    ep.emit('got_file', content);
  });
}
```
after方法适合重复的操作，比如读取10个文件，调用5次数据库等。将handler注册到N次相同事件的触发上。
达到指定的触发数，handler将会被调用执行，每次触发的数据，将会按触发顺序，存为数组作为参数传入。

2.持续型异步协作

此处以股票为例，数据和模板都是异步获取，但是数据会是刷新，视图会重新刷新。

```bash
var ep = new EventProxy();
ep.tail('tpl', 'data', function (tpl, data) {
  // 在所有指定的事件触发后，将会被调用执行
  // 参数对应各自的事件名的最新数据
});
fs.readFile('template.tpl', 'utf-8', function (err, content) {
  ep.emit('tpl', content);
});
setInterval(function () {
  db.get('some sql', function (err, result) {
    ep.emit('data', result);
  });
}, 2000);
```
tail与all方法比较类似，都是注册到事件组合上。不同在于，指定事件都触发之后，如果事件依旧持续触发，将会在每次触发时调用handler，极像一条尾巴。

3.异常处理

EventProxy经过很多实践后，提供了优化的错误处理方案：

```bash
exports.getContent = function (callback) {
 var ep = new EventProxy();
  ep.all('tpl', 'data', function (tpl, data) {
    // 成功回调
    callback(null, {
      template: tpl,
      data: data
    });
  });
  // 添加error handler
  ep.fail(callback);

  fs.readFile('template.tpl', 'utf-8', ep.done('tpl'));
  db.get('some sql', ep.done('data'));
};
```





