---
layout: post
title: Responsive Sprite Animations with ImageMagick and GreenSock (用ImageMagick 和 GreenSock 制作响应式的雪碧图动画）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://www.sitepoint.com/responsive-sprite-animations-imagemagick-greensock/)

### 作者: Tom Bennet

### 译者: 临水照影


## 正文之前

果然在这种虐单身狗的日子里才有学习的动力。。。

## 正文

Css雪碧图并不是一门新的技术，自从[A List Apart in 2004](http://alistapart.com/article/sprites),这项不起眼的技术就被很多Web开发者作为开发工具的必备。但是，虽然雪碧图带来的加快网页加载速度这个好处显而易见，但它们在移动端开发的讨论非常少。它们的原理都是一样的:将多张图合并成一张主图，在需要的地方根据位置来显示。

在这篇文章中文章中，我们将探索一种简单地方法来创建轻量级的，对移动端更加友好的，甚至有更强互动性的响应式的CSS雪碧图动画。你可以不需要任何特殊的图像软件，仅仅只需要懂一点点css和javascript 。让我们开始吧。

### 创建一个雪碧图

雪碧图基于一个视频游戏（拳皇=-=）。

我们需要将一个连续动作的每一帧放进我们的一张图中。这里有数十种工具可以帮助我们完成这项任务，甚至有些还可以买一送一送张样式表给你。。。[Compass’s built-in spriting features](http://compass-style.org/help/tutorials/spriting/)是非常强大的一个工具，[SpritePad](http://wearekiss.com/spritepad)是一个基于
WEB的雪碧图管理工具。然而对于我们的需求而言，一个简单的命令行工具更加适合。

[ImageMagick](http://www.imagemagick.org/script/index.php)，图像处理的瑞士军刀，是一个免费而且开源的图像处理工具，它可以自动化执行那些非常费力的工作，比如合并图像。ImageMagick可以运行在任何一个操作系统。只需要去下载对应的软件就行了。

保存你动画中相同大小的帧为一系列png文件到一个文件夹。然后打开终端，执行cd跳转到你图像保存的目录，然后执行以下命令：

    convert *.png -append result-sprite.png
    
这个命令就是让ImageMagick把所有png文件保存到result-sprite.png一个文件中。这个图像将作为雪碧图，然后通过css改变取值的位置来使其动起来。

### 纯CSS动画

我们将开始一个简单的的动画，用css3的`keyframe`动画来使我们的雪碧图动起来。 如果你不熟悉这个属性请自行去找资料（=-=虽然原文中有介绍链接-0-）。我们将通过循环从图片中垂直的找到每个帧的位置然后从头到底取值并放到动画中。

为了让动画能连贯的运行起来，作为默认的行为，我们将需要将雪碧图的每一帧都展现。通过`steps()`过程，然而我们还需要控制渲染帧的数量，由于我们的工作原理是通过一个方法来实现以百分比控制背景图的位置（作者有提到原理详解，点[这里](https://css-tricks.com/i-like-how-percentage-background-position-works/)），我们需要设定步骤的数量会比我们动画总帧数的少一个。

这边是一个实现的例子。。。这次又是[codepen](http://codepen.io/SitePoint/pen/MYRKmJ)


注意我们精确定义了元素的宽度和高度到每个帧，来避免出现误差。我们设置了动画无限播放和动画帧之间每隔3.5秒，然后我们就能看拳霸中的人物进行一系列的动作。

到目前为止还是很不错的。尝试改变元素的高度，或者对我们的雪碧图应用`backgroud-size`，然后我们的动画效果将会失效，因为它现在还不是响应式的。它是基于像素尺寸的不变的图像。

### 使它响应式

如果我们希望能够自由的调整雪碧图动画大小，我们有两个选择，这两个选择的关键是我们的雪碧图对高宽比一致的强烈依赖。
而不是特定大小。我们必须保留它的比例使它能够正确的缩放。

第一个选择就是对我们的背景设置`background-size` 为雪碧图宽度的100%。然后转换高宽的单位从px转为em。然后再改变基本字体大小。这将满足我们的基本要求。

第二个选择是用纯css固定纵横比技术(可以点[这里](http://www.mademyday.de/css-height-equals-width-with-pure-css.html))。他利用基于百分比的伪元素，以保持一个真实元素比例的任意大小。

下面的例子将给我们的雪碧图增加响应式

点[这里](http://codepen.io/SitePoint/pen/zxXrzP)

这是一种对移动端很友好的方法。而且支持css3的浏览器也有那么多了，所以一般情况下也不需要考虑兼容了=-=。



## JavaScript 动画

如果我们想更加比Css3的keyframe更加精确的控制我们的动画？举个例子，如果我想用基于滚动条位置来将多个雪碧图组成一个复杂的场景，而且能够结合触发和定时功能来代替单一的雪碧图动画该如何做？

为了将设想变成可能，我们需要用javascript来控制我们的雪碧图动画。使用 [GreenSock](http://greensock.com/)动画平台和[ScrollMagic](https://github.com/janpaepke/ScrollMagic)将可以帮助我们实现这些。（不熟悉这些的童鞋可以去学一下。）

首先，让我们引发一个两秒长的动画在特定的scroll位置。它需要在最后一帧中停止。如果用户回转scroll它需要能够倒播（即使它正播放一半。）

下面是这个[例子](http://codepen.io/SitePoint/pen/vEMLJb)

注意我们用用的是之前基于em单位的字体方法，尝试改变字体大小然后缩放雪碧图。我们定义了`TweenMax.to`，设置了目的值为我们元素垂直背景位置的100%，然后用`SteppedEase`方法来达到逐帧动画的效果。

这一步我们将用滚动条同步我们的动画播放，就像用滚动条来播放一个动画。

这个例子你可以点[这里](http://codepen.io/SitePoint/pen/OPGMOL)

这个雪碧图在固定位置的动画播放持续时间由用户用滚动条控制代替。为了达到这个效果，我们用了`ScrollMagic.Scene`设置`duration`为1500px，然后用`setPin`方法来固定父元素在整个场景中的位置。如果想了解更多细节，可以去看ScrollMagic的文档。

JavaScript控制雪碧图动画比纯css更加方便。 ScrollMagic 和 GreenSock 这两个库IE9都支持。。。而且对于移动端来说这更加友好。

### 总结

哎呀每次我总结都不想翻译=-=，虽然他说的很有道理。。。不过本篇文章翻译到底才发现只是领了个路，但是作者推荐的技术和库都非常值得去深入研究，因此我打算研究完后再开一章来仔细讲解两个库的使用，而且深入雪碧图的使用也应该会包含其中。





























