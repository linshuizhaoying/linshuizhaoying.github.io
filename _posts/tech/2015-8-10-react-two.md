---
layout: post
title: React从入门到上手三部曲(2)
category: 技术
tags: [react，react学习,react实例,react入门,react总结]
keywords: 学习资料，程序语言
description: 
---

## 正文
本文来打造一个基于Node.js+Express4.x的React组件应用。刚开始我们先设定一下我们需要做的事情。首先这个应用是基于爬虫原理的个人每日资讯。类似今日头条这个应用。我们需要两个组件，一个是父组件，它接受传来的URL并解析爬取内容，一个是子组件，它根据父组件传来的参数进行显示（包括显示颜色，内容）。
我们给两个组件命名为NewsComponent 和 NewsItem。

### 准备

我们直接新建一个开发环境，可以直接clone我github上的开发环境。
在根目录运行
    
    supervisor ./bin/www
    监视文件改动
    gulp
    自动构建
之后在public/javascript/src/下建立两个新文件
一个命名为newsComponent.jsx 一个命名为newItem.jsx
然后我们先来做个组件嵌套的测试。

在newItem.jsx中我们写入：

    var React = require('react');
	module.exports = React.createClass({
	render: function() {
    	return (
    	<h1>这是子项</h1>
    	)
	}
	});
在newsComponent.jsx 中写入

	var React = require('react');
	var NewsItem = require('./newsItem.jsx');
	module.exports = React.createClass({
	render: function() {
    	return (
    		<div>
    	 	 <NewsItem/>
    	  	<h1>666 linshuizhaoying!</h1>
    		</div>
    	)
	}
	});
记得如果gulp和supervisor如果因为写的代码有bug崩掉要手动重启。
之后我们访问127.0.0.0:3000就能看到我们的测试结果。

![img1](http://img.haoqiao.me//react01.png?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
看示例图中下方显示就是我们要做的组件的大概界面。因此我们需要现在index.jade中写入：

      .portlet.box.blue
        .portlet-title
          .caption
            i.fa.fa-cogs
            | 今日资讯
          .tools
            a.collapse(href='javascript:;', data-original-title='', title='')
        .portlet-body
          .panel.panel-primary
            .panel-heading
              h3.panel-title Primary Panel
            .panel-body
              span Panel content
然后在layout.jade里引入我们的css文件，我这里引入了Boosstrap.min.css和我模板中的component.css，界面显示的css代码可以去网上抄或者自己写。
分析代码我们可以看出非常简洁明了的能将二者分离并重组。因此我们开始用react来构造这个组件。
### React组件构造
对于newItem.jsx的改造：

    var React = require('react');

	module.exports = React.createClass({
	render: function() {
    return (
		<div className="panel panel-primary">
			<div className="panel-heading">
				<h3 className="panel-title">这是子项的标题</h3>
			</div>
			<div className="panel-body">
				<span>这是子项的内容</span>
			</div>
		</div>
       	  )
	}
	});
对于newsComponent.jsx的改造：
    
	var React = require('react');
	var NewsItem = require('./newsItem.jsx');

	module.exports = React.createClass({
	render: function() {
	return (
		<div className="portlet box blue">
			<div className="portlet-title">
			<div className="caption">
				<i className="fa fa-cogs"></i>
				 今日资讯
				</div>
			</div>
			<div className="portlet-body">
			   <NewsItem/>
			</div>
		</div>  
    );
	}
	});
   
此时如果代码无误的话我们刷新浏览器就能看到我们的组合组件。
![img2](http://img.haoqiao.me//react02.png?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
事实上我们在测试前用jade写好效果，然后直接用chrome分析节点复制，然后把class替换为className就可以了。

### 界面参数的传递
    
我们需要的是一个可复用的组件，因此，界面的style不能写死，而是在初始化的时候赋值调用，首先根据component.css
父组件有以下几种颜色：
    
    grey-cascade
    purple
    red
    blue
    green
    yellow
而子组件有以下几种分类
    
    default
    primary
    success
    info
    warning
    danger
属性的赋值应该在初始化赋值给props，因为state是来记录自身变化的。
下面我们来写参数传递的代码，首先在app.jsx里面把render内容换掉

    React.render(
      <NewsComponent component_style="green" item_style="info" url="www.baidu.com"/>,
      document.getElementById('example1')
	);
	React.render(
      <NewsComponent component_style="blue" item_style="danger" url="www.google.com" />,
      document.getElementById('example2')
	);
来到newsComponent.jsx

    render: function() {
	  	//取得属性值
		var url = this.props.url;
		var component_style = "portlet box " + this.props.component_style;
		var item_style = this.props.item_style;
    return (
			<div className={component_style}>
				<div className="portlet-title">
					<div className="caption">
					  <i className="fa fa-cogs"></i>
					  今日资讯
					</div>
				</div>
				<div className="portlet-body">
				    <span>{url}</span>
					  <NewsItem item_style={item_style}/>
				</div>
			</div>  
    );
	}
接着修改newItem.jsx
    
	render: function() {
	  var item_style = "panel panel-"+this.props.item_style;
    return (
		<div className={item_style}>
			<div className="panel-heading">
				<h3 className="panel-title">这是子项的标题</h3>
			</div>
			<div className="panel-body">
				<span>这是子项的内容</span>
			</div>
		</div>
    )
	}
	
	
然后我们刷新浏览器，我们能看到如下图
![img3](http://img.haoqiao.me//%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202015-08-10%2003.23.58%20PM.png?imageView2/2/w/500/h/500/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
接下来就是对url这个值进行处理，这已经不是React的范围，因此本章结束。
