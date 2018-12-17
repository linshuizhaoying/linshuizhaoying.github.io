---
layout: post
title: Understanding the CSS ‘content’ (理解css的content属性）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://www.sitepoint.com/understanding-css-content-property/)

### 作者: Gajendar Singh


### 译者: 临水照影


## 正文之前

这应该也算一篇基础文章，闲暇之余补下基础。

## 正文

如果你是一个前端开发者你一定听过伪属性像[css content 属性](http://www.w3.org/TR/CSS21/generate.html#content)。我不会在这里深入讨论伪元素，但是我建议你看看[这篇文章](http://www.smashingmagazine.com/2011/07/13/learning-to-use-the-before-and-after-pseudo-elements-in-css/)了解更多。

在这篇文章中，我们将专注于content属性。Css的content属性在::before和::after伪元素之中工作。这个属性用来在网页中插入内容，而且所有浏览器都支持。

## Content属性的基础语法

Content语法的细分如下：
    
    p::before {
      content: normal|counter|attr|string|open-quote|url|initial|inherit;
    }

这些值之前的用法略有不同。比如，想在::before或者::after中用attr()，你需要这么写css：
    
    p::after {
      content: " (" attr(title) ")";
    }

这是一个例子，接下来还会有更多。在接下来的部分我们将讨论每个值，包括attr()。

## 值:none或者normal

当设置none,这个伪元素不会被产生。如果你设置normal它将在::before和::after伪元素中计算none。

    p::before {
      content: normal;
    }

    p::after {
      content: none;
    }
## 值:<string>

这个值在content中设置将转为字符串。如果用非拉丁字符，这个字符需要被编码。来看个例子：
    
    <h2>Tutorial Categories</h2>
    <ol>
      <li>HTML and CSS</li>
      <li class="new">Sass &amp; Less</li>
      <li>JavaScript</li>
    </ol>

    <p class="copyright">SitePoint, 2015<p>

然后加下面的css：
    
    .new::after {
      content: " New!";
      color: green;
    }

    .copyright::before {
      content: "\00a9  ";
    }

我们插入了一个文本到列表中，并编码了一个字符到段落元素。

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/s2r4w7dn/)

## 值:<url>

这<url>可以展示你感兴趣的媒体文件。你可以通过它指向外部资源（如图片），但是如果无法访问到资源，它会用占位符代替。让我们来看一些代码：
    
    <a class="sp" href="http://www.sitepoint.com/">SitePoint</a>
    
然后添加css
    
    .sp::before {
      content: url(http://www.sitepoint.com/favicon.ico);
     }
     

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/s2r4w7dn/1/)

## 值:counter()或者counters()

这个值是content属性中最复杂的一个。

你可以看这篇[文章](http://www.sitepoint.com/understanding-css-counters-and-their-use-cases/)来了解更加详细的内容。

我们这里只简要的概述。counter()是用来计数当前class出现的次数。

counter(name,string)计数传进来的第一个参数。

下面是一个例子：

    <h2>Name of First Four Planets</h2>
    <p class="planets">Mercury</p>
    <p class="planets">Venus</p>
    <p class="planets">Earth</p>
    <p class="planets">Mars</p>
    
这是css:
    
    .planets {
      counter-increment: planetIndex;
    }
    .planets::before {
      content: counter(planetIndex) ". ";
    }

这将对每一个子项生成编号。就像有序列表。

这里是[Demo](http://jsfiddle.net/linshuizhaoying/s2r4w7dn/2/)

(译者Ps:可能觉得这个属性好像没什么用，但是在拖拽排序中它可能发挥意想不到的作用，拖拽很难进行重新排序，尤其是获得拖拽结果，但是用了这个属性，它是根据class在页面中第几个显示来计数，你拖拽之后会发现顺序已经更改了。通过取值可以获得排好序的列表。)

## 值：attr()

attr()函数将插入唯一参数的值。

这是例子：
    
    <ul>
      <li><a href="http://www.sitepoint.com/html-css/">HTML and CSS</a></li>
      <li><a href="http://www.sitepoint.com/javascript">JavaScript</a></li>
      <li><a href="http://www.sitepoint.com/mobile/">Mobile</a></li>
    </ul>

下面的css将在内容中显示链接的内容。
   
    a::after {
     content: " (" attr(href) ") ";
    }
 
[Demo Click Here](http://jsfiddle.net/linshuizhaoying/s2r4w7dn/3/)

这通常用来打印样式表。

# 值：open-quote 或 close-quote

当设置这些值，content属性会产生开或者闭合引号。它一般跟<q>元素搭配，但是你可以用在任意元素中。
   
    blockquote::before {
      content: open-quote;
    }

    blockquote::after {
      content: close-quote;
    } 


[Demo Click Here](http://jsfiddle.net/linshuizhaoying/s2r4w7dn/4/)


## 值：no-open-quote 或者 no-close-quote

顾名思义，no-open-quote 移除开引号， no-close-quote移除闭引号

这是例子：
    
    <p><q>A wise man once said: <q>Be true to yourself,
     but don't listen to those who say <q>Don't be true to 
     yourself.</q></q> That is good advice.</q></p>

注意整个段落，整个包含在双引号中，每个引用都是单引号。

添加css后
    
    q {
      quotes: '“' '”' '‘' '’' '“' '”';
    }

    q::before { 
      content: open-quote;
    }

    q::after {
      content: close-quote;
    }
 
这让引用变得正常。

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/s2r4w7dn/5/)

no-open-quote 和 open-quote 可以搭配使用。

[这是例子](http://jsfiddle.net/linshuizhaoying/s2r4w7dn/6/)
