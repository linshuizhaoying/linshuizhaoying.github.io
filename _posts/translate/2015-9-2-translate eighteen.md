---
layout: post
title: A Visual Guide to CSS3 Flexbox (CSS3 Flexbox 可视化教程）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](https://scotch.io/tutorials/a-visual-guide-to-css3-flexbox-properties)

### 作者: Dimitar Stojanov


### 译者: 临水照影


## 正文之前

Flex好像火了很久，但是以前我还在用基本的栅格系统，而且好像够用了就没怎么仔细去研究。不过今天抽到了这篇库存里的翻译就好好学习一下。

## 正文

Flexbox Layout 官方名是[CSS Flexible Box Layout Module](http://www.w3.org/TR/css-flexbox/)

它是一种新的css3布局模块，可以提高在容器内项目布局和秩序的能力，即使它们是动态添加，大小未知。Flexbox主要的能力就是对容器内的元素进行调整宽度高度以适应不同的屏幕尺寸。

很多设计师和开发者发现flexbox非常容易使用。用更少的代码实现更复杂的布局。Flexbox因为更适用与小的应用组件，而[新的css栅格布局模块](http://www.w3.org/TR/css-grid/)更适于处理大规模布局。

这个教程将专注于可视化flex属性的效果。而不是解释flex属性的工作原理。

## 基础

在开始介绍flexbox属性之前我们先来介绍一下flexbox模型。flex 布局是由叫做`flex container`的父容器和称为`flex item`的直属子容器组成。

![img1](http://7s1say.com1.z0.glb.clouddn.com//CSS3-Flexbox-Model.jpg)

在这个盒子中你可以看到属性和描述flex容器及子容器的术语。更多你可以看官网的[flexbox model](http://www.w3.org/TR/css-flexbox/#box-model)

如何在老的浏览器中使用flebox，你可以看这篇[文章](https://css-tricks.com/using-flexbox/)

这是最新的浏览器支持情况:
    
    Chrome 29+
    Firefox 28+
    Internet Explorer 11+
    Opera 17+
    Safari 6.1+ (prefixed with -webkit-)
    Android 4.4+
    iOS 7.1+ (prefixed with -webkit-)

当然你可以[点这里](http://caniuse.com/#feat=flexbox)看近期的支持情况

## 用法

用flexbox 布局仅仅需要在`display`属性中设置:
    
    .flex-container {
      display: -webkit-flex; /* Safari */
      display: flex;
    }

或者你想要在内联元素中使用：

    .flex-container {
      display: -webkit-inline-flex; /* Safari */
      display: inline-flex;
    }
    
注意：这个属性你只应该设置在父容器中，而且该容器的直属子容器会立刻自动变为`flex items`

# FlexBox 容器属性

## flex-direction

这个属性指定在flex容器中flex item该如何布局。通过设置flex容器主轴的方向，它们可以布局成两种方向，水平或者垂直。

用法：
    
    .flex-container {
      -webkit-flex-direction: row; /* Safari */
      flex-direction:         row;
    }
    
通过`row`设置上下文从左到右方向

    .flex-container {
      -webkit-flex-direction: row-reverse; /* Safari */
      flex-direction:         row-reverse;
    }
    
    
![imgn2](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-direction-row.jpg)

通过`row-reverse`设置上下文从右到左方向

    .flex-container {
      -webkit-flex-direction: column; /* Safari */
      flex-direction:         column;
    }

![imgn3](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-direction-row-reverse.jpg)


通过` column `设置上下文从上到下方向

    .flex-container {
      -webkit-flex-direction: column; /* Safari */
      flex-direction:         column;
    }


![imgn4](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-direction-column.jpg)



通过` column-reverse `设置上下文从下到上方向


    .flex-container {
      -webkit-flex-direction: column-reverse; /* Safari */
      flex-direction:         column-reverse;
    }



![imgn5](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-direction-column-reverse.jpg)


### 默认值: row

注意:`row` 和 `row-reverse` 都是根据书写方式因此在`rtl`上下文中它们将分别逆转。

## flex-wrap

flex-wrap 控制 子容器新起一行。

例子：
    
    .flex-container {
      -webkit-flex-wrap: nowrap; /* Safari */
      flex-wrap:         nowrap;
    }

flex项目显示在一行，而且默认情况下它们会被缩小到适应flex容器的宽度


![imgn5](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-wrap-nowrap.jpg)


    .flex-container {
      -webkit-flex-wrap: wrap; /* Safari */
      flex-wrap:         wrap;
    }

flex项目将自动根据需求新起一行。

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-wrap-wrap.jpg)

    .flex-container {
      -webkit-flex-wrap: wrap-reverse; /* Safari */
      flex-wrap:         wrap-reverse;
    }

flex将显示多行并逆向显示

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-wrap-wrap-reverse.jpg)

### 默认值：nowrap

注意：这些属性都是根据书写方式因此在`rtl`上下文中它们将分别逆转。


## flex-flow

这个属性是简化设置`flex-direction` 和 `flex-wrap`。

例子：
    
    .flex-container {
      -webkit-flex-flow: <flex-direction> || <flex-wrap>; /* Safari */
      flex-flow:         <flex-direction> || <flex-wrap>;
    }

默认值是:row nowrap

## justify-content

这个属性让flex items沿着当前主容器的轴对齐，它可以帮助分配剩余空间。

例子：

根据上下文从左侧开始对齐

    .flex-container {
      -webkit-justify-content: flex-start; /* Safari */
      justify-content:         flex-start;
    }


根据上下文从右侧对齐

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-justify-content-flex-start.jpg)

    .flex-container {
      -webkit-justify-content: flex-end; /* Safari */
      justify-content:         flex-end;
    }


![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-justify-content-flex-end.jpg)

根据上下文居中对齐


    .flex-container {
      -webkit-justify-content: center; /* Safari */
      justify-content:         center;
    }
    
![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-justify-content-center.jpg)


将容器的所有空间分配，每个子容器分配相同的间距，首尾两项沿着主容器边沿

    .flex-container {
      -webkit-justify-content: space-between; /* Safari */
      justify-content:         space-between;
    }

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-justify-content-space-between.jpg)

将容器的所有空间分配，每个子容器分配相同的间距，首尾也如此

    .flex-container {
      -webkit-justify-content: space-around; /* Safari */
      justify-content:         space-around;
    }
    
![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-justify-content-space-around.jpg)

默认值：flex-start 


## align-items

Flex items可以根据当前横轴线对齐，像`justify-content`一样但是是在垂直方向。

这个属性默认对齐所有flex items，包括匿名的。

例子：
    
    .flex-container {
      -webkit-align-items: stretch; /* Safari */
      align-items:         stretch;
    }

Flex items填补整个flex容器的高度(或者宽度)

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-items-stretch.jpg)

    .flex-container {
      -webkit-align-items: flex-start; /* Safari */
      align-items:         flex-start;
    }

从容器交叉开始处堆放

    .flex-container {
      -webkit-align-items: flex-start; /* Safari */
      align-items:         flex-start;
    }

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-items-flex-start.jpg)

从容器交叉结束处堆放

    .flex-container {
      -webkit-align-items: flex-end; /* Safari */
      align-items:         flex-end;
    }
    
    
![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-items-flex-end.jpg)   


从容器交叉轴中心堆放

    .flex-container {
      -webkit-align-items: center; /* Safari */
      align-items:         center;
    }

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-items-center.jpg)

