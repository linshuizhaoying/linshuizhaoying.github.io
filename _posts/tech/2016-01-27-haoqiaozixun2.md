---
layout: post
title: 打造支持代码插件的爬虫咨询应用
tags: [vue.js,私货,项目小结]
keywords: vue.js,代码插件,爬虫,咨询,前端,eval
description: 
---

# 前言

前端时间在博客里讲过手里在重构一个项目，其实已经脱离重构的地步了。。。已经重写了，这个应用是我用来爬取自己以前积累的一些网站的内容，当做每日咨询。但是很早以前因为技术水平和眼界的关系，导致第一版的功能只满足了一小部分，而且逻辑方式有些错误。那个时候是用react来写的，第一次组件化的经历。不过爬取方式是每打开页面一次去爬一次。这造成了很大的浪费，而且我都不敢公布出来-0-怕服务器崩了。所以那个时候就有了以后要重构的想法。前一个月前开始了自己项目的计划，于是开始慢慢重新写。这篇文章将是新年之前的最后一篇了，其它的小文我都积累到gitbook到时候一并放出来。

# 正文

重构版本的技术栈如下:

```

1.express
2.vue.js
3.webpack

```

提到之前的第一版本的每日资讯应用，它的缺陷有

```
1.不支持后续添加网站，因为一开始都写死了。
2.不支持服务端自动根据任务爬取而是每次打开页面都去爬一次，这造成了极大的浪费。
3.不支持收藏功能。即阅后收藏。

```
因此新开发的应用我用思维导图构建了如下：

```

好巧咨询
	1.定时抓取函数。利用爬虫抓取页面数据，保存到数据库
	2.访问页面的时候先调用check函数。查看最近获取的数据并返回。
	自动更新，每天早上2点 6点 10点14点16点自动更新 20点
	3.添加新的解析模块时自动执行爬虫爬取
	4.数据库
		recent 用来保存最新爬的数据 包含 title name link created-time 
		like用来保存日常点赞的内容
		list 用来保存每个站的名称和地址，新增解析内容时候自动添加
		
```

其实亮点主要是它支持爬虫代码的解析和插入。我们来看下实体图就明白了:

![imgn](http://img.haoqiao.me/hqzx1.png)

可以随时更新或者添加新的爬虫代码。如果目标网站结构变化了,只需要简单修改下爬虫插件里面的部分代码就能重新解析了。

这个功能是如何实现的呢？

很简单，我用上了`魔鬼eval`。这是我在翻阅第二版高程的时候突然有的想法。执行动态代码不仅仅只有eval这种方法，但是eval明显是最合适的。因为它支持上下文环境的替换。

但是需要注意的是你eval里面包含的代码必须压缩成一行，不然你会遇到很多奇怪的问题。为了解决这个问题我引入了`var jsmin = require('jsmin2')`

数据库方面用到三张表，`liked` `recent` `site`
都是用Mongodb来快速构建的。

爬虫方式选择的还是和原来一样，用到了`superagent` `cheerio`.直接放出核心代码:

```

function excuteCode(url,code,type){
		superagent.get(url)
	  .end(function (err,response) {
	  	if (err) {
	     return next(err);
	    }
		  var $ = cheerio.load(response.text); //获取文本
		 // console.log(cheerio.load(res.text));
      var data_result = new Array();
		 	eval(code);
		 	if(data_result.length > 0){
		 		for (var i = 0; i < data_result.length; i++) {
		 			insertItem(data_result[i],type);
		 		};
		 	}else{
		 		console.log("抓取出错,网络可能有问题。");
		 	}
	  });

 }

```

之后完成雏形之后想到，得搭建一个测试平台来写最基础的爬虫插件。于是开发了如下平台

![imgn](http://img.haoqiao.me/hqzx2.png)

它的核心代码如下:

```

//测试平台
router.post('/testing', function(req, res,next) {
	var temp = req.body;
	var url = temp.url;
  var cacheCode = jsmin(temp.code).code; //通过压缩代码片段使其能在eval中执行
  console.log(cacheCode);
	var data_result = new Array();

	superagent.get(url)
	.end(function (err,response) {
		  if (err) {
			res.send(err);//输出结果
	    res.end();
	 //  return next(err);
	  }
	  console.log("this is responese:" + response);
	  if(response.text.length < 0){
		   res.send("抓取失败，请重试!");//输出结果
		   res.end();
	  }else{
			var $ = cheerio.load(response.text);
	//		console.log(cacheCode);
		 	function getcache(){
		 	 eval(cacheCode); //执行代码插件
		   res.send(cacheCode + JSON.stringify(data_result));//输出结果
		   res.end();
		 	}
		  
		  getcache();//执行并输出结果
	  }


	});
});

```

最后要解决的是服务器端能够每日定时去抓取。google了一下找到了`schedule`,它的用法很简单

```

var schedule = require('node-schedule');
var rule = new schedule.RecurrenceRule();
rule.dayOfWeek = [0,1,2,3,4,5,6,7];
rule.hour = [0,4,8,12,16,20,23];
rule.minute = 0;

var linshui = schedule.scheduleJob(rule, function(){
  getAlldata();
  console.log("任务正在执行中..." + "hour" + rule.hour);
});

```

重要的思路其实是要把这段代码放在`app.js`，也就是我们express一开始执行的文件。但是很多方法都是直接写在了路由里。因此需要单独把爬取数据里网站的代码独立出来。

在`app.js`同级目录下新建`getAlldata.js`，里面这么写:

```
var cheerio = require('cheerio');
var superagent = require('superagent');
var jsmin = require('jsmin2')

var Sites = require('./data/site');
var Recent = require('./data/recent');
var Liked = require('./data/liked');
function getAlldata(){
	//爬虫代码...
}

module.exports = getAlldata;

```

然后在app.js里面添加

```

var getAlldata = require('./getAlldata');

```

最后效果图如下：

![imgn](http://img.haoqiao.me/active77.gif)


# 总结

其实个人项目开发主要难点技术与界面是对半开的。我写界面也耗了不少时间，这界面是仿买的一本书里面看到的一个网站。有时候看上去简介的页面要废的功夫可能更多。写完这个项目之后我就可以把愉快的把时间放在gitbook上面了。

最后，大家新年快乐。

