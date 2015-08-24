---
layout: post
title: Node.js + React.js 开发评论插件系统
category: 技术
tags: [react，react学习,react实例,react入门,react总结]
keywords: 学习资料，程序语言
description: 
---

## 前言

一开始是因为Jekyll的评论插件如果是放到github还需要分支什么的，后来我用了多说系统，但是它页面每次只加载一次，导致如果页内跳转，转到的页面是没有评论的。因此萌生了自己开发一个评论插件系统，通过跳转将参数传来，事实上只需要在加载的前的js写好，可以达到和多说系统一样的外置功能，但是处理起来比较麻烦，因此我选择了直接传参。其实一开始我像选择用react.js+vue.js来开发的，但是用React写完后发现没vue啥事。。。毕竟同是view层，React用的顺手，因此我打算下一个大项目中用vue.js做前端，react做后台。

## 正文

由于本篇并不是在开发中写的，而是一篇总结，因此很多东西我都会跳过不说，只讲重点的，工程我已经放到了github上。有兴趣的可以点[这里](https://github.com/linshuizhaoying/commets.haoqiao.me)

首先我们来看下界面![gif1](http://7s1say.com1.z0.glb.clouddn.com//test.gif)
因此我们可以先确定要构建的组件
    
    1.Content 用来作为包含内容的父组件
    2.Comments 用来作为一个评论组合的容器
    3.Admin 用来作为管理回复
    4.SubmitComment 发布评论
    
其实官网是有个评论的例子，但是-0-我之前写的时候不知道。之后查资料的时候发现了=-=，整个人斯巴达，因为很多设计都是类似的。。。不过由于是之前一手设计的，因此我在开发途中大改了好几次。因为一开始是没有总设计稿，因此我的设计步骤是这样的。
    
    1.设计评论列表静态布局
    2.转为react
    3.添加动态效果
    4.添加测试数据
    5.设计发表评论组件
    6.设计动态效果
    7.后台设计数据库
    8.后台添加代码
    9.前后台沟通
    
### 代码分析
因为这篇是总结类型，我就挑着讲了。
首先从需要渲染的页面index.jade来讲。关键代码如下：
    
    span#name.hide #{name}
    span#link.hide #{link}
    span#adminpass.hide #{adminpass}
    span#lock.hide #{lock}
    
name和link是传过来的参数，我需要后台route拿到后重新渲染进来，方便其他组件获取这个参数。

adminpass是一个开启管理员模式的关键，后台会根据获取到的密码来判断。之所以做这个是因为懒得再写个管理页面。
lock就是是否开启回复功能，因此游客是不给回复的，组件需要判断是否是管理员，如果是才解锁。

来看content.jsx

    loadCommentsFromServer:function() {
	if($("#adminpass").text().length !=0){
    $.post("/getAllData" , {link:$("#link").text(),adminpass:$("#adminpass").text()}, function(result) {
      if (this.isMounted()) {
        this.setState({
          data:JSON.parse(result)
        });
      }
    }.bind(this));
	}else{
    $.post("/getData" , {link:$("#link").text(),title:$("#name").text()}, function(result) {
      var list = result;
      if (this.isMounted()) {
        this.setState({
          data:JSON.parse(result)
        });
      }
    }.bind(this));
	}
    },
    componentDidMount: function() {
      this.loadCommentsFromServer();
      setInterval(this.loadCommentsFromServer, 2000);
    },
    
 数据的获取是在组件渲染之前，之后再每隔2秒去获取更新。
 这个更新方法是从官网学来的
 
     我们把旧的评论数组替换成从服务器拿到的新的数组，然后UI自动更新。正是有了这种响应式，一个小的改变都会触发实时的更新。这里我们将使用简单的轮询，但是你可以简单地使用WebSockets或者其它技术。

接着我们可以看到后台是如何处理参数的

    router.get('/show/:name/:link', function(req, res, next) {
	NewsData.findOne({ link:req.body.link,title: req.body.title }, function(err, content) {
		if(content!=null){
			console.log("找到了"); // { name: 'Frodo', inventory: { ringOfPower: 1 }}
		}else{
			console.log("没找到");
		}
	});
     res.render('index', {title: '临水照影  评论系统',lock:"t",name:req.params.name,li
     nk:req.params.link});});
这是对每个网站的每个文章进行分类。数据的获取还是通过getDate这个来获得的。

    router.post('/getData', function(req, res, next) {
    //	NewsData.find({link:req.body.link}, function(err, content) {
    	NewsData.find({link:req.body.link,title: req.body.title},null,{sort:                           {_id:-1}},function(err, content){  
		if(content!=null){
		console.log("getdate");
			res.send(JSON.stringify(content));
			res.end();
			//console.log("找到了content" + JSON.stringify(content)); // { name: 'Frodo', inventory: { ringOfPower: 1 }}
		}else{
		  res.send(JSON.stringify(""));
		  res.end();
		}
	});
    });
    
这里我们可以看到数据先经过排序再通过json格式化再输出的。

##总结
使用方法就是在需要评论的地方加一个跳转。比如我在jekyll的posts.html里面是这么写的
    
       <button><a href="http://comments.haoqiao.me/show/page.title/haoqiao.me">跳到评论系统</a></button>
       
然后把代码部署到服务器就行了。其实展开讲还是挺多的，但是事情比较多，点到为止。

 