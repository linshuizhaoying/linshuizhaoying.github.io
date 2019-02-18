---
layout: post
title: Sprite动画与ScrollMagic的使用
tags: [Css3,Sprite,ScrollMagic]
keywords: 学习资料,前端,Css3
description: 
---
## 写在前面
这几天没怎么写东西，光忙着翻译国外的优质文章了，对于我来说，收获是极大的。尤其是干货满满的文章，翻译的时候真的能感受到作者的所付出的心血。因此我积压了还有十几篇的文章=-=还有好多没加进来的优质文章。言归正传，昨天翻译了Tom Bennet的雪碧图动画。觉得挺有收获，因此按照作者的给的彩蛋延伸开来，发现了一个新世界。


## 正文

首先我在翻译的时候发现作者写的雪碧图动画好像很理所当然，好像很顺利。于是我自己去GOOGLE了一堆Sprite图。经过几个小时的实践。我得到了以下结论：
    
    1.童话里都是骗人的。童话里都是骗人的。童话里都是骗人的。重要的事情说三遍。尝试了很多种雪碧图来做连贯的动画，最后我发现，想要利用雪碧图做连贯的动画，首先你要有图。
    2.文中使动画连贯起来的重点是自己调好 animation的参数。
    举个例子，文中
       animation: sprite 3.5s steps(45);
    他是一个60张连贯的PNG图组合的，因此45/3.5=30 也就是一秒两张的播放。这样连贯性才能保证。
    =-=但是60张连贯的雪碧图我至今只看到国外一个设计师自己画的猫咪跑。。。因此做连贯性的雪碧图动画代价是蛮大的。
    
接下来我们介绍今天的主角：ScrollMagic

### ScrollMagic

ScrollMagic是一个非常棒的库，它能帮助你处理用户当前滚动的位置。现在网上很多根据滚动条滚动的动画效果就是基于它。很多产品展示站都有它的影子在。

来看个典型的例子

