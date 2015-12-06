---
layout: post
title: 从零打造自己的弹幕插件(基础版)
category: 技术
tags: [js,vue.js,css3,jquery,基础,积累]
keywords: 前端,资料,学习,javascript
description: 
---

## 前言

昨天下午提出的需求是制作一个弹幕组件，花了半天完成了基础版本以及一些高级扩展。这个系列将一直更新下去，说不定还能造出个小轮子什么的。。。

## 正文

基础版本的思路借鉴于网上的版本。我们这里先理清一下这种写法的弹幕插件的思路。因为后期我们可能会因为一些需求进行重构逻辑以及优化代码。

大概是这样的：

```
1.通过javascript的原型继承特性来构造基础的单个弹幕。
2.需要一个弹幕的舞台(这个一开始为了测试是先添加了一个div，但是后期为了即插即用的组件会把这块也写到代码里)
3.配置一个支持事件机制的对象，利用它来监听单个弹幕配置以及触发动作。
4.一开始弹幕都将隐藏于舞台右侧(然而我后面添加的一些高级扩展是改变了单一的动画方式。)
```

事实上一个弹幕的表面形式应该是这样的:

```
1.新增一个弹幕，从配置中读取文本，字体大小，颜色，动画时间(这决定了它移动的速度，因为弹幕速度最好相同。因此后期会增加默认值。)。
2.一个伪队列，然后根据配置中的播放时间和位置弹幕。
```

因为我后期业务是结合vue来写了，因此这里放出beta可独立引用版本，直接保存为Js引用即可.后面的很多扩展都会根据该版本。不过再后期的一些逻辑修改可能就会改的面目全非TvT。

```
  		var that = this;
			//舞台是全局变量
			var stage = $('#Barrage');
			//弹幕的总时间,这个是值得思考的问题，根据业务而已，这个不应该是一开始写死，因为是动态的弹幕，不过这里是为了测试方便，后面会修改
			var totalTime = 9000;
			//检测时间间隔
			var checkTime = 1000;
			//总飞幕数
			var playCount = Math.ceil(totalTime / checkTime);

			//构造一个单独的弹幕
			var BarrageItem = function(config){
				//保存配置
				this.config = config;
				//设置样式，这里的样式指的是一个容器，它指包含了单个弹幕的基础样式配置的div
				this.outward = this.mySelf();
				//准备弹出去，先隐藏再加入到舞台，后面正式获取配置参数时会把一些样式修改。
				this.outward.hide().appendTo(stage);
			}

			//单个弹幕样式,从config中提取配置
			BarrageItem.prototype.mySelf = function(){
			//把配置中的样式写入
				var outward = $('<div style="min-width:400px;font-size:'+this.config.size +'px;color:'+this.config.color+';">'+this.config.text+'</div>');
	    	return outward;
			}

			//定义弹的过程，这是弹幕的核心，而且一些高级扩展也是在这里添加
			
			BarrageItem.prototype.move = function(){
				var that = this;
				var outward = that.outward;
				var myWidth = outward.width();
				//用jq自带animate来让它运动
				outward.animate({
					left: -myWidth
				},that.config.duration,'swing',function(){
					outward.hide(); //弹完我就藏起来
				});
			}

			//开始弹弹弹

			BarrageItem.prototype.start = function(){
				var that = this;
	    	var outward = that.outward; //这里引用的还是原型中的那个outward
				//开始之前先隐藏自己
				outward.css({
					position: 'absolute',
					left: stage.width() + 'px', //隐藏在右侧
					top:that.config.top || 0 , //如果有定义高度就从配置中取，否则就置顶
					zIndex:10,//展示到前列
					display: 'block'
				});

				//延迟时间由配置的开始时间减去队列中该弹幕所处的位置所需要等的位置，而这里的队列位置是由驱使者diretor分配的，事实上根据我的调试发现这种写法只能近似于模仿顺序，然而如果两个播放时间间隔不大将会同时出发，不过这个对于普通体验影响不大。后期如果有强需求可能需要把整个逻辑改掉
				var delayTime = that.config.time - (that.config.queue - 1) * checkTime;
				setTimeout(function(){
					that.move();
				},delayTime);

			}

			//设置一个支持事件机制的对象，也就是弹幕们的驱使者，它来驱使弹幕弹弹弹
			
			var diretor = $({});//创建一个空的对象

			//对舞台进行样式设置，其实可以直接写到css里面
			stage.css({
			    position:'relative',
			    overflow:'hidden'
			});
			
			//批量读取写好的弹幕配置内容，然而后期是需要动态弹幕，打算采用websocket来分配因此这里也只是为了测试而简写
			
			//that.messages 是配合vue的data来设置的，如果是为了在单个文件中引用，去掉that,把message写在该js里面
			
			$.each(messages,function(k,config){
				//确认弹出的时间
				var queue = Math.ceil(config.time / checkTime);
				config.queue = queue;

				//新建一个对象给它配置
				var go = new BarrageItem(config);
				//驱动者监听驱使动作
				diretor.on(queue+'start',function(){
					go.start();
				})
			});

			var currentQueue = 0;
			setInterval(function(){
			    //从队列中取第n个开始谈
			    diretor.trigger(currentQueue+'start');
			    //如果都弹完了 循环来一遍
			    if (currentQueue === playCount) {
			        currentQueue = 0;
			    }else{
			        currentQueue++;
			    }

			},checkTime);

```

看到这里你会发现核心代码其实蛮简洁的，的确我们只需要熟悉原型继承，属性Jquery的一些常规函数调用就能完成一个简单的弹幕。更多需要我们发挥的其实还是高级的扩展。

我们看下对于基础版本的弹幕配置应该怎么写：

```
     messages:
				     [{
				    //从何时开始
				    time:0,
				    //经过的时间
				    duration:4292,
				    //舞台偏移的高度
				    top:10,
				    //弹幕文字大小
				    size:16,
				    //弹幕颜色
				    color:'#000',
				    //内容
				    text:'前方高能注意'
				},{
				    //从何时开始
				    time:100,
				    //经过的时间
				    duration:6192,
				    //舞台偏移的高度
				    top:100,
				    //弹幕文字大小
				    size:14,
				    //弹幕颜色
				    color:'green',
				    //内容
				    text:'我准备追上前面那条',
				},{
				    //从何时开始
				    time:130,
				    //经过的时间
				    duration:4192,
				    //舞台偏移的高度
				    top:130,
				    //弹幕文字大小
				    size:16,
				    //弹幕颜色
				    color:'red',
				    //内容
				    text:'遮住遮住遮住。。',
				},{
				    //从何时开始
				    time:1000,
				    //经过的时间
				    duration:6992,
				    //舞台偏移的高度
				    top:250,
				    //弹幕文字大小
				    size:20,
				    //弹幕颜色
				    color:'blue',
				    //内容
				    text:'临水照影Testing....～～',
				}]

```

我们可以看到对于基础弹幕我们和代码里的配置已经对应上了。后面的扩展也是同理。

然后就是简单的测试html页面:

```

css:
#Barrage{
	width:800px;
	height:400px;
	margin:0 auto;
	border:1px solid #000;
}

html:
<div id="Barrage"></div>

```

然后我们看下演示效果:

![imgn](http://haoqiao.qiniudn.com/active58.gif)

哦，我贴的是高级扩展的，基础的就只有往前滚动的效果。

对于高级扩展我现在完成的是这几种:

![imgn](http://haoqiao.qiniudn.com/42C4E611-EE01-4890-BA59-63A88BE2EAB0.png)

## 结尾

静下心继续学习-0-


