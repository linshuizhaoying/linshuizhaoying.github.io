---
layout: post
title: 最近做的项目小结以及一些私货
tags: [vue.js,私货,项目小结]
keywords: vue.js,私货,项目小结,资源,前端
description: 
---

## 正文

### 最近在用vue.js做项目，用到现在感觉vue真的非常非常棒。然后期间我又学会一点小技巧和处理了一些小BUG,在这里分享一下。还有就是文章中会放彩蛋！

### 项目小结

首先我一开始确定用vue.js做项目是因为公司需要从头积累组件，而vue在我眼里是最好的选择。

期间我有意识的注意了代码的耦合情况，尽量将组件单纯的抽离。但是因为一些业务逻辑，导致部分还是有些耦合。

先来谈一点，如何把ajax中的获得的数据赋值给上个作用域中的变量。


```
          var result = []; 
	      $.ajax({
	            type: "GET",
	            url: "http://localhost:3317/db",
	            dataType: "json",
	            success: function(data) {
	             
							 result = data; 
	             localStorage.Cache = JSON.stringify(data); 
	            },
	            error: function(jqXHR){
	               console.log("发生错误：" + jqXHR.status);
	            }
	        });
```
一开始因为我基础不牢，然后我想用闭包好像能解决，但是并不能。闭包不是这么用的。后来发现是参数问题。
正确的代码是这样：

```
	   var result = []; 
	      $.ajax({
	            type: "GET",
	            url: "http://localhost:3317/db",
	            dataType: "json",
	            async:false,//为了把数据赋值给上个作用域，需要同步进行
	            success: function(data) {
	             
							 result = data; 
	             localStorage.Cache = JSON.stringify(data); 
	            },
	            error: function(jqXHR){
	               console.log("发生错误：" + jqXHR.status);
	            }
	        });

```
对，一开始想偏了，后来想到之前闭包看过一个例子就是for执行完了但是最后给内容赋值是for之后的参数。因此想到了可能是异步出了错。但是只是有点萌芽并没有解决方案，因此搜索了一下。得到完美解决。

#### 模块化

vue.js给我最大的感受就是以后组建自己的组件库非常方便。把所有组件该隐藏的隐藏，该暴露的暴露。拒绝耦合（但实际上这很难）。最终抽离。可能这几块还需要多看别人的东西0-0。

### 前端mock

这是重头戏。好像有Mock.js这回事。但是粗略看了眼好像太重了。

事实上我接触Mock这个概念还是知乎上大V关注的话题。

mock无需等待，让前端独立于后端进行开发。这个概念在接触MOCK之前我也有过想法，就是自己写一个应用，能够根据拖拽的字段自动生成需要的json数据，而且符合项目的中文字符串,图片,以及特定长度的字符都能够根据字典生成。这个工具做细了感觉还是能加快开发效率的-0-不过现在还没开坑。

不过在这次项目中，我是想直接用收购构建json格式数据并且本地加载。但是问题来了：
    
    问题1：如果不搭建服务器，那么用ajax加载json会让浏览器报错
    问题2：搭了node.js环境，会发现跨域
    问题3：换成php环境，还是跨域
    问题4：跨域想到jsonp但是我们写的是json而且我们并不打算后天用php改。只改.jsop为jsonp还是报错-0-
    
于是我把自己屯的资料扫了一遍，找到一个非常非常好用的东西，它叫做`json server`。

注意需要Node.js环境。

这是[github地址](https://github.com/typicode/json-server)

![imgn](http://7s1say.com1.z0.glb.clouddn.com//active39.gif)

它可以完美解决跨域以及环境问题，更重要的是它可以用到时直接调用。支持基本的route:

```
GET    /posts
GET    /posts/1
POST   /posts
PUT    /posts/1
PATCH  /posts/1
DELETE /posts/1
```

但是官方版本有个问题，就是它端口固定死为3000，我们如果想对多个json进行调用还是不可以的。

因此需要自己去修改。
以下是我的修改方案，可以对照着来。我是mac，因此先查一下它安装路径。

```
 which json-server
```
跳到给出的目录，然后因为是快捷键邮件查看详情然后会获取到真实目录。

找到其中一个Index.js文件，看到

```
var updateNotifier = require('update-notifier')
var yargs = require('yargs')
var run = require('./run')
var pkg = require('../../package.json')


```

开头的文件就是。

```
.options({
      port: {
        alias: 'p',
        description: 'Set port',
        default:3000
       },
```

发现写死的端口，只需要写个函数自动生成指定范围的端口号就行。

```
  updateNotifier({ pkg: pkg }).notify();
  function getport(max ,min)
  {
    var a = parseInt(Math.random()*(max-min+1)+min,10);
    var b = Math.floor(Math.random()*(max-min+1)+min);
    
    return b;
  }
  var randomPort = getport(3000,4000);

```

在

```
updateNotifier({ pkg: pkg }).notify();
```
下面加入上面生成函数。
然后用randomPort替换掉3000。

这样就能在运行的时候自动监听3000-4000之间的任意一个端口.这样很大程度可以避免重复。当然如果你遇到千分之一的几率也没办法0-0，只能结束进程重新生成。

## 总结

先总结到这里。对你有帮助请到评论区手动写赞=-=。