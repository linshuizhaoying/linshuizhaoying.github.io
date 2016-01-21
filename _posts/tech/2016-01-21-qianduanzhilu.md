---
layout: post
title: 从checkbox开始
category: 技术
tags: [js,设计模式,jsfiddle,Gitbook,折腾,积累,Css,Css3]
keywords: 前端,资料,学习,javascript,Css
description: 
---

# 前言

最近在捣鼓自己的重构项目，大概也写了接近半个月，总算快写完了。然后我最近又开始整理了一本GitBook<<前端之路>>。

简介如下:

```

前端之路这本书将逐步汇总一些个人觉得身为前端所需要掌握的所需要知道的技能。

它将包括


css之路
它包含平时阅读所看到的优秀文章或者例子，本人将溯本求源，力争取将原理与实践结合，能够准确的在工作中复用。

js之路

同css之路类似。它也将理论与实践结合。

前端之路

这里会包含前端工程化，组件化，es6，webpack，等等理论的整理与实践的结合。

```

之所以整理这么一本持续性添加的gitbook，是因为我有时候看完文章就结束了，或者GET几个点作为积累或者用印象笔记保存。但是这些对于深入学习而言并不是很理想。因为你存的内容你不一定看，看了你不一定深入。因此我打算搞这么一本gitbook，将一些我看到的我觉得很棒的内容进行深入学习分析，然后再进行举一反三写demo，能够让我在工作或者自己的项目中快速复用这些知识。

这本书最终会开源，应该是在每个板块内容至少都有10篇文章左右吧=0=。可以来看下现在的进度：

