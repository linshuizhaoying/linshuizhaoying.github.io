---
layout: post
title: Icons Filling Effect (Icons 填充效果）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://codyhouse.co/gem/icons-filling-effect/)

### 作者: Sebastiano Guerriero


### 译者: 临水照影


## 正文之前

这个效果好像也是蛮早的了，不过正好要用到，于是进行翻译整理。

## 正文

为了你的Icons添加一个吸引人的填充效果，它将使单页更加的看起来酷。

有些时候你只是想要创建一些很酷的东西，或许是一些你想制作一些独特风格的单页，这个效果能帮你通过一些小图标来使你的网页更加炫酷。

灵感来源：[Elliot Condon beautiful portfolio](http://www.elliotcondon.com/)

### 创建结构

在写代码之前，我将介绍一下原理。最重要的一点是使你想要填充图标的区域是透明的。

![img1](http://7s1say.com1.z0.glb.clouddn.com//animation-2.gif)

我们的处理方式是Icons跟随页面的scroll移动，颜色层是固定不变的。

为了100%说明这一点，看下图：

![img2](http://7s1say.com1.z0.glb.clouddn.com//icons-filled-effect-animation.gif)

这说明，这个结构只是一个无序的列表，我创建两个空列表项来让顶部和底部有更多的空间。
    
    

    <ul>
    	<li class="cd-service cd-service-divider"></li>
 
	    <li class="cd-service cd-service-1">
	    	<h2>Web Design</h2>
	    	<p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Perferendis pariatur tenetur quod veritatis nulla aspernatur architecto! Fugit, reprehenderit amet deserunt molestiae ut libero facere quasi velit perferendis ullam quis necessitatibus!</p>
	    </li> <!-- cd-service -->
 
	    <li><!-- ... --></li>
 
	    <li class="cd-service cd-service-divider"></li>
    </ul>

### 添加style

我们在body中用两个伪元素`::before`和`::after`来创建作为颜色box。正如你所见，我添加了css3 过渡属性，因为我们需要用jquery在内容页用滚动的时候改变颜色。图标总是`::before`选择器来创建。
    
    body::before, body::after {
      /* the 2 underneath colored sections */
      content: '';
      position: fixed;
      /* trick to remove some annoying flickering on webkit browsers */
      width: 89.8%;
      max-width: 1170px;
      left: 50%;
      right: auto;
      transform: translateX(-50%);
      height: 50%;
      z-index: -1;
    }    
 
    body::before {
      top: 0;
      background-color: #f4bd89;
          transition: all 0.8s;
    }
 
    body::after {
      top: 50%;
      background-color: #71495b;
    }
 
    .cd-service {
      position: relative;
      z-index: 2;
      min-height: 50px;
      margin-left: 56px;
      background-color: #3e253c;
      padding: 1em 1em 4em;
    }
 
    .cd-service::before, .cd-service::after {
      content: '';
      position: absolute;
      width: 56px;
      right: 100%;
      z-index: 2;
    }
 
    .cd-service::before {
      top: 0;
      height: 50px;
      background-repeat: no-repeat;
    }
 
    .cd-service::after {
      top: 50px;
      bottom: 0;
      background-image: url("../img/cd-pattern-small.svg");
      background-repeat: repeat-y;
    }
 
    .cd-service.cd-service-1::before {
      background-image: url("../img/cd-icon-1-small.svg");
    }
 
    .cd-service.cd-service-2::before {
      background-image: url("../img/cd-icon-2-small.svg");
    }
 
    .cd-service.cd-service-3::before {
      background-image: url("../img/cd-icon-3-small.svg");
    }
 
    .cd-service.cd-service-4::before {
      background-image: url("../img/cd-icon-4-small.svg");
    }
 
## 事件绑定

另一个我们想要完成的是效果是当滚动时改变填充颜色。

为了完成这个效果，对于每一个`.cd-service`列表项都创建一个背景颜色。

    body.new-color-1::before {
    	background-color: #c06c69;
    }
 
    body.new-color-2::before {
    	background-color: #bf69c0;
    }

这意味着，如果你有n个列表项，你需要创建n-1个样式。

当用户开始滚动，当滚动到`.cd-service-2`，我们分配`.new-cokor-1`给body。以此类推。

最后，当一个新的`.cd-section`在视图中可见，我们分配`.focus`来高亮内容。

## 译者

codyhouse作者讲的都是流程，大概在他们眼中，给个思路就够了。。。事实上，原文其实有demo还有下载地址。自己去下载并分析源码然后复现效果这才是学习该效果最好方式。后续我可能还会贴类似效果的翻译。基本上都能在原文中找到源码。

