根据它们的基线对齐

    .flex-container {
      -webkit-align-items: baseline; /* Safari */
      align-items:         baseline;
    }

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-items-baseline.jpg)

### 默认值:stretch

注意：关于基线内容更多请点[这里](http://www.w3.org/TR/css-flexbox/#flex-baselines)

## align-content

这个属性对齐flex容器的在交叉轴有额外空间的flex容器。就像`justify-content`对齐独立在主线轴的子项。

例子：
    
    .flex-container {
      -webkit-align-content: stretch; /* Safari */
      align-content:         stretch;
    }

对每一排都分配相同空间

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-content-stretch.jpg)


从交叉轴开始处堆放

    .flex-container {
      -webkit-align-content: flex-start; /* Safari */
      align-content:         flex-start;
    }

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-content-flex-start.jpg)


从交叉轴的结尾处堆放
    
    .flex-container {
      -webkit-align-content: flex-end; /* Safari */
      align-content:         flex-end;
    }
    
![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-content-flex-end.jpg)

从容器交叉轴中心堆放

    .flex-container {
      -webkit-align-content: center; /* Safari */
      align-content:         center;
    }

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-content-center.jpg)

flex items的每一行都间隔相同距离，第一行和最后一行沿着边沿对齐

    .flex-container {
      -webkit-align-content: space-between; /* Safari */
      align-content:         space-between;
    }

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-content-space-between.jpg)