![imgn](http://haoqiao.qiniudn.com/active76.gif)

如果你看到比较优秀的文章欢迎发给我。（国内国外皆可。要求就是最好都有demo。）

现在来看一篇已经整理过的。

# 正文

## 背景：

这是网上看到一篇译文，然后点了[原文链接](http://codersblock.com/blog/checkbox-trickery-with-css/)，发现原作者其实写了很多总结性的文章。因此从这篇来开头一点点学习。

## 正文

![imgn](http://haoqiao.qiniudn.com/active61.gif)

可以看到是很常见的效果。我们来看它是如何实现的.

code:

```
<div class="container">
  <input id="toggle1" type="checkbox" checked>
  <label for="toggle1">Toggle me!</label>
</div>

.container {
  position: absolute;
  top: 50%;
  left: 50%;
  -webkit-transform: translate(-50%, -50%);
          transform: translate(-50%, -50%);
}

input {
  position: absolute;
  left: -9999px;
}

label {
  display: block;
  position: relative;
  margin: 20px;
  padding: 15px 30px 15px 62px;
  border: 3px solid #fff;
  border-radius: 100px;
  color: #fff;
  background-color: #6a8494;
  box-shadow: 0 0 20px rgba(0, 0, 0, .2);
  white-space: nowrap;
  cursor: pointer;
  -webkit-user-select: none;
     -moz-user-select: none;
      -ms-user-select: none;
          user-select: none;
  -webkit-transition: background-color .2s, box-shadow .2s;
  transition: background-color .2s, box-shadow .2s;
}

label::before {
  content: '';
  display: block;
  position: absolute;
  top: 10px;
  bottom: 10px;
  left: 10px;
  width: 32px;
  border: 3px solid #fff;
  border-radius: 100px;
  -webkit-transition: background-color .2s;
  transition: background-color .2s;
}

label:hover, input:focus + label {
  box-shadow: 0 0 20px rgba(0, 0, 0, .6);
}

input:checked + label {
  background-color: #ab576c;
}

input:checked + label::before {
  background-color: #fff;
}

```

`container`样式是用来确定位置以及大小我们可以跳过。


```
input {
  position: absolute;
  left: -9999px;
}
```

这段是为了隐藏input.文章中提到为什么不用display:none，这是因为你如果用display:none的话在多个checkbox时你按键盘tab键是无法选中下一个单选框。

接下来的label也很好分析就是简单的按钮样式。

```

label::before {
  content: '';
  display: block;
  position: absolute;
  top: 10px;
  bottom: 10px;
  left: 10px;
  width: 32px;
  border: 3px solid #fff;
  border-radius: 100px;
  -webkit-transition: background-color .2s;
  transition: background-color .2s;
}

```

这段利用伪元素画了一个圆圈，通过` position: absolute`以及设定bottom，left,right,top，定位在复选框左侧。

最关键的代码如下:

```

input:checked + label {
  background-color: #ab576c;
}

input:checked + label::before {
  background-color: #fff;
}

```
利用checked伪类和相邻兄弟元素选择器(+)组合实现选中Input元素后面紧跟着的label。

`input:checked + label::before` 利用伪元素将之前画的圆进行填色。


一次为基础可以来看下扩展的效果:

![imgn](http://haoqiao.qiniudn.com/active62.gif)

以上效果也是纯css来完成的。

我们可以先不看源码来考虑，如果是你，类似点击然后显示的效果你会怎么写？

首先肯定是隐藏一个div，然后选中内容的时候把它显示了。

我们可以先在第一个例子里先试试:

![imgn](http://haoqiao.qiniudn.com/active63.gif)

看看我们修改的代码

```
html:
  <input id="toggle1" type="checkbox" checked>
  <label for="toggle1">Toggle me!</label>
  <div class="test">
    <h1>Linshuizhaoying测试</h1>
  </div>

css3:

.test{
 display:none;
}

input:checked ~ .test{
  display:block;
}

```

我们可以发现关键改的地方是这里`input:checked ~ .test`

查看[mdn文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS/General_sibling_selectors),可以看到如下描述

```

在使用 ~ 连接两个元素时,它会匹配第二个元素,条件是它必须跟(不一定是紧跟)在第一个元素之后,且他们都有一个共同的父元素 .

```

清楚原理了我们回头来看扩展版的代码:

```

html：

<div class="container">
  <section>
    <input id="how-internet" name="how" type="radio">
    <label for="how-internet" class="side-label">Somewhere on the internet</label>

    <input id="how-other" name="how" type="radio">
    <label for="how-other" class="side-label">Other...</label>

    <div class="how-other-disclosure">
      <label for="how-other-explain" class="top-label">Please explain</label>
      <textarea id="how-other-explain"></textarea>
    </div>
  </section>
  <section>
    <input id="permitted" type="checkbox">
    <label for="permitted" class="side-label">I am legally permitted to submit forms</label>
    <div class="blocked">(you have to check the checkbox to continue)</div>
    <button>Submit</button>
  </section>
</div>

css3:


*, *::before, *::after {
  box-sizing: border-box;
}

.container {
  max-width: 500px;
  margin: 0 auto;
  padding: 10px 20px 40px;
  color: #fff;
  background-color: rgba(0, 0, 0, .6);
}



.top-label {
  display: block;
  margin: 10px 0;
}

input[type="checkbox"], input[type="radio"] {
  position: absolute;
  left: -9999px;
}

.side-label {
  display: block;
  position: relative;
  margin: 10px 0;
  padding-left: 35px;
  cursor: pointer;
}

.side-label::before, .side-label::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
}

input[type="radio"] + .side-label::before,
input[type="radio"] + .side-label::after {
  border-radius: 50%;
}

.side-label::before {
  display: block;
  width: 20px;
  height: 20px;
  border: 2px solid #fff;
}

.side-label::after {
  display: none;
  width: 12px;
  height: 12px;
  margin: 4px;
  background-color: #9ab593;
}

input:checked + .side-label::after {
  display: block;
}

.how-other-disclosure {
  display: none;
  margin: 10px 0 0 35px;
}

#how-other:checked ~ .how-other-disclosure {
  display: block;
}

button {
  display: none;
  -webkit-appearance: none;
     -moz-appearance: none;
          appearance: none;
  margin: 40px auto 0;
  padding: 5px 40px;
  border: 2px solid #fff;
  color: rgba(0, 0, 0, .8);
  background-color: #9ab593;
  font-family: inherit;
  font-size: inherit;
  font-weight: bold;
  cursor: pointer;
}

#permitted:checked ~ .blocked {
  display: none;
}

#permitted:checked ~ button {
  display: block;
}

```

我们先来看一下这段

```

*, *::before, *::after {
  box-sizing: border-box;
}

```

事实上很多地方你都能看到类似代码，但很多时候都不会去在意为什么这么做，只要能用不就好了的心态导致至今都没清楚原理。

通过查阅资料我们可以查到如下信息：

```
 box-sizing 属性用来改变默认的 CSS 盒模型 对元素高宽的计算方式。

content-box
默认值，标准盒模型。 width 与 height 只包括内容的宽和高， 不包括边框，内边距，外边距。注意: 内边距, 边框 & 外边距 都在这个盒子的外部。 比如. 如果 .box {width: 350px}; 而且 {border: 10px solid black;} 那么在浏览器中的渲染的实际宽度将是370px;

 border-box
 width 与 height 包括内边距与边框，不包括外边距。这是IE 怪异模式（Quirks mode）使用的 盒模型 。注意：这个时候外边距和边框将会包括在盒子中。比如  .box {width: 350px; border: 10px solid black;} 浏览器渲染出的宽度将是350px.


当你设置一个元素为 box-sizing: border-box; 时，此元素的内边距和边框不再会增加它的宽度

```

当然这么一堆资料给你你可能过一遍会发现，唉，好像有点道理。然后伪装自己好像知道了的样子。。。

但其实还是不明白是不是。

所以来看这张图：

![imgn](http://haoqiao.qiniudn.com/active64.gif)

仔细看这张图几分钟，我们可以看到

当我们将`box-sizing`设置为border-box时，伪元素内部的宽高都是16px，而我们设置`box-sizing`设置为content-box.伪元素内部的宽高都是20px.而且它还有宽度为2的边框。

解决这个问题我们继续往下走，

```

input[type="checkbox"], input[type="radio"] {
  position: absolute;
  left: -9999px;
}

```

这段是不是很熟悉。和第一个demo类似，将原生的复选框和单选框都隐藏起来。

之后的就不需要分析了，因为我们之前都讲过了。

然后我们继续看下一个内容。在看一个例子之前我先去对本地fiddle调试工具加上awesome.min.css的引用。

![imgn](http://haoqiao.qiniudn.com/active65.gif)

这个例子我们也可以在看源码之前把它分析部分

```
首先，点击了文件夹小图标，图标变换，这说明改动了content内容。

内容显示隐藏应该跟之前的display类似。

但是点击按钮两个都隐藏，不怎么清楚，因此重点看这个。

```

然后就是继续贴源码：

```
html:


<form>
  <div class="tree">
    <div>
      <input id="n-0" type="checkbox">
      <label for="n-0">Black</label>
      <div class="sub">
        <a href="#link">Plague Rats</span>
        <a href="#link">Sengir Vampire</a>
      </div>
    </div>
    <div>
      <input id="n-1" type="checkbox">
      <label for="n-1">Blue</label>
      <div class="sub">
        <a href="#link">Mana Leak</a>
        <a href="#link">Time Warp</a>
      </div>
    </div>



  </div>
  <input type="reset" value="Collapse All">
</form>


css3:


*, *::before, *::after {
  box-sizing: border-box;
}


.tree {
  padding: 20px 0;
}

.tree::after {
  content: '';
  display: block;
  clear: left;
}

.tree div {
  clear: left;
}

input[type="checkbox"] {
  position: absolute;
  left: -9999px;
}

label, a {
  display: block;
  float: left;
  clear: left;
  position: relative;
  margin-left: 25px;
  padding: 5px;
  border-radius: 5px;
  color: #5c5d5e;
  text-decoration: none;
  cursor: pointer;
}

label::before, a::before {
  display: block;
  position: absolute;
  top: 6px;
  left: -25px;
  font-family: 'FontAwesome';
}

label::before {
  content: '\f07b'; /* closed folder */
}

input:checked + label::before {
  content: '\f07c'; /* open folder */
}

a::before {
  content: '\f068'; /* dash */
}

input:focus + label, a:focus {
  outline: none;
  background-color: #b9c3c9;
}

.sub {
  display: none;
  float: left;
  margin-left: 30px;
}

input:checked ~ .sub {
  display: block;
}

input[type="reset"] {
  display: block;
  width: 100%;
  padding: 10px;
  border: none;
  border-radius: 10px;
  color: #e8ebed;
  background-color: #6b7c87;
  font-family: inherit;
  font-size: .9em;
  cursor: pointer;
  -webkit-appearance: none;
  -moz-appearance: none;
}

input[type="reset"]:focus {
  outline: none;
  box-shadow: 0 0 0 4px #b9c3c9;
}


```

其他很多东西我们之前都分析过因此跳过。然后发现这段

```

input[type="reset"] {
  display: block;
  width: 100%;
  padding: 10px;
  border: none;
  border-radius: 10px;
  color: #e8ebed;
  background-color: #6b7c87;
  font-family: inherit;
  font-size: .9em;
  cursor: pointer;
  -webkit-appearance: none;
  -moz-appearance: none;
}

```

通过表单的重置按钮将所有复选框重置。

然后我们再来看一个比较惊艳的效果:

![imgn](http://haoqiao.qiniudn.com/active66.gif)


这个效果非常炫酷，而且也是纯css制作。基于一开始什么也分析不出来我们直接看源码:

```

html:


<div class="container">
  <h1>To-Do List</h1>
  <div class="items">
    <input id="item1" type="checkbox" checked>
    <label for="item1">Create a to-do list</label>

    <input id="item2" type="checkbox">
    <label for="item2">Take down Christmas tree</label>

    <input id="item3" type="checkbox">
    <label for="item3">Learn Ember.js</label>



    <h2 class="done" aria-hidden="true">Done</h2>
    <h2 class="undone" aria-hidden="true">Not Done</h2>
  </div>
</div>


css3:

@import url(http://fonts.googleapis.com/css?family=Roboto:500,700);

*, *::before, *::after {
  box-sizing: border-box;
}

html {
  min-height: 100%;
}

body {
  margin: 20px;
  color: #435757;
  background: linear-gradient(-20deg, #d0b782 20%, #a0cecf 80%);
  font: 500 1.2em/1.2 'Roboto', sans-serif;
}

.container {
  max-width: 450px;
  margin: 0 auto;
  border-top: 5px solid #435757;
  background-color: rgba(255, 255, 255, .2);
  box-shadow: 0 0 20px rgba(0, 0, 0, .1);
  user-select: none;
}

h1 {
  margin: 0;
  padding: 20px;
  background-color: rgba(255, 255, 255, .4);
  font-size: 1.8em;
  text-align: center;
}

.items {
  display: flex;
  flex-direction: column;
  padding: 20px;
  counter-reset: done-items undone-items;
}

h2 {
  position: relative;
  margin: 0;
  padding: 10px 0;
  font-size: 1.2em;
}

h2::before {
  content: '';
  display: block;
  position: absolute;
  top: 10px;
  bottom: 10px;
  left: -20px;
  width: 5px;
  background-color: #435757;
}

h2::after {
  display: block;
  float: right;
  font-weight: normal;
}

.done {
  order: 1;
}

.done::after {
  content: ' (' counter(done-items) ')';
}

.undone {
  order: 3;
}

.undone::after {
  content: ' (' counter(undone-items) ')';
}

/* hide inputs offscreen, but at the same vertical positions as the correpsonding labels, so that tabbing scrolls the viewport as expected */
input {
  display: block;
  height: 53px;
  margin: 0 0 -53px -9999px;
  order: 4;
  outline: none;
  counter-increment: undone-items;
}

input:checked {
  order: 2;
  counter-increment: done-items;
}

label {
  display: block;
  position: relative;
  padding: 15px 0 15px 45px;
  border-top: 1px dashed #fff;
  order: 4;
  cursor: pointer;
  animation: undone .5s;
}

label::before {
  content: '\f10c'; /* circle outline */
  display: block;
  position: absolute;
  top: 11px;
  left: 10px;
  font: 1.5em 'FontAwesome';
}

label:hover, input:focus + label {
  background-color: rgba(255, 255, 255, .2);
}

input:checked + label {
  order: 2;
  animation: done .5s;
}

input:checked + label::before {
  content: '\f058'; /* circle checkmark */
}

@keyframes done {
  0% {
    opacity: 0;
    background-color: rgba(255, 255, 255, .4);
    transform: translateY(20px);
  }
  50% {
    opacity: 1;
    background-color: rgba(255, 255, 255, .4);
  }
}
@keyframes undone {
  0% {
    opacity: 0;
    background-color: rgba(255, 255, 255, .4);
    transform: translateY(-20px);
  }
  50% {
    opacity: 1;
    background-color: rgba(255, 255, 255, .4);
  }
}

```

事实上当看到

```

.items {
  display: flex;
  flex-direction: column;
  padding: 20px;
  counter-reset: done-items undone-items;
}

```

我们就知道这个效果其实是根据flex的特性来完成的。

```

CSS flexbox 可以使用 order 属性重排元素。当复选框选中的时候，<label> 的 order 值由 4 变为 2，列表选项就从 “Not Done” <h2> 下面移到了 “Done” <h2> 下面。

```

比较好奇的是conter-reset属性，查一下文档:


```
counter-reset

The counter-reset CSS property is used to reset CSS Counters to a given value.

光看描述可能看不出来什么意思，我们看下mdn上的例子

/* Set counter-name to 0 */
counter-reset: counter-name;

/* Set counter-name to -1 */
counter-reset: counter-name -1;

/* Set counter1 to 1, and counter2 to 4 */
counter-reset: counter1 1 counter2 4;

```

这样我们大概了解`  counter-reset: done-items undone-items;` 是在css中定义了两个counter计数变量，它们值一开始为0。等同于`  counter-reset: done-items 0 undone-items 0;`

关于Order的解释我从[w3cplus](http://www.w3cplus.com/css3/a-visual-guide-to-css3-flexbox-properties.html)上引用一张图

`order属性是用来控制flex容器中flex项目的排列顺序。默认情况flex项目在flex容器的顺序是flex项目出现的顺序。`

![imgn](http://haoqiao.qiniudn.com/flexbox-order.jpg)

` counter-increment` 是对其中counter变量自增1

`counter-increment 属性设置某个选取器每次出现的计数器增量。默认增量是 1。`

经过一些测试，发现如果你把`counter-reset`写在一些内容项的class里，你的计数会出现错误，因此把`counter-reset`写在你要计数的内容的父元素上是比较正确的。


最后一个例子是根据data里内容来做筛选，这里我不展开分析，只记录一下效果以及源码。

![imgn](http://haoqiao.qiniudn.com/active67.gif)

源码:


```

html:
<!--
  Checkbox Trickery with CSS:
  http://codersblock.com/blog/checkbox-trickery-with-css/
-->

<div class="container">
  <h1><span class="marvel">Marvel</span> Mutant Teams</h1>

  <input id="original" type="radio" name="team" checked>
  <label for="original">Original X-Men</label>

  <input id="force" type="radio" name="team">
  <label for="force">X-Force</label>

  <input id="factor" type="radio" name="team">
  <label for="factor">X-Factor</label>

  <input id="hellfire" type="radio" name="team">
  <label for="hellfire">Hellfire Club</label>

  <br>
  <ul class="characters">
    <li id="angel" data-teams="original force factor hellfire">
      <h2>Angel</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-angel.png" alt="">
    </li>
    <li id="beast" data-teams="original factor">
      <h2>Beast</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-beast.png" alt="">
    </li>
    <li id="cyclops" data-teams="original force factor">
      <h2>Cyclops</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-cyclops.png" alt="">
    </li>
    <li id="emma-frost" data-teams="hellfire">
      <h2>Emma Frost</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-emma-frost.png" alt="">
    </li>
    <li id="iceman" data-teams="original factor">
      <h2>Iceman</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-iceman.png" alt="">
    </li>
    <li id="jean-grey" data-teams="original factor">
      <h2>Jean Grey</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-jean-grey.png" alt="">
    </li>
    <li id="magneto" data-teams="hellfire">
      <h2>Magneto</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-magneto.png" alt="">
    </li>
    <li id="nightcrawler" data-teams="force">
      <h2>Nightcrawler</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-nightcrawler.png" alt="">
    </li>
    <li id="professor-x" data-teams="original">
      <h2>Professor X</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-professor-x.png" alt="">
    </li>
    <li id="psylocke" data-teams="force hellfire">
      <h2>Psylocke</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-psylocke.png" alt="">
    </li>
    <li id="quicksilver" data-teams="factor">
      <h2>Quicksilver</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-quicksilver.png" alt="">
    </li>
    <li id="rictor" data-teams="force factor">
      <h2>Rictor</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-rictor.png" alt="">
    </li>
    <li id="storm" data-teams="force hellfire">
      <h2>Storm</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-storm.png" alt="">
    </li>
    <li id="sunspot" data-teams="force hellfire">
      <h2>Sunspot</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-sunspot.png" alt="">
    </li>
    <li id="tithe" data-teams="hellfire">
      <h2>Tithe</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-tithe.png" alt="">
    </li>
    <li id="wolverine" data-teams="force">
      <h2>Wolverine</h2>
      <img src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/77020/ct-wolverine.png" alt="">
    </li>
  </ul>
</div>

css:

@import url(http://fonts.googleapis.com/css?family=Oswald:400,700);

*, *::before, *::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  color: #fff;
  background-color: #444;
  font: 0/1 'Oswald', sans-serif;
  text-align: center;
}

.container {
  margin: 0 auto;
  padding: 20px 5px;
}

h1 {
  margin: 0 5px 15px;
  font-size: 45px;
  line-height: 1.2;
  text-transform: uppercase;
  text-shadow: 0 0 20px #000;
}

.marvel {
  color: #c00;
}

input {
  position: absolute;
  left: -9999px;
}

label {
  display: inline-block;
  position: relative;
  width: 150px;
  margin: 5px;
  padding: 10px 10px 10px 38px;
  background-color: #000;
  font-size: 18px;
  white-space: nowrap;
  cursor: pointer;
}

label::before {
  content: '';
  display: block;
  position: absolute;
  top: 10px;
  left: 10px;
  width: 18px;
  height: 18px;
  border: 2px solid #fff;
  background-color: #444;
}

input:focus + label {
  outline: 1px solid #fff;
}

input:checked + label::before {
  background-color: #c00;
}

/* width is controlled here and in media queries to ensure that there's always an even number of columns (prevents orphans) */
.characters {
  display: inline-block;
  width: 240px;
  list-style-type: none;
  margin: 15px -5px 0;
  padding: 0;
}

.characters li {
  display: inline-block;
  position: relative;
  margin: 10px;
  padding: 0;
  width: 100px;
  height: 100px;
}

.characters h2 {
  display: none;
  position: absolute;
  z-index: 2;
  top: 70px;
  left: 0px;
  right: 0px;
  margin: 0;
  padding: 4px;
  border: 1px solid #fff;
  color: #fff;
  background-color: #000;
  font-size: 14px;
  font-weight: normal;
}

.characters img {
  width: 100px;
  height: 100px;
  border: 4px solid #fff;
  border-radius: 50%;
  background-color: #fff;
  box-shadow: 0 0 20px #000;
  opacity: .25;
  transform: scale(.5);
}

#original:checked ~ .characters [data-teams~="original"] h2,
#force:checked ~ .characters [data-teams~="force"] h2,
#factor:checked ~ .characters [data-teams~="factor"] h2,
#hellfire:checked ~ .characters [data-teams~="hellfire"] h2 {
  display: block;
}

#original:checked ~ .characters [data-teams~="original"] img,
#force:checked ~ .characters [data-teams~="force"] img,
#factor:checked ~ .characters [data-teams~="factor"] img,
#hellfire:checked ~ .characters [data-teams~="hellfire"] img {
  animation: avatar .3s forwards;
}

/* fade in and expand image to full size, with a slight bounce */
@keyframes avatar {
  70% {
    opacity: 1;
    transform: scale(1.1);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

@media only screen and (min-width: 500px) {
  .characters {
    width: 480px;
  }
}

@media only screen and (min-width: 740px) {
  .characters {
    width: 720px;
  }

  h1 {
    font-size: 60px;
  }
}

@media only screen and (min-width: 980px) {
  .characters {
    width: 960px;
  }
}


```



