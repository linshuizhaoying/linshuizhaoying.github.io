---
layout: post
title: Building A Circular Navigation with CSS Clip Paths (用CSS路径裁剪遮罩制作环形导航）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](https://css-tricks.com/building-a-circular-navigation-with-css-clip-paths/)

### 作者: Sara Soueidan

### 译者: 临水照影


## 正文之前

还记得水波纹效果的作者提到的Sara么，这篇就是那位作者的大作，好像她专注于css，很多长篇的文章都能看到她的作品，由于太长了，可能会翻译时间长一点。

## 正文

Css的路径裁剪属性是一个尚未被充分利用但是非常有意思的属性之一。它可以和Css形状属性结合创建有意思的布局（[点这里](http://alistapart.com/article/css-shapes-101)）。它也可以用来创建令人难以置信的动画([点这里](http://species-in-pieces.com/#)就是之前流传很广的30个动物变来变去的动画，国内有人专门为这个动画做过分析，感兴趣的可以自己去搜索。)

当尝试去用css和svg创建任意形状时，路径裁剪对我而言是一个非常有帮助的属性。它可以结合svg来创建环形菜单。特别是考虑到需要在浏览器中用指针触碰裁剪区域需要触发事件的时候。让我们来深入探讨这个想法。

### 写作背景

几年前，我为[codcops](http://tympanus.net/codrops/)写关于通过css转换属性和一些css技巧来用纯css创建环形按钮。那个时候的技术是很匮乏的，你不得不用大量奇淫巧计来完成你所需要完成的效果。

为了在那个技术时代创建一个菜单，所需要的步骤是非常多的而且不是很灵活，需要一些hack，而且它的内部只能用icon，因为其它内容很难被定位在一个倾斜的子项中，你可以看看之前的[这篇文章](http://tympanus.net/codrops/2013/08/09/building-a-circular-navigation-with-css-transforms/)。

今天，CSS路径裁剪属性结合svg路径，可以很轻松的创建环形菜单。这个技术不是hack，不需要任何转换工作。而且它的效果正如你所预料的那么棒。虽然这篇文章用的技术有些限制，但是制作菜单的代码是很简短干净的。

在你深入了解代码之前，即使代码非常简单很容易模仿，但你依旧需要学一些裁剪路径的知识，除非你已经非常熟悉了。这里你可以看我之前写的[文章](http://sarasoueidan.com/blog/css-svg-clipping/)。它已经包含了前期的知识要点。(简直是业界良心(๑•̀ㅂ•́)و✧)

在我们准备写代码之前我们同时需要注意浏览器支持的情况。

### 浏览器支持

我们要用css裁剪路径这个属性，首先要先明白浏览器支持情况。[查表](http://caniuse.com/#feat=css-clip-path)查表之后我们可以看到这个属性的支持情况并不算很好。
    
    IE不支持，但是他们说他们在考虑
    
    如果你用css形状方法来定义裁剪属性，FireFox不支持裁剪路径因为他们现在只支持SVG的裁剪路径属性值。
    
    Webkit浏览器不支持SVG定义的裁剪路径的指针事件。这是一个BUG，默认的指针事件不应该被局限于可见的裁剪元素外。
    
    按照期望的行为来说，当你用CSS形状方法来定义一个裁剪路径，这些浏览器应该是正确支持指针事件。然而，当你通过用SVG的裁剪路径属性来路径裁剪，指针事件应该依旧被局限于可见的裁剪元素外。这会干扰和阻止任何隐藏于裁剪路径下的元素的指针事件。我在写这篇文章的时候提交了bug，让我们期待能够尽快被修复。这个BUG意味着在这篇文章的例子中，依旧存在。对不起。
    还有一个渲染的BUG也存在于其他浏览器中，我已经提交了这个BUG。
    

(总之：这篇文章的DEMO应该是只工作在FireBox，但是文章提到的技术和知识可以扩散性的应用到其他地方。)

如果你打算用css基础形状来代替文章中用到的裁剪路径，那么这个DEMO将可以用到其它浏览器，但不能在FireFox。
    
 如果你想用css形状方法中定义裁剪路径而且想让它工作在Firefox，你需要用SVG的<clipPath>元素创建相同的形状并将其作为Firefox的后备选择。
 
 举个例子：
 
    .element {
      clip-path: url(#SVGPolygonShape); /* For Firefox */
      clip-path: polygon(...); /* For other browsers */
    }
最后要注意的一件事是Firefox只支持外部引用的SVG，所有的其他浏览器要求SVG路径定义在文档之中。

现在浏览器的BUG全部解决了，让我们开始制作我们的菜单。

### 标记

这个标记非常直接，这个菜单是包含一些子项的无序列表，其中包含了一些的内容像文本或者ICON。我用简单的标签来代替"ICON"。
    
    <ul class="menu">
      <li class="one">
        <a href="#">
          <span class="icon">icon-1</span>
        </a>
      </li>
      <li class="two">
        <a href="#">
          <span class="icon">icon-2</span>
        </a>
      </li>
      <li class="three">
        <a href="#">
          <span class="icon">icon-3</span>
        </a>
      </li>
      <li class="four">
        <a href="#">
          <span class="icon">icon-4</span>
        </a>
      </li>
      <li class="five">
        <a href="#">
          <span class="icon">icon-5</span>
        </a>
      </li>
      <li class="six">
        <a href="#">
          <span class="icon">icon-6</span>
        </a>
      </li>
    </ul>

现在整个粗略页面就完成了。

### 定义裁剪路径（扇形）

为了创建一个扇形，你需要画弧形。但是现在基础css并没有提供类似的方法： `circle()`,`ellipse()`,`inset()`,`polygon()`这些都不能。（当然除非你用`polygon()`来配合一大堆点的坐标来实现。。。问题是，你愿意这么做么？）

所以，我们需要找到一个更加方便的方法来制作一个扇形。幸运的是，`clip-path`属性给了我们通过引用SVG路径作为路径裁剪的值的能力。

换句话说，你可以通过SVG定义一个你想要的的裁剪路径（它可以是多个分离的路径），然后SVG的`clipPath`元素配合对应ID包含这个图形。然后再通过CSS URL语法来引用
    
    clip-path: url(#clipPathID);

因此，通过SVG定义一个扇形变得非常简单。然而，这里需要考虑一些数学和定位的问题。

首先，你需要确定你的菜单有多少个子项。它将决定扇形中心角的值。

接着。你需要用SVG的`path`元素画一个扇形。这个在SVG存在的路径命令可以让你能否基于一些简单的绘图原则来画路径。

我们将用四个路径命令来画扇形：
   
     M
     L
     A
     z
 
 但在我们画之前，我们需要考虑子项与扇形之间的对应关系。因此，我们需要在执行前先规划。
 
 
### 决定菜单项在裁剪路径中的位置

菜单项将绝对定位于它们之间的顶部，然后将所有菜单项裁剪到扇形之中。

为了达到这个目的，我们需要考虑这些子项的该如何放置层次（字面上的意思），然后再裁切这些层，最后只有这些层的扇形是可见的。我们来看一张原理图。

![menu](http://img.haoqiao.me//menu-layers.png)

半透明的黑色盒子仅用于演示需要。这些项在实际中是旋转而不是路径裁剪。因此对于每一个项，都将被裁剪为相同大小的扇形。然后旋转相应的角度来使它们之间不会重叠。我们再来看另一张原理图。

![menu2](http://img.haoqiao.me//menu-layers-rotated.png)

如果你倾斜下脑袋，你会更加清晰的看到里面的结构。对于第一个子项，它不是旋转的结构而是已经裁剪了。

注意第二张原理图，黑色盒子还是存在于这张页面，即使是已经通过裁剪来显示部分区域，但这个矩形依旧存在，只不过是隐藏了而已。

在我们想要继续之前。这里有个动图来展现我们之前讲到的理论。你可以看到它们都是从同一的地方出发，然后旋转不同角度。

![menu3](http://img.haoqiao.me//circ.gif)

即使这些元素被裁剪，原则上它们还是一个矩形。所以你看到扇形在旋转，你要记得这是一个矩形在旋转。


### 决定应用在菜单项中裁剪路径的点坐标

现在我们知道了子项是如何旋转的，我们知道了所有扇形初始状态都是一样的。接下来我们需要在svg中画这些形状。为了完成这个任务，我们需要决定扇形中这些点的坐标。

让我们来剖析一下扇形的代码：为了能够让css引用。我们需要将svg放到页面中
    
    <svg height="0" width="0">
      <defs>
        <clipPath clipPathUnits="objectBoundingBox" id="sector">
          <path fill="none" stroke="#111" stroke-width="1" class="sector" d="M0.5,0.5 l0.5,0 A0.5,0.5 0 0,0 0.75,.066987298 z"></path>
        </clipPath>
      </defs>
    </svg>
    
代码中最重要的部分是`clipPathUnits="objectBoundingBox"`声明。`clipPathUnits`属性接受两个值，

`userSpaceOnUse` 和 `objectBoundingBox`，前者将用整个页面作为坐标系，后者将用当前项目中的坐标系，例子中我们的是ID为sector的项目。

当我们用`objectBoundingBox`，用于绘制扇形路径的点坐标的值将在[0,1]之间的区间中。这些值近似于百分比。我们需要计算相对于元素盒子边框的坐标。这就是为什么我们取的值都小于1.

如果你使用`userSpaceOnUse`，浏览器将用整个页面作为坐标系或者用户当前所用的坐标系（cavans或者svg），这意味着它可能不是真正应用在你的元素中，你可以看我关于这点所写的[文章](http://sarasoueidan.com/blog/css-svg-clipping/index.html#clippathunits)

知道了原理，我们来看一下原理演示图

![menu4](http://img.haoqiao.me//bounding-box-coordinates.png)

这些坐标，环绕着扇形半径，将被应用于path命令来画扇形。

由于我们希望应用在我们的菜单子项中，这些坐标需要在图像中被明确指明。我们需要三个点来确定一个扇形，然后还需要这个扇形的半径。

演示中半径为0.5 第一个坐标是中心坐标(0.5,0.5)，第二个坐标是(1,0.5)第三个坐标需要简单的计算，(0.75,0.66987298).然后我们用Svg弧形命令A，和线命令l。来画一个路径。我们不需要知道它的工作原理，因为它不在本文的探讨范围之内。接下来是一个Demo

[DEMO Click Here](http://jsfiddle.net/linshuizhaoying/vn3nxk7w/)


想要了解更多Svg的Path命令，可以参考我的[文章](http://sarasoueidan.com/blog/building-a-circular-navigation-with-svg/)

### 裁剪菜单项

SVg的路径裁剪已经准备好了，接下来我们将用`clip-path`属性来裁剪我们的菜单项。

首先，子项需要被绝对定位于菜单的最高层。我们先创建一个容器
    
    .menu {
      position: relative;
    
      list-style: none;
      margin: 30px auto;

      /* padding trick for maintaining aspect ratio */
      height: 0;
      padding: 0;
      padding-top: 70%;
      width: 70%;
    }

为了让menu是正方形，我用了padding的hack，来让它保持1：1的宽高比。

因为我们希望它能流动，因此我们设定它的宽度为百分比的值。通过媒体查询，你可以指定最小的和最大的尺寸，然后调整padding。下面的例子中已经包含了我所讲的内容。

接着，指定子项在菜单的位置然后裁剪为扇形

    .menu li { 
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;

      clip-path: url(#sector)
    }

    .menu li a {
      display: block;
      width: 100%;
      height: 100%;
    }

    .menu li:hover {
      background-color: gold;
    }

这将使所有的菜单项都变为扇形，然后只有最后一个子项显示，因为重叠了。然后我们需要旋转这些子项

通过`x * angle = x * 60 = 60xdeg`公式来计算
下面是sass代码

    .one {
      background-color: $base;
      transform: rotate(0deg);
    }
    .two {
      background-color: darken($base, 7%);
      transform: rotate(-60deg);
    }
    .three {
      background-color: darken($base, 14%);
      transform: rotate(-120deg);
    }
    .four {
      background-color: darken($base, 21%);
      transform: rotate(-180deg);
    }
    .five {
      background-color: darken($base, 28%);
      transform: rotate(-240deg);
    }
    .six {
      background-color: darken($base, 35%);
      transform: rotate(-300deg);
    }

下面是一个demo

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/9er0xdh7/)

注意，在写这篇文章的时候chrome浏览器对指针事件有Bug,你可以用下面的来代替SVG裁剪

    clip-path: polygon(50% 50%, 100% 50%, 75% 6.6%);

然后这么改了火狐可能不支持，不过你可以把原先那句加到后面备用。

这是polygon()函数裁剪的路径

![menu6](http://img.haoqiao.me//diamond-menu.png)


### 往菜单里增加icons/内容

这个非常简单。你只需要确定icons/文本/其它内容在可见区域里面可视。再次的，用坐标系来确定元素的高和宽。
你可以估计这些子项的位置然后在这些新的扇形区域展现。

![menu7](http://img.haoqiao.me//text-position.png)

这可能需要一些实验才能精确定位。

在我的demo里，我用了文本。所以我制定了30%的下边距和15%的左边距与右边距。这看起来很和谐。

最后，你需要旋转你的内容来达到它确实像在菜单里这个效果。

这是错误的例子

![menu8](http://img.haoqiao.me//text-not-rotated.png)

这才是正确的

![menu9](http://img.haoqiao.me//text-rotated.png)

## 另一种方法

以上技术我们是使用css的路径裁剪。

另一种技术是比较复杂的，因为你需要指定坐标系然后旋转每一个svg`<path>`，更不用说增加内容时所带来的更多工作量。因此我们不能说这个的技术是一个非常好的实现。

但是我还是想要给你看下这个代码长什么样子：
    
    <svg height="0" width="0">
      <defs>
        <clipPath clipPathUnits="userSpaceOnUse" transform="matrix(1,0,0,1,0,0)" id="one-2">
          <path fill="none" stroke="#111" stroke-width="1" class="sector" d="M250,250 l250,0 A250,250 0 0,0 375,33.49364905389035 z"></path>
        </clipPath>
        <clipPath clipPathUnits="userSpaceOnUse" transform="matrix(0.5,-0.86602,0.86602,0.5,-91.5063509461097,341.5063509461096)" id="two-2">
          <path fill="none" stroke="#111" stroke-width="1" class="sector" d="M250,250 l250,0 A250,250 0 0,0 375,33.49364905389035 z"></path>
        </clipPath>
        <clipPath clipPathUnits="userSpaceOnUse" transform="matrix(-0.49999,-0.86602,0.86602,-0.49999,158.49364905389024,591.5063509461097)" id="three-2">
          <path fill="none" stroke="#111" stroke-width="1" class="sector" d="M250,250 l250,0 A250,250 0 0,0 375,33.49364905389035 z"></path>
        </clipPath>
        <clipPath clipPathUnits="userSpaceOnUse" transform="matrix(-1,0,0,-1,500,500)" id="four-2">
          <path fill="none" stroke="#111" stroke-width="1" class="sector" d="M250,250 l250,0 A250,250 0 0,0 375,33.49364905389035 z"></path>
        </clipPath>
        <clipPath clipPathUnits="userSpaceOnUse" transform="matrix(-0.5,0.86602,-0.86602,-0.5,591.5063509461097,158.4936490538905)" id="five-2">
          <path fill="none" stroke="#111" stroke-width="1" class="sector" d="M250,250 l250,0 A250,250 0 0,0 375,33.49364905389035 z"></path>
        </clipPath>
        <clipPath clipPathUnits="userSpaceOnUse" transform="matrix(0.5,0.86602,-0.86602,0.5,341.5063509461096,-91.5063509461097)" id="six-2">
          <path fill="none" stroke="#111" stroke-width="1" class="sector" d="M250,250 l250,0 A250,250 0 0,0 375,33.49364905389035 z"></path>
        </clipPath>
      </defs>
    </svg>

这是完全由SVG代码组成的。

但注意，我并不是手工写的，我之前写了一个生成器来帮助我完成这项工作。（PS:这样好像也是可以接受使用的样子。。。）

### SVG环形菜单

注意如果你用SVG来画，你需要用绝对值而不是相对值，因为你是在cavans画布上绘图。你不需要应用这些扇形到任何元素，因为它们本身就是菜单项。

因此，如果你需要一个环形菜单，svg是一个更好的选择，毕竟所有现代浏览器都支持这个属性。另外你可以获得更加灵活的svg图标来代替传统字体图标，如果你对此感兴趣，你可以看我写的[文章](http://sarasoueidan.com/blog/building-a-circular-navigation-with-svg/)

最后附赠一个作者写的生成器

![menu8](http://img.haoqiao.me//circulus-screenshot.png)

[生成器](http://sarasoueidan.com/tools/circulus/)


## 后记

当前最适合做环形菜单的还是svg，如果你想用css的路径裁剪，至少要等所有浏览器都支持再来。

## 译者后记

在我翻译第一篇Sara Soueidan的文章的时候我就视她为女神，她的文章每一个细节都处理的无可挑剔，厚实的基本功，以及对持着对读者负责态度的各种提醒，还有各种彩蛋引用让人能从她的一篇文章中吸取更多养分。

这篇文章是我翻译的最长的一篇，即使那么长，我还是坚持去翻译它。因为对我而言，翻译是一个学习的过程，当你遇到一篇质量很高的文章时，你会感到由衷的开心。

希望这篇文章能够帮到你。






