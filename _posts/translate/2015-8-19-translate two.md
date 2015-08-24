---
layout: post
title: Create Text Filling with Water Effect（创建水波纹效果文字）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](https://blogs.adobe.com/dreamweaver/2015/08/create-a-text-filling-with-water-effect.html)

### 作者:by Lucas Bebber  

### 译者:临水照影


## 正文之前

吸取第一篇翻译的教训，将在译文中加入自己的东西，可能会扩充来讲。

## 正文
在这边文章中，我将向你展示如何制作“文字被水逐渐填满”的效果。它可用于预加载，标题等等。但总的来说，它终归只是一个svg效果的学习。

具体效果戳[这里](http://codepen.io/lbebber/pen/RPvxGp)

### 在实现效果之前

在我们写代码之前，我们先来探讨这个效果实现的流程。首先，我们需要创建一个卷曲的水波纹的效果，这个可以通过波状的图形来实现。（译者PS:比如word里的柱形图拉一拉。）

![img1](http://7s1say.com1.z0.glb.clouddn.com//t-2-1.png)      
 
然后将它无缝重叠。

![img2](http://7s1say.com1.z0.glb.clouddn.com//t-2-2.png)

然后让它无限滚动起来。

![img3](http://7s1say.com1.z0.glb.clouddn.com//t-2-3.gif)

从上图中的红色框框中，你可以看到基本我们将要实现的效果。

然后我们同时增加它的高度。

我们将从零实现。基础代码点[这里](http://codepen.io/lbebber/pen/doadze/)

增加图片的高度，而不是简单的向上移动，这会让细节看起来更加细腻。也可以让水波效果更加真实，因为它在不断填充它的容器。

最后，我们只需要遮住掉我们的文字动画就大功告成了，

现在，我们来看看我们代码该如何写。

### Css

这个效果用Css来写更加简单，但是这有个问题：用文字来作为遮罩。我们需要用到 `background-clip:text`这个属性，但是不幸的是只有支持Webkit的浏览器支持(PS:也就是IE呵呵哒)。不过我们可以用图像和文本来绕过它。但又带来一个新问题，文字不是可选的，它无法被编辑，它可能很难被辨认。因此，我不打算深入css版本来写，但是你可以看到另一个实现的例子。点[这里](http://codepen.io/lbebber/pen/xrwja/)

### Svg

  另一个选择就是Svg，Sara Soueidan写了一篇关于文字效果的很棒的文章，你可以点[这里](http://blogs.adobe.com/dreamweaver/2015/07/css-vs-svg-graphical-text.html).在那篇文章中，她介绍了Svg遮罩和它的工作原理。
 
 但这里实现的原理不同，与其用图形来遮罩文字，还不如用文字来作为遮罩，来遮罩我们的水波图像。
 
 这是我们创建的遮罩
 
     

      <defs>
         <text id="text" font-size="100">YOUR TEXT HERE</text>
           <mask id="text-mask">
              <use x="0" y="0" xlink:href="#text" fill="#ffffff"/>
           </mask>
      </defs>
      
将文本放在`<defs>`标签里面是因为我们需要重复使用，一个作为遮罩，一个作为背景。通过这种实现方式，我们可以做到一次编写，多次引用。

通过`<use>`标签，当我们想用的时候，只要通过`xlink:href`属性指向对应`ID`就行了。

注意，当用文字作为遮罩，我们设置`fill`属性来使其填充色为白色。这是因为Svg遮罩的原理：白色遮罩使后面的内容可视，黑色使它隐藏。任何深浅之间使其透明对应。

水波图形需要提前制作，然后反复调整整体，使其看上去更加形似。你可以用Svg的`pattern`，因为它可以重复自身并包含任意大小的文字。Svg的`pattern`可以模糊，这可以单开一章来说明。但现在我们只需要使用它：

    <defs>
      ...

      <pattern id="water" width=".25" height="1.1" patternContentUnits="objectBoundingBox">
        <path fill="#fff" d="..."/>
      </pattern>
    </defs>

    <rect class="water-fill" fill="url(#water)" mask="url(#text-mask)" x="0" y="0" width="1600" height="120"/>


注意`<rect>`的`fill`属性指向 `#water`(这是我们上面提到的),同时要注意到`mask`属性指向我们之前提前写好的内容。

如果你想用其他类型的元素来作为波形图，像`path`属性来代替`patterns`，你需要自己来设置`mask`的属性。

所以我们现在可以看到实现的效果，点[这里](http://codepen.io/lbebber/pen/waNyQv)

到目前为止都很棒，现在我们让它动起来。。。

### 动起来

再次的，我们原本可以简单的用css来实现，但是当我们运行在浏览器时，很多浏览器并不支持用css改变SVG的一些属性，比如`x`和`y`.更多的浏览器支持用`transform:translate(x,y)`设置SVG元素的位置。然而这并不能在我们的例子中工作，因为它会带动遮罩一起移动。如果你的浏览器支持用css来操作SVG，你可以点[这里](http://codepen.io/lbebber/pen/doagyV/)

这个Css动画非常简单：

    @keyframes wave {
      0% {
        x: -400px;
      }
      100% {
        x: 0;
      }
    }

    @keyframes fill-up {
      0% {
        height: 0;
        y: 130px;
      }
      100% {
        height: 160px;
        y: -30px;
      }
    }

    .water-fill{
      ...
      animation: wave 0.7s infinite linear, fill-up 10s infinite ease-out alternate;
    }

我们这里同时运行两个动画，一个叫做wave，这让动画效果看起来向右边无限的波动(注意波动的距离只是其中一个波的宽度，循环设置为无限，easing设置为线性。)然后让fill-up不断的增加高度。

对于浏览器支持问题我们解决的办法是用javascript。如果你想把这个效果用在一个基于进程的预加载，你也不得不使用javascript,不过好消息是我们已经进行到一半了。

你可以用任意的js动画库作为你的选择，像[Transit](http://ricostacruz.com/jquery.transit/) 或者[Velocity.js](http://julian.com/research/velocity/).在这个例子中，我们将用到[GSAP](http://greensock.com/gsap)

### GSAP动画

这是[GSAP](http://greensock.com/get-started-js)的简介，为了以防你之前从没用过它。

我们将把刚刚css的动画写成js版本：

    // Select the element
      var fill=document.querySelector(".water-fill");

        // "Wave" animation
        TweenMax.fromTo(fill,0.8,{
          // Set the "from" position
          attr:{
            x:-400
          }
        },
        {
          // Set the "to" position
          attr:{
            x:0,
          },
          // Repeat infinitely
          repeat:-1,
          // Linear easing
          ease:Linear.easeNone
        });


        // "Fill up" animation
        TweenMax.fromTo(fill,10,{
          // From
          attr:{
            y:120,
            height:0
          },
        },{
          // To
          attr:{
            y:-20,
            height:140
          },
          repeat:-1,
          // Reverse animation on loop
          yoyo:true,
          ease:Linear.easeNone
        });
      
最后，我们可以选择用变暗的文字拷贝来作为背景，来使在被填满之前的效果更清晰。为了达到这个效果，我们可以重复使用我们之前的代码：
    
    <use x="0" y="0" xlink:href="#text" fill="#222"/>
      
这是我们最终的结果，你可以点[这里](http://codepen.io/lbebber/pen/RPvxGp)

希望你能感到有趣并觉得有帮助。

## 译者言
根据原理调了下参数，然后改成了一个vue组件，只需要在页面加载的时候加载作者提到的TweenMax.min.js就行了。
例子如下：
    
    
    <template>
    	<svg class="loading" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
	 width="574.558px" height="60px" viewBox="0 0 574.558 120" enable-background="new 0 0 574.558 120" xml:space="preserve">
      <defs>
        <pattern id="water" width=".25" height="1.1" patternContentUnits="objectBoundingBox">
          <path fill="#fff"     d="M0.25,1H0c0,0,0-0.659,0-0.916c0.083-0.303,0.158,0.334,0.25,0C0.25,0.327,0.25,1,0.25,1z"/>
        </pattern>
    
        <text id="text" transform="matrix(1 0 0 1 -8.0684 116.7852)" font-family="'Cabin Condensed'" font-size="48">{{msg}}</text>
    
        <mask id="text-mask">
          <use x="0" y="0" xlink:href="#text" opacity="1" fill="#ffffff"/>
        </mask>
      </defs>
 
	      <use x="0" y="0" xlink:href="#text" fill="#222"/>
  
      <rect class="water-fill" mask="url(#text-mask)" fill="url(#water)" x="-400" y="0" width="800" height="10"/>


    </template>




    <script>
        module.exports = {
          created: function () {
	    	document.addEventListener("DOMContentLoaded",function(){
	    	  var fill=document.querySelector(".water-fill");
	    	  TweenMax.fromTo(fill,0.8,{
	    	    attr:{
	    	      x:-200
	    	    }
	    	  },
	    	  {
	    	    attr:{
	    	      x:0,
	    	    },
		        repeat:-1,
		        ease:Linear.easeNone
		      });
		  
		      TweenMax.fromTo(fill,10,{
		        attr:{
		          y:120,
		          height:0
		        },
		      },{
	    	    attr:{
	    	      y:60,
	    	      height:140
	    	    },
		        repeat:-1,
	    	    yoyo:true,
		        ease:Linear.easeNone
	    	  });
		  
		    });
      },
      data: function () {
        return {
          msg: '这是一个测试' 
        }
      }

    }
    </script>
      
希望有所帮助。
      
      
      
      
      