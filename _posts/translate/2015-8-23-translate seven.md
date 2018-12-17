---
layout: post
title: Responsive Typography With Sass Maps (通过sass地图来打造响应式印刷）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://www.smashingmagazine.com/2015/06/responsive-typography-with-sass-maps/)

### 作者: Jonathan Suh

### 译者: 临水照影


## 正文之前

这应该只是一篇小品文。


## 正文

管理一致的字体印刷不是一件容易的事情，又是是当需要字体能够响应式，事情会变得更加麻烦。Sass Map技术能够让响应式印刷更加易于管理。

写代码是一件事，保持跟踪字体大小值又是另一件事。从`h1`到`h6`，每一段有不同的字体尺寸，跟追会变得很麻烦，尤其是这个类型不能够线性扩展。

如果你想要实现响应式字体，你可能会这么写：

    p { font-size: 15px; }

    @media screen and (min-width: 480px) {
      p { font-size: 16px; }
    }
    @media screen and (min-width: 640px) {
      p { font-size: 17px; }
    }
    @media screen and (min-width: 1024px) {
      p { font-size: 19px; }
    }

Sass变量在项目中可以重复使用，这个特性很棒，但是当它管理字体响应式化，将从简单变成复杂。

    $p-font-size-mobile : 15px;
    $p-font-size-small  : 16px;
    $p-font-size-medium : 17px;
    $p-font-size-large  : 19px;

    $h1-font-size-mobile: 28px;
    $h1-font-size-small : 31px;
    $h1-font-size-medium: 33px;
    $h1-font-size-large : 36px;

    // I think you get the point…（2333）
    
这里将能体现到Sass maps和循环大强大之处。它们可以帮你管理z-index的值(可以看这篇[文章](https://jonsuh.com/blog/organizing-z-index-with-sass/))和颜色（可以看这篇[文章](https://jonsuh.com/blog/sass-maps/#loops-and-maps)）还有你很快就能看到的,字体尺寸。

## 用Sass Maps管理字体尺寸

让我们开始用键值对创建Sass maps，断点（这里可以理解为你所要响应的尺寸）作为键，字体大小作为值

    $p-font-sizes: (
      null  : 15px,
      480px : 16px,
      640px : 17px,
      1024px: 19px
    );

首先我们可以看到 `null`代表默认字体尺寸（并不是在媒体查询中），断点按升序排列。

接下来，在`@mixin`中，它将通过迭代一个Sass map来生成相应的媒体查询

    @mixin font-size($fs-map) {
      @each $fs-breakpoint, $fs-font-size in $fs-map {
        @if $fs-breakpoint == null {
          font-size: $fs-font-size;
        }
        @else {
          @media screen and (min-width: $fs-breakpoint) {
            font-size: $fs-font-size;
          }
        }
      }
    }
注意：值得一提的是该`@mixin`中，混入了一些编程的逻辑。Sass，在[SassScript](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#sassscript)的帮助下实现了类似编程逻辑的 if /else 这类判断,甚至是each循环等。我强烈推荐你花点时间来看文档。该文档将会向你介绍sass的各个方面。

我们可以这么应用该`@mixin`
    
    p {
      @include font-size($p-font-sizes);
    }
    
它将产生以下的css
    
    p { font-size: 15px; }

    @media screen and (min-width: 480px) {
      p { font-size: 16px; }
    }
    @media screen and (min-width: 640px) {
      p { font-size: 17px; }
    }
    @media screen and (min-width: 1024px) {
      p { font-size: 19px; }
    }
    
管理和跟踪这类字体尺寸将变得非常简单。

对每一个新元素，都创建一个map然后调用mixin。

    $h1-font-sizes: (
      null  : 28px
      480px : 31px,
      640px : 33px,
      1024px: 36px
    );

    h1 {
      @include font-size($h1-font-sizes);
    }

然后使各类元素保持一致。
    
     p, ul, ol {
       @include font-size($p-font-sizes);
      }

### 解决断点碎片

但是等等！假如当我们想要响应式700x中将17px 的p 和 33px 的h1 代替640px是，该如何做？为了解决这个问题，这需要我们改变640px中的每个实例。为了解决这个问题，我们又不经意间创建了另一个问题：断点碎片。

如果我们在Sass maps里面管理字体尺寸，我们可以做同样的断点对吗？当然！

让我们创建一个普通断点的map然后给它们取一个合适的名字。我们将这断点Map和字体尺寸Map进行映射建立联系

     $breakpoints: (
       small : 480px,
       medium: 700px, // Previously 640px
       large : 1024px
     );

     $p-font-sizes: (
       null  : 15px,
       small : 16px,
       medium: 17px,
       large : 19px
     );

     $h1-font-sizes: (
       null  : 28px,
       small : 31px,
       medium: 33px,
       large : 36px
     );
    
这最后一步是在迭代字体尺寸map的时候在mixin里面混入一点东西，它在建立媒体查询之前将用断点的名称来从$breakpoints中获取对应的值。

    
    @mixin font-size($fs-map, $fs-breakpoints: $breakpoints) {
      @each $fs-breakpoint, $fs-font-size in $fs-map {
        @if $fs-breakpoint == null {
          font-size: $fs-font-size;
        }
        @else {
          // If $fs-font-size is a key that exists in
          // $fs-breakpoints, use the value
          @if map-has-key($fs-breakpoints, $fs-breakpoint) {
            $fs-breakpoint: map-get($fs-breakpoints, $fs-breakpoint);
          }
          @media screen and (min-width: $fs-breakpoint) {
            font-size: $fs-font-size;
          }
        }
      }
    }

这里用到了map-has-key方法（Returns whether a map has a value associated with a given key.）

    Examples:

    map-has-key(("foo": 1, "bar": 2), "foo") => true
    map-has-key(("foo": 1, "bar": 2), "baz") => false
    
它用来验证这个Key的名字是不是存在于$breakpoints，如果存在，它将用key里的值，否则它会用一个自认为重要的值然后产生媒体查询。

    p { font-size: 15px; }

    @media screen and (min-width: 480px) {
      p { font-size: 16px; }
    }
    @media screen and (min-width: 700px) {
      p { font-size: 17px; }
    }
    @media screen and (min-width: 900px) {
      p { font-size: 18px; }
    }
    @media screen and (min-width: 1024px) {
     p { font-size: 19px; }
    }
    @media screen and (min-width: 1440px) {
      p { font-size: 20px; }
    }


### 提升垂直间距的行高

行高是垂直间距的重要部分。让我们在这个解决方案中包含行高

通过一个列表中的字体尺寸和行高作为键值扩展字体尺寸map。
    
    $breakpoints: (
      small : 480px,
      medium: 700px,
      large : 1024px
    );

    $p-font-sizes: (
      null  : (15px, 1.3),
      small : 16px,
      medium: (17px, 1.4),
      900px : 18px,
      large : (19px, 1.45),
      1440px: 20px,
    );

注意：即使行高值能用任何的CSS单位定义。为了避免继承意外的东西，推荐用无单位行高。(可以点[这里](https://css-tricks.com/almanac/properties/l/line-height/)看详细的内容，还有[这里](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height#Prefer_unitless_numbers_for_line-height_values))

我们需要修改mixin

    @mixin font-size($fs-map, $fs-breakpoints: $breakpoints) {
      @each $fs-breakpoint, $fs-font-size in $fs-map {
        @if $fs-breakpoint == null {
          @include make-font-size($fs-font-size);
        }
        @else {
          // If $fs-font-size is a key that exists in
          // $fs-breakpoints, use the value
          @if map-has-key($fs-breakpoints, $fs-breakpoint) {
            $fs-breakpoint: map-get($fs-breakpoints, $fs-breakpoint);
          }
          @media screen and (min-width: $fs-breakpoint) {
            @include make-font-size($fs-font-size);
          }
        }
      }
    }

    // Utility function for mixin font-size
    @mixin make-font-size($fs-font-size) {
      // If $fs-font-size is a list, include
      // both font-size and line-height
      @if type-of($fs-font-size) == "list" {
        font-size: nth($fs-font-size, 1);
        @if (length($fs-font-size) > 1) {
          line-height: nth($fs-font-size, 2);
        }
      }
      @else {
        font-size: $fs-font-size;
      }
    }

这个mixin验证了一下font-sizes map的值是不是一个字体尺寸列表。如果是一个列表，通过`nth`[函数](http://sass-lang.com/documentation/Sass/Script/Functions.html#nth-instance_method)
将从里面获得正确的值。它假设第一个值是字体尺寸第二个是行高。
    
    p {
     @include font-size($p-font-sizes);
    }
它会生成以下css
    
    p { font-size: 15px; line-height: 1.3; }

    @media screen and (min-width: 480px) {
      p { font-size: 16px; }
    }
    @media screen and (min-width: 700px) {
      p { font-size: 17px; line-height: 1.4; }
    }
    @media screen and (min-width: 900px) {
      p { font-size: 18px; }
    }
    @media screen and (min-width: 1024px) {
      p { font-size: 19px; line-height: 1.45; }
    }
    @media screen and (min-width: 1440px) {
      p { font-size: 20px; }
    }

这个最终方案很容易扩展，像字体宽度，margin等等。关键是用 `make-font-size`和`nth`

## 结论

还有很多方法可以实现响应式字体尺寸，但是我觉得这种对我来说是最好的。

利用mixin可以在你编译css时创建重复的媒体查询，但还有很多争议.[点这里](https://tech.bellycard.com/blog/sass-mixins-vs-extends-the-data/)

我也意识到我的方法不是最好的，但是我觉得我更喜欢手工写复杂的媒体查询。

## 备用方案

Viewport 单位((vh, vw, vmin and vmax)([资料](https://css-tricks.com/viewport-sized-typography/))可以创建响应式印刷。

一个viewport单位等于百分之一viewport宽度或者高度。

Viewport 单位还可以创建[Hero文本](http://demosthenes.info/blog/739/Creating-Responsive-Hero-Text-With-vw-Units)但它不适合用在body里面。它会变得太大或太小。

[Modular Scale](http://www.modularscale.com/)是一个创建响应式印刷的很好的工具。 Sara Soueidan也有篇关于响应式印刷的很好的[文章](http://tympanus.net/codrops/2013/11/19/techniques-for-responsive-typography/)

[Image source](https://www.flickr.com/)帮你把图片制作各种响应式版本提供下载。


## 译者

总的来说，这篇文章能帮助更好的了解sass。至于你用不用他的方案，我觉得这已经不是很重要了-0-不过我反正会去尝试用。因为我也喜欢将能掌握的东西都手写。








