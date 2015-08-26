---
layout: post
title: Graphical Text Effects (CSS VS SVG:图像文字效果）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://blogs.adobe.com/dreamweaver/2015/07/css-vs-svg-graphical-text.html#)

### 作者: Sara Soueidan

### 译者: 临水照影


## 正文之前

午睡睡得昏昏沉沉，翻译一下sara女神的文章学点东西并提提神。译文有删减-0-，有些东西我懒得翻译。觉得有影响的可以去看原文-0-


## 正文

这篇文章是一系列技术探索文章中的第一篇，文中的例子可以同时用css和svg来实现。然后我们将进行对比。由于我偏好于svg，而且这篇文章真的证明了svg在解决某些WEB设计问题上比css更棒，因为它是图像和文档的结合格式。但为了保持一个客观的角度，我们会权衡每种技术的优点和缺点，并找出CSS和SVG可以更好的为设计服务的一面。

在这篇文章中，我们将用CSS和SVG创建一些图形文字效果。

## 用CSS来创建图形文字效果

### 老的CSS方法

假设你有一个`H1`标题而且你希望用令人印象深刻的图形效果来代替文本，这是你将用的CSS：
    
    h1.hide-text {
        text-indent: 100%;
        white-space: nowrap;
        overflow: hidden;
        background-image: url(…);
    }

文本缩进确保内容超过了自身的边界并溢出它们。然后隐藏溢出的内容确保超出边界的内容。因此，你最终会得到一个没有文本内容的矩形的区域。

然后这个区域被背景图片填满，你可以用PS来完成吸引人的文本的图片。

用漂亮的图片来代替文本就是你要做的。

### 新的css方法

最常见的文字效果就是文字内部填充背景。

### 纹理填充文字

![img1](http://7s1say.com1.z0.glb.clouddn.com//svgcss1.png)

利用css的`background-clip`属性，我们能够用图片填充文本。这个属性决定一个元素的背景绘图区。

`background-clip`属性在webkit内核的浏览器下有第四个属性 [`text`](https://www.webkit.org/blog/164/background-clip-text/),它可以裁剪背景图片到前景文字。然后，通过用`-webkit-text-fill-color`给文本一个透明色，这个背景图片将贯穿整个文字，完成裁剪效果。

举个例子：
    
    .clippedElement {
      /* background image that will serve as text fill */
      background: url(path/to/your/image.jpg) no-repeat center center;
        /* -webkit-background-clip clips the background of the element to the text */
        -webkit-text-fill-color: transparent; /* overrides the white text color in webkit browsers */
        -webkit-background-clip: text;

      /* general styles */
        background-size: 100% auto;
        color: #fff;
        text-align: center;
        padding: 2em;
    }
    
但不幸的是，这些效果无法在IE和Firefox上显示。

所以，如果我们给之前的H1应用这个类，它不会显示纹理效果。但如果用一个`DIV`来包含它，并给这个`DIV`的背景图片应用样式。它会显示对应效果。

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/s7gnvgm6/)

如果你想用在非webkit内核的浏览器上展现这种效果，你可以用图片来代替。

注意，这里的图像可以是任何图像，甚至是CSS的[梯度](http://tympanus.net/codrops/css_reference/gradient/)


## 纹理文本(带背景混合)

接下来，我们将用css 遮罩来制作下面的效果

![img2](http://7s1say.com1.z0.glb.clouddn.com//css-mask2.png)

当用css遮罩时，我们让文本来采取其遮罩图片的形状，而不是让遮罩图片采取文字的形状。

这个效果用的遮罩图片来自是我之前写的一篇[文章](http://tympanus.net/codrops/2013/12/02/techniques-for-creating-textured-text/).图片用来当做遮罩来遮住文本区域并让它背后的背景通过遮罩显示。如果你选择了合适的纹理和背景，你将获得无缝融合的效果。我们采用的是一张飞溅墨点图，为了简单起见，我们没用任何混合，只是应用这个飞溅图。

![img3](http://7s1say.com1.z0.glb.clouddn.com//splatter-mask1.png)

当你应用css遮罩到你的文本，这个文本需要在黑色区域中可见，遮罩图像的部分的文本必须是透明的。因为遮罩图像在CSS默认是透明遮罩，而不是亮度遮罩。当我们提供透明遮罩，黑色区域将被定义用来绘制文本。

我们将应用`mask-image`属性

    h1 {
        /* the line that applies the splatter effect */
        mask-image: url(../img/splatter-mask_1.png); 

    /* any general styles go here like font family, alignment, etc. */
    }

Firefox只支持svg遮罩，webkit浏览器要求webkit前缀而且只支持特殊的两个属性。这是[浏览器支持表](http://caniuse.com/#feat=css-masks)

[这是DEMO](http://jsfiddle.net/linshuizhaoying/s7gnvgm6/1/)

如果你想用梯度遮罩，可以用下面的代码
    
    mask-image: linear-gradient(black, transparent);
    
用svg，事情好像变得更加好了

## 图形文字与SVG

Svg超级棒（我不得不说）。现在，为了简单起见，我们将深入代码并解释它。

在SVG中工作时，无论是文本还是其它我们都将在`<svg>`元素中定义。

## 文本纹理填充

为了用纹理填充文本，我们需要定纹理，这里我们给svg文本填充颜色。

![img4](http://7s1say.com1.z0.glb.clouddn.com//gradient-text1.png)


代码如下：
    
    <svg xmlns=“http://www.w3.org/2000/svg”  viewBox=“0 0 1250 400” width=“1250” height=“400”> 
        <title>Gradient-filled Text</title>
        <!— Source: http://lea.verou.me/2012/05/text-masking-the-standards-way/ —>
        <defs>
           <linearGradient id=“filler” x=“0%” y=“100%”>
               <stop stop-color=“gold” offset=“0%”></stop>
                 <stop stop-color=“purple” offset=“20%”></stop>
               <stop stop-color=“deepPink” offset=“40%”></stop>
               <stop stop-color=“orange” offset=“60%”></stop>
               <stop stop-color=“yellow” offset=“80%”></stop>
               <stop stop-color=“skyblue” offset=“100%”></stop>
          </linearGradient>
        </defs>
        <text x=“100” y=“70%” font-size=“205” fill=“url(#filler)”> Radiant Text</text>
    </svg>

注意：我们需要给svg元素定义高度和宽度，但为了让svg能够响应式，我们需要用css百分比覆盖默认尺寸。你可以在这里看到[相关资料](http://tympanus.net/codrops/2014/08/19/making-svgs-responsive-with-css/)

`<defs>`中定义我们的纹理为线性填充。

这是[demo](http://jsfiddle.net/linshuizhaoying/s7gnvgm6/2/)

这里的纹理也可以是任何图片或者梯度。甚至可以是一个svg图案。

因为svg是一个图形，确保给它一个title让它能被屏幕阅读器识别。title对于svg就像alt属性对于在html页面中的img元素。

## 文本应用纹理（与背景混合）

与之前的技术类似，混合遮罩和背景。

这是我们想要完成的效果：
    
![img5](http://7s1say.com1.z0.glb.clouddn.com//svg-mask-demo2.png)


这像文字被咬了好几口。

首先我们在图形编辑器中绘制创建一些咬痕遮罩。

之后我们导出为它们为svg形状，并放在`<defs>`中，像这样：
    
    <svg viewBox=“0 0 900 400”>
      <defs>
        <mask id=“mask”>
          <rect x=“0” y=“0” width=“100%” height=“100%” fill=“#fff”></rect>
                <path fill=“#000” d=“…”></path>
                <path fill=“#000” d=“…”></path>
                <path fill=“#000” d=“…”></path>
                <path fill=“#000” d=“…”></path>
                <path fill=“#000” d=“…”></path>
                <path fill=“#000” d=“…”></path>
                <path fill=“#000” d=“…“></path>
                <path fill=“#000” d=“…“></path>
        </mask>
      </defs>

      <text font-size=“230” fill=“#FF481E” mask=“url(#mask)”>
        <tspan x=“0” y=“150”>nom</tspan>
        <tspan y=“280” x=“150”>nom<tspan>
        <tspan y=“400” x=“350”>nom<tspan>
      </text>
    </svg>

这是一个很重要的知识点：在SVG中，不像CSS，元素会被默认的在白色区域中绘制，在黑色区域中不绘制。任何在黑白之间的值都会被渲染为半透明，越接近白色，遮罩越不透明，越接近黑色，遮罩越透明。

所以，在示例中，我给咬痕形状黑色填充。

[DEMO CLICK HERE](http://jsfiddle.net/linshuizhaoying/s7gnvgm6/3/)

然后你在这个SVG应用任何纹理，因为在`<defs>`有很多元素选择，你可以创建动态纹理来填充。

## SVG中动态文本填充

Svg不仅能够让我们更好的支持模块化，SVG代码更能自然的动画化。

当你在`<defs>`中填充一个元素，你还可让它动起来。像下面这样：


![img6](http://7s1say.com1.z0.glb.clouddn.com//animated-fills.gif)

作为一个简单的例子，你可以用gif动画来填充。

以上的图片中例子都可在[这里](http://tympanus.net/codrops/2015/02/16/create-animated-text-fills/)找到。

注意观察浏览器支持情况。



## 结尾

Svg比css更适合制作文本特效。




## 译者

Sara女神的每篇文章都很值得学习，而且干货满满。这篇文章因为太长了，我就精简了一下-0-，觉得不放心可以去看原文。











