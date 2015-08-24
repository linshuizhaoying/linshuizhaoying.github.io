---
layout: post
title: Creating Autocomplete datalist Controls (创建自动完成的DataList控件）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://www.sitepoint.com/creating-autocomplete-datalist-controls/)

### 作者: Aurelio De Rosa 

### 译者: 临水照影


## 正文之前

上一篇感觉状态不错，这篇会继续努力的。

## 正文
      
如果之前你在生活中看到了一些不错的网站，你肯定注意到了一些反复出现的小组件。比如搜索框，通讯组件，文本自动完成框组件。文本自动完成是一种使用非常广泛的组件，特别是当网站需要几个可能出现的字段和需要创建完整的新值。很多javascript框架都用他们自己的方式完成它们自己的自动完成组件。

几年前，并没有这种本地HTML元素来处理这种情况，而且开发者们并没有类似的概念。不过现在这块HTML拼图已经补上了。今天，我们有了一个HTML元素叫做`datalist`来为这种情况服务。在本篇文章中，我们将来讨论`datalist`是什么以及如何用`datalist`。


### 什么是`datalist`元素？


`datalist`元素“代表为其它控件的一组预定义选项内容的元素”。因此这个元素可以被看做是一个包装了一组可能匹配的输入值的。默认的`datalist`里的子元素都是隐藏的，所以你在网页中是看不到这些值的。事实上，`datalist`必须被其它元素用`list`属性来链接。该属性的值必须为`datalist`的ID值。

下面是一个简单的例子：

    <input name="city" list="cities" />
    <datalist id="cities">
     <option value="Naples" />
     <option value="London" />
     <option value="Berlin" />
     <option value="New York" />
     <option value="Frattamaggiore" />
    </datalist>


代码中定义了`input`和包含了一些`Option`元素的`datalist` 。你可以看到，`datalist`的ID为`cities`，`input`的list属性也是这个。

上面代码的[DEMO](http://jsfiddle.net/s8a932s2/)，这次是在jsfiddle里.

因为`datalist` 本身的特性，它很适合和javascript一起使用，举个例子，你可以用ajax请求到服务器来获取值然后来检索用户输入的值。

下面是一个例子，动态生成了`datalist` 里的值。

你可以点[这里](http://jsfiddle.net/4gpyekg2/)

到目前为止，我们已经讨论了自动完成组件的构成，但这并不是我们使用`datalist`的唯一方式。

### `datalist` 和  <input type="color">

之前的例子很棒，但是你可以用`datalist` 做更多的事情，比如你想用<input type="color">提供一个颜色给你的用户？在这个例子里，你可以写下以下代码：
    
    <input type="color" list="colors" />
     <datalist id="colors">
        <option value="#00000"/>
        <option value="#478912"/>
        <option value="#FFFFFF" />
        <option value="#33FF99" />
        <option value="#5AC6D9" />
        <option value="#573905" />
     </datalist>

这个例子只在Chrome 37和Opera24中使用。IE11不支持，FireFox32不支持。(不过你可以把它写在自己项目后台里，这个功能是很实用的)

这是[DEMO](http://jsfiddle.net/7rhe5zuw/)


### `datalist` 和  <input type="range">

另一个例子是是配合`<input type="range">`：
    
    <input type="range" value="0" min="0" max="100" list="numbers" />
    <datalist id="numbers">
        <option value="20" /> 
        <option value="40" /> 
        <option value="60" /> 
        <option value="80" /> 
    </datalist>


所有浏览器都支持这种写法。在这个例子中，范围栏中有四个垂直标志，每个标志都是数据列表中定义的值（PS：你会发现移动这些标志的时候，有一种卡顿会让你停放位置的时候自动对齐标志）

这是这个例子的[DEMO](http://jsfiddle.net/weew16za/)

### 浏览器支持

[CanIUse](http://caniuse.com/#feat=datalist)展示了支持`datalist`这个属性的浏览器列表。事实上，现在你可以很放心的使用，毕竟没啥浏览器不支持了。（除非你是IE10之前，那就GG了）


有一些移动端也是支持的，不过你需要仔细看下版本哦。


### Polyfills

如果你想在一些不支持`datalist`的浏览器上使用`datalist`，你可以用[Relevant Dropdowns](https://github.com/CSS-Tricks/Relevant-Dropdowns)或者[jQuery HTML5 datalist plugin](https://github.com/miketaylr/jquery.datalist.js)

但是你要知道，这些只能替代文本的内容，不能够像color和range那种展开使用。



### 结尾

一堆废话我就不翻译了。。。
