![magic1](http://img.haoqiao.me//scrollmagic1.gif)
 
它可以做到钉帧spnning效果。如此炫酷的效果却只需要短短几行代码

		var scene = new ScrollMagic.Scene({triggerElement: "#trigger1", duration: 300})
						.setPin("#pin1")
						.addIndicators({name: "1 (duration: 300)"}) // add indicators (requires plugin)
						.addTo(controller);

它还能配合svg做动画效果，以及配合雪碧图动画等等。

我们来看个例子

由于某些同学不喜欢codepen，所以我例子放在了jsfiddle里。[Click here](http://jsfiddle.net/linshuizhaoying/z16u2wjt/)

国内关于ScrollMagic的详解基本没有，官方文档也只是给例子。因此我这边例子举的国外某位大神写的文章中的引用。你可以[点这里](https://scotch.io/tutorials/building-interactive-scrolling-websites-with-scrollmagic-js)看原文。

他写的比我写的好很多，我之所以厚脸皮写中文版的，一个是为了加深印象，一个是为了给懒得看英文原文的同学方便。

从例子中我们可以看到，它先定义了一个scrollMagicController，然后再定义一个scene，然后再载入tween插件，最后加一个演示指标到页面。

我们来总结一下，一个基础的scrollMagic的用法：

    1.创建一个scrollMagic控制器
    2.创建一个动画对象
    3.创建一个舞台对象
    4.将动画对象加到舞台
    5.将舞台放到控制器内。
    
一个动画对象是需要插件的，scrollmagic用的是gsap。我原本想去深入学习下的。。看到官网的演示我被吓退了。。。

不过一个简洁的我们还是可以探讨下的，设置对应ID的动画。

      var tween = TweenMax.to('#animation', 0.5, {
        backgroundColor: 'rgb(255, 39, 46)',
        scale: 5,
        rotation: 360
      });

然后是最后几步

        // Create the Scene and trigger when visible with ScrollMagic
        var scene = new ScrollScene({
            triggerElement: '#scene',
            offset: 150 /* offset the trigger 150px below #scene's top */
        })
        .setTween(tween)
        .addTo(scrollMagicController);
        
我们可以看到例子还是很简单的。。。

我们将动画中的offset改为duration这个属性。
这代表这将动画绑定到了scroll上，滚动的距离就是播放对应的帧数。这样你回滚也可以看到动画在回放。两者差别你可以从动画效果中看出。[click here](http://jsfiddle.net/linshuizhaoying/a62Ly4hc/)

这样你是不是觉得非常简单就能做到自己想要的效果，如果想要改变更多属性，你可以直接去看gsap的例子，我这边找到了对应的css插件，你可以从里面挑选想要修改的属性[click here](http://greensock.com/docs/#/HTML5/GSAP/Plugins/CSSPlugin/)

tween.to你可以联想到有to就有from，对，它还有个属性叫做from，效果和to是相反的。你可以点这里的例子查看.[click here](http://jsfiddle.net/linshuizhaoying/6e0homcu/).

如果机智的你发散思维，你会想到，有了to有了from，那么一定有组合的fromto吧。是的，它还有个属性叫做fromto，效果你可以点[这里](http://jsfiddle.net/linshuizhaoying/foagpapz/)

代码在这里：

    var tween = TweenMax.fromTo('#animation', 0.5,
        {
            backgroundColor: 'rgb(255, 39, 46)',
            scale: 5,
            left: -400
        },
        {
            scale: 1,
            left: 400,
            rotation: 360,
            repeat: -1, /*无限循环 */
            yoyo: true /* 是否回转 */
        }
    );

### 组合

单一动画会做后，你该了解下如何将动画组合。

官网上的写法是这样的：

    var tween = new TimelineMax()
				.to("#animate2", 1, {top: "-=200",
						onStart: function () {$box.html("This");},
						onReverseComplete: function () {$box.html("Let's draw!");}
					}
				)
				.to("#animate2", 1, {top: "+=200", left: "+=200",
						onStart: function () {$box.html("is");},
						onReverseComplete: function () {$box.html("This");}
					}
				)
				.to("#animate2", 1, {top: "-=200",
						onStart: function () {$box.html("the");},
						onReverseComplete: function () {$box.html("is");}
					}
				)
				
				
直接将多个动画写成链式。

如果是相同动画想要按次序播放你可以这么写：
    
    <div class="col-xs-3">
      <div class="wrap">  
        <h2>Magic Happens Here</h2>
        <div class="animation"><i class="fa fa-heart"></i></div>
      </div>
    </div>
    <div class="col-xs-3">
      <div class="wrap">  
        <h2>Magic Happens Here</h2>
        <div class="animation"><i class="fa fa-heart"></i></div>
      </div>
    </div>
    <div class="col-xs-3">
      <div class="wrap">  
        <h2>Magic Happens Here</h2>
        <div class="animation"><i class="fa fa-heart"></i></div>
      </div>
    </div>
    <div class="col-xs-3">
      <div class="wrap">  
        <h2>Magic Happens Here</h2>
        <div class="animation"><i class="fa fa-heart"></i></div>
      </div>
    </div>


    var tween = TweenMax.staggerFromTo('.animation', 0.5,
    {
        scale: 1,
    },
    {
        backgroundColor: 'rgb(255, 39, 46)',
        scale: 5,
        rotation: 360
    },
    0.4 /* 间隔的duration */
    );
    
你可以点[这里](http://jsfiddle.net/linshuizhaoying/so730Loz/1/)看效果

### 触发class

比如你想让在某个场景出现时触发某些动画，你可以用setClassToggle这个方法。

举个例子：
    
     <div class="col-md-6 col-md-offset-3">
      <div class="wrap" id="scene-1">  
        <h2>Magic Happens Here</h2>
        <div class="animation" id="animation-1"><i class="fa fa-heart"></i></div>
      </div>
    </div>
    
    
    
      var tween1 = TweenMax.to('#animation-1', 0.3, {
          backgroundColor: 'rgb(255, 39, 46)',
          scale: 10,
          rotation: 360
        });
        var scene1 = new ScrollScene({
          triggerElement: '#scene-1',
          offset: 50
        })
        .setClassToggle('body', 'scene-1-active')
        .setTween(tween1)
        .addTo(scrollMagicController);
        
想看具体效果点[这里](http://jsfiddle.net/linshuizhaoying/yLymhbL1/)

### 移动端

在移动端，当scroll停止时它可能都没检测到事件。不过有个解决方法就是将设置每个元素overflow: scroll，这样就能让设备时时刻刻接受到事件。

### 子区域使用scrollmaic

你可能不想在整个body中使用scroll，因此你可以这么做：
    
    var scrollMagicController = new ScrollMagic({container: '#my-container'});
    
移动端也支持这种方式。

但这个是中hack，因此你可能需要增加一个css3属性
    
    -webkit-overflow-scrolling: touch;
    
然后有个悲剧就是不是所有浏览器都支持这个属性-0-不过你可自己试试对吧。

### 在移动端禁用

    两种方式：
    如果你加载了Modernizr.js
        if (!Modernizr.touch) {
         // Start ScrollMagic code
         }
     如果你没加载
        if (!is_touch_device()) {
          // Start ScrollMagic code
       }
       function is_touch_device() {
         return 'ontouchstart' in window // works on most browsers 
             || 'onmsgesturechange' in window; // works on ie10
       };
  
    
## 结尾
  ScrollMagic开辟了一种网页风格，我个人觉得还是很有必要去学习并应用到实践中。