---
layout: post
title: Custom Data and Pseudo-Elements (使用自定义数据和伪元素）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://tympanus.net/codrops/2013/07/05/using-custom-data-attributes-and-pseudo-elements/)

### 作者: Mary Lou


### 译者: 临水照影


## 正文之前

该文章属于web特效教程。

## 正文

![imgn](http://img.haoqiao.me//PseudoElementsImageCaptions.jpg)

今天我将向你介绍简单的关于数据属性和伪元素的css窍门。

目的是用一个锚点和图片创建一个图片标题。我们将探索如何用创建伪元素配合自定义数据属性来达到hover后将它们在图片上展示。

如果你想创建更多有意思的Hover效果你可以看[这里](http://tympanus.net/Tutorials/CaptionHoverEffects/)

注意一些css特性在老的浏览器中依然不支持，比如伪元素动画。

让我们先写一个例子：

    <a 
	    class="caption" 
	    href="http://cargocollective.com/jaimemartinez/Illustration" 
	    data-title="Smug Eagle" 
	    data-description="I watched the four riders ...">
        <img src="images/1.jpg" alt="Illustration of Smug Eagle">
	
    </a>

正如你所见，我们定义了两个数据属性，一个是标题，一个是描述，这两个属性的值将在伪元素`:before`和`:after`
中被使用。

我们的第一个图片标题是静态的，因此我们讲把数据属性的内容显示在图片右边。这是我们添加的css：
    
    .caption {
    	display: inline-block;
    	position: relative;
    	margin: 10px;
    }

    .caption img {
    	display: block;
    	max-width: 100%;
    }

这个caption将作为`inline-block`元素显示，来让其中的内容流动围绕它。我们添加一个margin和定位为relative。这非常重要因为我们将把伪元素绝对定位，让它能够相对caption显示。

给图片一个max-width能够让它适应响应式环境。

## 例子1：标题旁边的图片

现在，让我们做第一个效果。我们希望两个伪元素都能够绝对定位。并在图片右侧显示。让我们定义一些常规的样式：
    
    .caption::before,
    .caption::after {
    	position: absolute;
    	left: 100%;
    	width: 90%;
    	margin: 0 0 0 10%;
	    font-weight: 300;
	    color: #89867e;
    }

通过设置left为100%,我们让伪元素在图片右侧显示。width: 90%是自己来根据图片大小来调试。

让我们对每个伪元素设置内容。我们将从数据属性中获得数据然后通过`attr()`给伪元素添加内容：
    
    .caption::before {
	    content: attr(data-title);
	    top: 0;
	    height: 25%;
	    padding: 5px 30px 15px 10px;
	    font-size: 40px;
	    border-bottom: 1px solid rgba(0,0,0,0.1);
    }

除了尺寸有所改变，我们会给:after添加类似的样式：
    
    .caption::after {
	    content: attr(data-description);
	    top: 25%;
	    padding: 20px 10px 0;
	    font-size: 18px;
    }

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/tpjvwbtr/)

## 例子2: 悬停时展现

现在，让我们给标题添加一个Hover,简单的给伪元素添加一个透明度改变的效果。

首先我们需要给它们设置绝对定位。但这次，我们需要覆盖整张图片，设置透明度为0并且给它一个过渡效果。
    
    .caption::before,
    .caption::after {
    	opacity: 0;
	    position: absolute;
	    width: 100%;
	    color: #fff;
	    padding: 20px;
	    transition: opacity 0.3s; 
    }

标题和描述的背景颜色将不同而且它的高度是整个高度的30%：
   
   
    .caption::before {
	   content: attr(data-title);
	   top: 0;
	   height: 30%;
	   background: #a21f00;
	   font-size: 40px;
	   font-weight: 300;
    }

对于描述我们将不仅仅给它一个简单的数据，因为我们将它置于一个闭合引号之间。
    
    .caption::after {
    	content: '\201C' attr(data-description) '\201D';
    	top: 30%;
    	height: 70%;
    	background: #db2e00;
	    font-size: 16px;
	    text-align: right;
    }
    
因为标题占了30%，因此给描述剩下的70%。

最后我们设置Hover时的透明度为1

    .caption:hover::before,
    .caption:hover::after {
    	opacity: 1;
    }


[Clcik Demo Here](http://jsfiddle.net/linshuizhaoying/3qfgwx60/)


## 例子3：悬停时滑动

这个例子中我将展示一个标题伴随更多花哨。在悬停时，我希望标题能从顶端滑出，描述能从底端滑出。我们同时希望图像能够变暗。因此我们需要做更多的事情。

锚的溢出需要被隐藏，因为我们需要通过它来定位我们的标题和描述。当然，我们不想用看见它们。另外，我们将给它一个黑色背景因此我们能够让图像变暗：
    
    .caption {
    	display: inline-block;
	    position: relative;
	    margin: 10px;
	    overflow: hidden;
	    background: #000;
    }

在hover时我们需要设置图像的透明度为0.5，它将让图像变暗
    
    .caption img {
    	display: block;
    	max-width: 100%;
	    transition: opacity 0.3s ease-in-out; 
    }

    .caption:hover img {
    	opacity: 0.5;
    }

伪元素的样式和之前的类似。我们只需要让它们拥有相同的高度和宽度，并置顶。让它们留在图像的顶部。

    .caption::after,
    .caption::before {
    	position: absolute;
    	width: 100%;
	    height: 50%;
	    color: #fff;
	    z-index: 1;
    	transition: transform 0.3s ease-in-out; 
    }

让我们给伪元素设置背景颜色，并且添加过渡。我们通过转化为Y轴定位伪元素在整个矩形之外。
    
    .caption::after {
    	content: attr(data-title);
    	top: 0;
    	background: #0083ab;
	    font-size: 40px;
	    font-weight: 300;
	    padding: 30px;
	    transform: translateY(-100%);
    }

    .caption::before {
	    content: '...' attr(data-description) '...';
	    top: 50%;
	    background: #f27545;
	    font-size: 14px;
	    padding: 20px;
	    transform: translateY(100%);
    }

在hover时，我们给设置过渡为translateY(0%)它们会自动定位到顶部。

    .caption:hover::after,
    .caption:hover::before {
    	transform: translateY(0%);
    }

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/26br973o/)


## 例子4：hover时侧推

最后一个例子我们将完成侧推效果。在Hover时，我们希望将图片向右侧滑动，伪元素左滑入。

让我们再一次创建阴影效通过让背景颜色从半透明到黑色透明动画。

    .caption {
    	display: inline-block;
    	position: relative;
    	margin: 10px;
    	overflow: hidden;
    	background: rgba(0,0,0,0.2);
	    transition: background 0.3s ease-in-out;
    }

    .caption:hover {
    	background: rgba(0,0,0,0);
    }

当Hover时，图片右滑：
    
    .caption img {
    	display: block;
    	max-width: 100%;
    	transition: transform 0.3s ease-in-out;
    }

    .caption:hover img {
    	transform: translateX(100%);
    }

伪元素需要在锚的下面，我们设置它的`z-index`为-1，刚开始将它们放在左侧：
    
    .caption::before,
    .caption::after {
    	position: absolute;
    	width: 100%;
    	z-index: -1;
	    background: #cecece;
	    transform: translateX(-30%);
	    transition: transform 0.3s ease-in-out;
    }

和以前一样，我们给伪元素添加内容和样式：

    .caption::before {
    	content: attr(data-title);
    	height: 30%;
    	color: #05b19a;
	    font-size: 40px;
	    font-weight: 300;
	    padding: 30px;
    }

    .caption::after {
    	content: '\201C' attr(data-description) '\201D';
    	top: 30%;
	    height: 70%;
	    color: #fff;
	    font-size: 14px;
	    padding: 20px 30px;
    }

在hover时，我们设置`translateX`为0：
    
    .caption:hover::before,
    .caption:hover::after  {
	    transform: translateX(0%);
    }

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/oxboj2yu/)

## 译者

这是非常不错的一部教程，当你有所不了解时，对demo进行一定程度的修改调试能够更加利于你对原理的理解。




























