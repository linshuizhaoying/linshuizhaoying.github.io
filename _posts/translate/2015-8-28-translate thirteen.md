---
layout: post
title: CSS-Only Raindrops on Window Effect (创建纯css雨滴窗效果）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](https://blogs.adobe.com/dreamweaver/2015/06/css-only-raindrops-on-window-effect.html)

### 作者: Lucas Bebber

### 译者: 临水照影


## 正文之前

事实上这个效果很久之前就出现了，但是并没有去好好研究，这次翻译顺便学习下。



## 正文

单单制作一个特效演示可能是没有什么意义的，但是这是一个探索css特性的好机会。尝试新的工具，在规定的限制中练习。今天我们将用haml和sass了来制造一个雨滴在窗口的效果。

[Demo click Here](http://codepen.io/lbebber/pen/ZGGNvZ)

## 前期准备

首先，我用haml/sass代替html/css,因为这些工具非常有用，尤其是它们能让我们从制作上百个雨滴中解脱出来。

## 窗口

首先展示窗口

    .container
      .window

sass

    // Our background image.
    // Will be used for the window as well as
    // for the raindrops themselves
    $image: 'http://i.imgur.com/xQdYC7x.jpg';

    // container width and height.
    // 100vw/vh so it fills the entire window
    $width:100vw;
    $height:100vh;

    .container{
      position:relative;
      width:$width;
      height:$height;
      overflow:hidden;
    }
    .window{
      position:absolute;
      width:$width;
      height:$height;
      background:url($image);
      background-size:cover;
      background-position:50%;
      filter:blur(10px);
    }

效果如下

![img](http://7s1say.com1.z0.glb.clouddn.com//window-optimized-700x375.png)

我们画了一个有背景图片的div作为窗口，并应用了模糊效果。

注意我们把图片存到了一个$image变量中，之所以这么做是因为我们在后面需要用到对雨滴用同样的图片。

## 真实生活

在我们继续之前，先看看真实生活中的雨滴。

![img2](http://7s1say.com1.z0.glb.clouddn.com//real-700x375.jpg)

由于光线折射，雨滴中含翻转图像。它的形状更像一个半球体，并配上黑色边框

## 雨滴
基于我们所见，来创建一个简单的雨滴

    .container
      .window
      .raindrop

sass
    
    $drop-width:15px;

    // our raindrops are not going to be perfectly round, so we will stretch them a bit
    // (not using transform:scale so our background doesn't get stretched)
    $drop-stretch:1.1;
    $drop-height:$drop-width*$drop-stretch;

    .raindrop{
        position:absolute;
        top:$height/2;
        left:$width/2;

        width:$drop-width;
        height:$drop-height;

        // border radius 100% instead of 100px, so the raindrop is elliptical rather than a capsule-shaped
        border-radius:100%;
        background-image:url($image);
        background-size:$width*0.05 $height*0.05;
        transform:rotate(180deg);
    }

这个效果非常简单，画一个椭圆的div,用之前的背景图片来填充它，缩放背景，然后颠倒。效果如下

![img3](http://7s1say.com1.z0.glb.clouddn.com//borderless-optimized-700x374.png)

现在我们将增加一个边框，让雨滴看起来更加蓬松
    
     ...
    .border
    .raindrop

sass
    
    ...

    .border{
        position:absolute;
        top:$height/2;
        left:$width/2;

        margin-left:2px; 
        margin-top:1px;

        width:$drop-width - 4;
        height:$drop-height;

        border-radius:100%;

        box-shadow:0 0 0 2px rgba(0,0,0,0.6);
    }
    
    
注意我们并没有直接在雨滴中直接增加一个边框，我们创建一个新的div并移动它，让它看起来更加自然

![img4](http://7s1say.com1.z0.glb.clouddn.com//bordered-optimized-700x374.png)


现在雨滴看起来更加棒，让我们创建上百个雨滴。
    
    .raindrops
        .borders
            - (1..100).each do
                .border
        .drops
            - (1..100).each do
                .raindrop

对于haml我们直接加循环，但对于sass比较棘手，我们一步步来实现它

首先，我们需要创建循环选中每个独立的元素
    
    @for $i from 1 through 100{

      // using the $i variable with the CSS nth-child pseudo-class
      .raindrop:nth-child(#{$i}){

       }

      .border:nth-child(#{$i}){

       }
    }
    
现在我们将生成随机位置和大小来让雨滴分散
    
    @for $i from 1 through 200{

    // generates a random number from 0 to 1, for the positioning
        $x:random();
          $y:random();

      // Random raindrop size and stretching.
      // Since each raindrop has different sizes, we'll do our sizing
      // calculations here instead of on the main .raindrop selector
         $drop-width:5px+random(11);
         $drop-stretch:0.7+(random()*0.5);
        $drop-height:$drop-width*$drop-stretch;

        .raindrop:nth-child(#{$i}){
            // multiply the random position value by the container's size
            left:$x * $width;
            top:$x * $height;

            width:$drop-width;
            height:$drop-height;
        }

        .border:nth-child(#{$i}){
            // we'll do the same positioning for the drop's border
            left:$x * $width;
            top:$x * $height;

            width:$drop-width - 4;
            height:$drop-height;
        }
    }


![img7](http://7s1say.com1.z0.glb.clouddn.com//raindrops-nopos-optimized-700x370.png)

最后，我们需要把改变每个雨滴中背景的位置，让它看起来更加自然
    
    ...

    .raindrop:nth-child(#{$i}){
        ...

        background-position:percentage($x) percentage($y);
    }
    ...
    
![img8](http://7s1say.com1.z0.glb.clouddn.com//raindrops-nofilter-optimized-700x370.png)    

这就是整个过程，希望你能享受它。

## 译者

改了一下haml，换成jade格式
    
    <template lang="jade">
    .container
      .window
      .raindrops
        .borders
          - for (var i = 0; i < 100; i++) {
              div(class='border')
            - }
        .drops
          - for (var i = 0; i < 100; i++) {
              div(class='raindrop')
            - }
    </template>

然后加载一下sass,只要把关键部分替换了，就能写出一个自定义的vue组件
    
        // our background image.
        // will be used for the window as well as
        // the raindthemselves
        $image: 'http://i.imgur.com/xQdYC7x.jpg';
        
        // container width and height.
        // 100vw/vh so it fills the entire window
        $width:100vw;
        $height:100vh;
        
        //number of raindrops
        $raindrops:100;
        
        body{
          background:#222;
        }
        .container{
          position:relative;
          width:$width;
          height:$height;
          overflow:hidden;
        }
        .window{
          position:absolute;
          width:$width;
          height:$height;
          background:url($image);
          background-size:cover;
          background-position:50%;
          filter:blur(10px);
        }
        // set all the containers to
        // position:absolute, since 
        // they will overlap
        .raindrops,
        .borders,
        .drops{
          position:absolute;
        }
        
        // little brightness filter so our raindrops look shiny
        .drops{
          filter:brightness(1.2);
        }
        
        // general settings for all the 
        // raindrops and borders
        .raindrop{
            position:absolute;
          border-radius:100%;
            background-image:url($image);
            background-size:$width*0.05 $height*0.05;
          background-position:50%;
            transform:rotate(180deg) rotateY(0);
        }
        .border{
            position:absolute;
            margin-left:2px; 
            margin-top:1px;
            border-radius:100%;
            box-shadow:0 0 0 2px rgba(0,0,0,0.5);
          transform:rotateY(0);
        }
        
        // looping through each one of them
        @for $i from 1 through $raindrops{
        
          // generates a random number from 0 to 1, for the positioning
          $x:random();
          $y:random();
        
          // Random raindrop size and stretching.
          // Since each raindrop has different sizes, we'll do our sizing calculations here instead of on the main .raindrop selector
            $drop-width:5px+random(11);
            $drop-stretch:0.7+(random()*0.5);
            $drop-height:$drop-width*$drop-stretch;
            .raindrop:nth-child(#{$i}){
                // multiply the random position value by the container's size
                left:$x * $width;
                top:$y * $height;
            
                width:$drop-width;
                height:$drop-height;
            background-position:percentage($x) percentage($y);
            }
        
            .border:nth-child(#{$i}){
                // we'll do the same positioning for the drop's border
                left:$x * $width;
                top:$y * $height;
        
                width:$drop-width - 4;
                height:$drop-height;
            }
        }
        

就这样。希望有所帮助。