flex items的每一行都间隔相同距离



    .flex-container {
      -webkit-align-content: space-around; /* Safari */
      align-content:         space-around;
    }
    
![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-content-space-around.jpg)
    
### 默认值：stretch

注意：该属性只影响多行内容，不影响单行。    
    

## flext containers的注意点

所有 `column-*`属性在flex容器内无效果

`::first-line` 和 `::first-letter`无法再flex容器内应用。


# FLEXBOX ITEM 属性

## order

order属性用来对flex容器内的子项进行排序。

例子：
    
    .flex-item {
      -webkit-order: <integer>; /* Safari */
      order:         <integer>;
    }
    
它可以让子项重新排序而不需要重组html代码。

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-order.jpg)    

### 默认值:0


## flex-grow    
    
这个属性直接看例子更有效果：

    .flex-item {
      -webkit-flex-grow: <number>; /* Safari */
      flex-grow:         <number>;
    }    
    
如果它们有相同`flex-grow`值它们将有相同大小。

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-grow-1.jpg)

第二项是其它项的三倍

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-grow-2.jpg)
    
### 默认值:0

注意:负数是无效的

## flex-shrink    

默认所有子项都是收缩的。

    .flex-item {
      -webkit-flex-shrink: <number>; /* Safari */
      flex-shrink:         <number>;
    }

如果你设置为0，它将采用原来的尺寸。

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-shrink.jpg)
    
### 默认值:1

注意:负数是无效的 
    
## flex-basis

例子：

    .flex-item {
      -webkit-flex-basis: auto | <width>; /* Safari */
      flex-basis:         auto | <width>;
    }

`flex-basis`决定了初始大小。

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-flex-basis.jpg)

### 默认值:auto

注意，auto这个命名问题将在未来解决。
    
## flex

简化版 `flex-grow`, `flex-shrink` 和 `flex-basis`

可以设置为 auto (1 1 auto) 或者 none (0 0 auto).

    .flex-item {
      -webkit-flex: none | auto | [ <flex-grow> <flex-shrink>? || <flex-basis> ]; /* Safari */
      flex:         none | auto | [ <flex-grow> <flex-shrink>? || <flex-basis> ];
    }

### 默认值：0 1 auto



## align-self

子项可以自己设置对齐方式

    .flex-item {
      -webkit-align-self: auto | flex-start | flex-end | center | baseline | stretch; /* Safari */
      align-self:         auto | flex-start | flex-end | center | baseline | stretch;
    }

![imgn](http://7s1say.com1.z0.glb.clouddn.com//flexbox-align-self.jpg)

### 默认值:auto

## flex items 注意点

`float`, `clear` 和 `vertical-align` 对flex item没有效果。

## FLEXBOX PLAYGROUND

这是一个能够帮助你理解flexbox的工具

[Demo Click Here](http://codepen.io/linshuizhaoying/pen/OyJrQr)

也可以在github中查看[源码](https://github.com/linshuizhaoying/flexbox-playground)




## 译者

翻译了大概3小时-0-，总的来说，学完这个基本就能上手，感觉用在vue的组件里好像挺不错的。









