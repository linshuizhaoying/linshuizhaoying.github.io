---
layout: post
title: Animated line drawing in SVG (创建svg线条动画）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](https://jakearchibald.com/2013/animated-line-drawing-svg/)

### 作者: Jake

### 译者: 临水照影


## 正文之前

准备实现一个效果，于是将这篇文章从我的翻译列表中找出来翻译学习。


## 正文

我喜欢用图像来作为展示信息流或者浏览器行为的方式。但是大的图片第一看上去就令人生畏。当我在会议上谈论有关应用缓存和渲染表现时我用了一块空屏幕让图片一点点的画出来作为描述的方法。这是方法：
    
### SVG PATHS

    <path fill="none" stroke="deeppink" stroke-width="14" stroke-miterlimit="0"
    d="M11.6 269s-19.7-42.4 6.06-68.2 48.5-6.06 59.1 12.1l-3.03 28.8 209-227s45.5-21.2 60.6 1.52c15.2 22.7-3.03 47-3.03 47l-225 229s33.1-12 48.5 7.58c50 63.6-50 97-62.1 37.9"
    />
    
我用 [Inkscape](https://inkscape.org/zh-tw/)创建了一些SVG。它有一点笨拙，但是它给你一个SVG DOM视图就像你编辑文本那样。而不是像 Adobe Illustrator导出svg格式。

`d`属性的每一个部分都告诉渲染器下一步需要渲染的点，绘制一条线段，绘制一个贝塞尔曲线的另一点，等等。

    <path stroke="#000" stroke-width="4.3" fill="none" d="…"
      stroke-dasharray=""
      stroke-dashoffset="0.00"
    />
  
`stroke-dasharray ` 让你指定渲染部分的长度，间隙的长度。` stroke-dashoffset`让你改变dasharray开始的点。

将它们设置为最大。然后逐渐降低dashoffset，你会发现线段开始绘制。你可以从dom中得到线段的长度。
    
    var path = document.querySelector('.squiggle-container path');
    path.getTotalLength(); // 988.0062255859375

### 让它运动

用css动画或者过渡能够很容易让svg动起来。但是它不支持ie，如果你想让IE支持，你需要用`requestAnimationFrame`并用脚本一帧一帧更新frame的值。

避免使用smil,IE不支持它，而且在chrome和safari中表现得也不算很好。

我准备用CSS过渡。

在第一个例子中我用svg来定义点，你可以用css完成同样的事情。大多数svg属性与css属性相同.([资料](http://www.w3.org/TR/SVG/styling.html))

    var path = document.querySelector('.squiggle-animated path');
    var length = path.getTotalLength();
    // Clear any previous transition
    path.style.transition = path.style.WebkitTransition =
      'none';
    // Set up the starting positions
    path.style.strokeDasharray = length + ' ' + length;
    path.style.strokeDashoffset = length;
    // Trigger a layout so styles are calculated & the browser
    // picks up the starting position before animating
    path.getBoundingClientRect();
    // Define our transition
    path.style.transition = path.style.WebkitTransition =
      'stroke-dashoffset 2s ease-in-out';
    // Go!
    path.style.strokeDashoffset = '0';

用`getBoundingClientRect` 来触发布局是一个hack，但是它挺有用。但不幸的是，如果你在同个javascript中同步修改同样的样式两次，那么只会计数最后的修改。我写了篇关于这个的细节的[文章](http://coding.smashingmagazine.com/2013/03/04/animating-web-gonna-need-bigger-api/)

我通常用offsetWidth来触发布局。但是在firefox中似乎没用。

### 更多乐趣

 Lea Verou用了同样的技术创建[加载动画](http://dabblet.com/gist/6089395),Josh Matz 和 El Yosh 扩展了该[技术](http://dabblet.com/gist/6089409)


## 译者

文中其实是有例子的，但是作者是直接写进了他的网站，因此想看例子的可以去原文看。
然后就是在mac下我试了一下，通过Inkscape这个工具并搭配以这项技术延伸出来的[脚本](https://github.com/ConnorAtherton/walkway/)制作了一个svg线条动画。

![img1](http://img.haoqiao.me//xxx.gif)








