---
layout: post
title: Building Great Mobile Menus (为你的网站创建移动菜单）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://www.sitepoint.com/building-great-mobile-menus-website/)

### 作者: Simon Codrington


### 译者: 临水照影


## 正文之前

像响应式的主题改过不少，很多都有移动端的菜单，以前只知道拿来主义，现在学习一下原理，以后就可以自己开发组件了。

## 正文

在这个教程中我将介绍如何创建友好的移动菜单。


## 移动端菜单

![img](http://7s1say.com1.z0.glb.clouddn.com//mobile1.jpg)

移动端菜单注意事项：

### 更小的设备尺寸

移动端只有很少的物理空间，这意味着我们需要一种组织菜单让它看起来更加易用的方式。移动端菜单通过创建自定义的主菜单来展示其他子菜单来实现我们的需求。

这些菜单应该是响应式的，能够让用户更加方便的浏览它们。

### 触摸屏幕与互动

开发者在创建互动的下拉菜单很依赖:hover和:active。但是在触摸屏上，这些属性并不能很好的工作。（你无法hover一个链接并让它的子菜单像下面这个例子一样出现）

![img2](http://7s1say.com1.z0.glb.clouddn.com//mobile2.jpg)


### 快速响应

对于手机设备而言导航非常重要。因此菜单的响应必须尽可能的快。没有什么比打开一个菜单然后看着它慢吞吞的展开或者关闭动画更糟糕。

一个移动端菜单应该响应的越快越好。

## 可供选择的菜单动画-CSS或者JQuery

你可以选择自己觉得合适的。

### css过渡/变换动画

如果你想要用css来做动画你可以用 transitions 或者 transformations


### transitions 

所有的手机都支持这个属性而且现代浏览器版本支持css3标准。老的浏览器需要用 `-moz` `-webkit` `-ms`前缀。在很多情况下你可以设置`height`或者`left`的值通过transition来让它们动起来。

### transformations

transformations有2d和3d两种变换。所有的浏览器都支持这个属性。举个例子：
    
     2D： transform: translate(50px,0px) 
     3D： transform: translate3d(50px,50px,1px)

这两个属性都很惊人的被所有手机支持。这意味着你可以用它们很好的完成工作。

### Jquery 动画

当我们谈到移动端浏览器，Jquery也被每个浏览器所支持。

## 易于实施/执行

两种方法都很相似而且很容易使用。然而当创建复杂的动画或优化动画速度时，难度往往会扩大。

### CSS transitions / transformations

CSS提供的属性很容易使用，设置开始结束值浏览器会自动让它们动起来。

举个例子，移动一个box：
    
    /*Move an item when hovering over it*/
    .my-container .my-box{
        left: 0px;
        transition: all 350ms linear;
        position: relative;
    }
    .my-container:hover .my-box{
        left: 100px;
    }
    
从一端移动到另一端，浏览器会自动填充过渡的距离。

对于更加复杂的动画或者一个未知的动值，计算需要用到javascript.

### Jquery 动画

用Jquery动画函数很容易达到想要的效果：
    
    //On hover, move the box inside left or right
    $('.my-container').hover(
      function() {
        $('.my-box').animate({left: "100px"},500);
      }, function() {
        $('.my-box').animate({left: "0px"},500);
      });

自从Jquery提供了动画函数，浏览器支持不再是一个问题。不再需要对不同的浏览器用不同版本的动画函数。

### 速度和响应

响应能力和速度是移动界面重要的组成部分，它影响到创建者者的选择。

### CSS transitions / transformations

css的动画一直比Jquery动画函数快。即使Jquery已经优化到现在的版本。css提供的动画还是更加平滑和更快。它直接让浏览器处理动画。如果没有硬件加速，浏览器将处理一切计算。如果有GPU加速，可以加快计算的过程。

### CSS 过渡

    /*transition an item with its top value when its active*/
    .transition-item{
        position: relative;
        top: 0px;
        transition: top 300ms ease;
    }
    .transition-item.active{
        top: 300px;
    }

在这个例子中当transition-item添加active，它会移动到距离顶端300px，这个过渡是很快的，但是它不会触发硬件加速。

### CSS 变换

    /*Apply an animation when the item is in its active state*/
    .animation-item{
        position: relative;
        transition: top 300ms ease;
        transform: translate3d(0px,-300px,1px);
    }
    .animation-item.active{
        transform: translate3d(0px,0px,1px);
    }

这个3D变换会触发硬件加速。

### 用3D变换来让菜单滑出

在最初，菜单置于屏幕左上角。当你触发菜单按钮，它将会3D旋转移动。

这个动画函数用javascript来移动元素。

虽然这个方法能工作，但是它还是回产生一种滞后或者卡顿的现象。

这种动画会让人觉得网站很慢。


[Click Demo ](http://jsfiddle.net/linshuizhaoying/srng416p/)
    
这个菜单用transformations 和 transitions 组合应用到旋转/滑动动画。

这个菜单本身绝对定位而且是windows的65%的宽度。

(ps：具体原理我就不翻译了，直接看demo里的代码已经很直观了。)

### 弹出菜单滑出子菜单

这个菜单有点不同。当我们切换菜单时它将他出一个导航菜单（配合淡入效果从屏幕中心弹出。）而且子菜单中还能再链接子菜单。

这是[DEMO](http://jsfiddle.net/linshuizhaoying/srng416p/1/)

`nav-menu`元素应用了`overflow:hidden`而且一开始透明度设为0以及一个`scale3d`为0.5

当它活动时，`nav-menu-toggle`会切换`nav-menu`的状态。如果菜单被打开，它会设置一个scale3d到1的变换。透明度也将变为1.(这将使它有从中心弹出的效果)

每一个子菜单又在`sub-menu `元素中。有子项的菜单将有一个浮动的元素在右边。

当这个子菜单的子项被触发，它将滑动并替换旧的原来的菜单。


## 译者

作者只讲了大概效果以及触发的行为，还是建议多看看demo然后自己尝试去实现。

























