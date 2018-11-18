---
layout: post
title: 前端日常恢复计划-轮播图
category: 技术
tags: [学习,开发,前端,总结,2018,原生js开发,Source Code,Node]
keywords: 总结,2018,Node开发,source code,前端
description: 
---

## 前言
前段时间有点忙于学业，这段时间打算先捡起一部分熟悉一下。
因为之前申请实习准备了不少js的笔试题，打算借此也将其给系统的总结一遍。

## 轮播图

轮播图作为前端最基础的展示组件也有很多种实现方式。但是基本原理是不变的。显示单一图片或图文组合，通过手动或者自动切换。


## Css轮播

用纯css实现轮播图的写法是出自[You-Dont-Need-JavaScript仓库](https://github.com/you-dont-need/You-Dont-Need-JavaScript)

[在线Demo](https://jsfiddle.net/mu3udxmm/)

![imgn](http://img.haoqiao.me/2017-carousel.gif)

它的具体原理：

```

1.CSS3 element+element 选择器（相邻兄弟选择器），element+element 选择器用于选取第一个指定的元素之后（不是内部）紧跟的元素。

2.CSS3 element1~element2 选择器，element1~element2 选择 element1 之后出现的所有 element2。两种元素必须拥有相同的父元素，但是 element2 不必直接紧随 element1。

3.CSS3 :checked 选择器，:checked 选择器匹配每个选中的输入元素（仅适用于单选按钮或复选框）。

4.HTML5 label 标签的 for 属性。

```

我们来看下HTML结构


```
<div class="carousel"> <-- 轮播图组件容器 -->
    <div class="carousel-inner">
        // 默认选择第一项显示，checked="checked" 如果去掉这个，将显示空白 设置hidden="" 将input隐藏
        //radio 放在每一个 carousel-item 之前，这样就可以用 + 选择器来匹配
                
        <input class="carousel-open" type="radio" id="carousel-1" name="carousel" aria-hidden="true" hidden="" checked="checked">
        <-- 为了避免屏幕识读设备抓取非故意的和可能产生混淆的输出内容（尤其是当图标纯粹作为装饰用途时）,为这些图标设置了 aria-hidden="true" 属性 -->
        
        //carousel-item现在只包含一个图片
        <div class="carousel-item">
            
            <img src="http://fakeimg.pl/2000x800/0079D8/fff/?text=Without">
        </div>
        // 
        <input class="carousel-open" type="radio" id="carousel-2" name="carousel" aria-hidden="true" hidden="">
        <div class="carousel-item">
            <img src="http://fakeimg.pl/2000x800/DA5930/fff/?text=JavaScript">
             
        </div>
        <input class="carousel-open" type="radio" id="carousel-3" name="carousel" aria-hidden="true" hidden="">
        <div class="carousel-item">
            <img src="http://fakeimg.pl/2000x800/F90/fff/?text=Carousel">
        </div>
        // for 属性指向对应radio id
        //每个 radio 指定了一组前后的按钮，同时用 css 来排布，每次显示当前显示图片前后图片的 input radio 对应的 label
        <label for="carousel-3" class="carousel-control prev control-1">‹</label>
        <label for="carousel-2" class="carousel-control next control-1">›</label>
        <label for="carousel-1" class="carousel-control prev control-2">‹</label>
        <label for="carousel-3" class="carousel-control next control-2">›</label>
        <label for="carousel-2" class="carousel-control prev control-3">‹</label>
        <label for="carousel-1" class="carousel-control next control-3">›</label>
        <ol class="carousel-indicators">
            <li>
                <label for="carousel-1" class="carousel-bullet">•</label>
            </li>
            <li>
                <label for="carousel-2" class="carousel-bullet">•</label>
            </li>
            <li>
                <label for="carousel-3" class="carousel-bullet">•</label>
            </li>
        </ol>
    </div>
</div>


```

然后来看具体css实现：

```
//设置一个背景图
html, body {
    margin: 0px;
    padding: 0px;
    background: url("http://digital.bnint.com/filelib/s9/photos/white_wood_4500x3000_lo_res.jpg");
}

// 设置布局方式
.carousel {
    position: relative;
    box-shadow: 0px 1px 6px rgba(0, 0, 0, 0.64);
    margin-top: 26px;
}

.carousel-inner {
    position: relative;
    overflow: hidden;
    width: 100%;
}

//设置checked状态时 取消透明
.carousel-open:checked + .carousel-item {
    position: static;
    opacity: 100;
}
// 默认状态绝对定位，item透明 过渡效果为渐入渐出
.carousel-item {
    position: absolute;
    opacity: 0;
    -webkit-transition: opacity 0.6s ease-out;
    transition: opacity 0.6s ease-out;
}

.carousel-item img {
    display: block;
    height: auto;
    max-width: 100%;
}

// Label两边的< > 默认隐藏
.carousel-control {
    background: rgba(0, 0, 0, 0.28);
    border-radius: 50%;
    color: #fff;
    cursor: pointer;
    display: none;
    font-size: 40px;
    height: 40px;
    line-height: 35px;
    position: absolute;
    top: 50%;
    -webkit-transform: translate(0, -50%);
    cursor: pointer;
    -ms-transform: translate(0, -50%);
    transform: translate(0, -50%);
    text-align: center;
    width: 40px;
    z-index: 10;
}

//确定位置
.carousel-control.prev {
    left: 2%;
}

.carousel-control.next {
    right: 2%;
}

.carousel-control:hover {
    background: rgba(0, 0, 0, 0.8);
    color: #aaaaaa;
}
// checked选中后 显示
#carousel-1:checked ~ .control-1,
#carousel-2:checked ~ .control-2,
#carousel-3:checked ~ .control-3 {
    display: block;
}

.carousel-indicators {
    list-style: none;
    margin: 0;
    padding: 0;
    position: absolute;
    bottom: 2%;
    left: 0;
    right: 0;
    text-align: center;
    z-index: 10;
}

.carousel-indicators li {
    display: inline-block;
    margin: 0 5px;
}

.carousel-bullet {
    color: #fff;
    cursor: pointer;
    display: block;
    font-size: 35px;
}

.carousel-bullet:hover {
    color: #aaaaaa;
}

#carousel-1:checked ~ .control-1 ~ .carousel-indicators li:nth-child(1) .carousel-bullet,
#carousel-2:checked ~ .control-2 ~ .carousel-indicators li:nth-child(2) .carousel-bullet,
#carousel-3:checked ~ .control-3 ~ .carousel-indicators li:nth-child(3) .carousel-bullet {
    color: #428bca;
}

#title {
    width: 100%;
    position: absolute;
    padding: 0px;
    margin: 0px auto;
    text-align: center;
    font-size: 27px;
    color: rgba(255, 255, 255, 1);
    font-family: 'Open Sans', sans-serif;
    z-index: 9999;
    text-shadow: 0px 1px 2px rgba(0, 0, 0, 0.33), -1px 0px 2px rgba(255, 255, 255, 0);
}

```

这个css轮播图方案其实最主要的还是`checkd:checked`属性与css选择器结合产生的联动效果。结合css3过渡属性`transition`生成动画效果。


## Js轮播
轮播图关于js方案其实有很多种，但大多都是基于隐藏其余图，展示图切换的原理增加。

因此，关于轮播图我们需要考虑如何将其做到比较精致的地步，比如该如何定义一个轮播组件，给用户使用时需要提供哪些对外API和属性。

凭空想总会有一些误差，考虑到当年第一次接触轮播图还是`bootstrap`。因此我们可以考虑借鉴。

```
bootstrap的carousel接受一个dom元素作为初始化对象

同时它可以设置以下参数：
interval 自动轮播间隔
direction 从左到右还是从右到左
pause   如果设置为“悬停”，请暂停鼠标滚轮上的旋转木马循环，并在鼠标滚轮上恢复轮播的循环。
如果设置为null，则在盘旋盘上悬停不会暂停。
keyboard 是否接受键盘事件

而且它可以作为触发器进行事件触发，比如：

.carousel('prev')

Cycles to the previous item.

.carousel('next')

Cycles to the next item.

.carousel('pause')

Stops the carousel from cycling through items.



```

我们可以先来看一份pure js的轮播图

[在线demo](https://jsfiddle.net/1pbmrzap/)

![imgn](http://img.haoqiao.me/2017-carousel2.gif)

它的html结构也类似于之前的。但是减少了一些重复的部分。
主要通过动态增减`active类名`隐藏显示item配合`@keyframes动画`以及达到轮播效果。

不过它没有自动播放等功能。仅仅是一个基础的轮播图。

我们简要的来看看它的js代码：

```
// 获取轮播的容器元素
var items = document.querySelectorAll('.carousel .item');
var dots = document.querySelectorAll('.carousel-indicators li');
// 默认当前展示图
var currentItem = 0;
var isEnabled = true;

//更改图的坐标,而且在限定范围内滚动，不会越界
function changeCurrentItem(n) {
	currentItem = (n + items.length) % items.length;
}

function nextItem(n) {
	hideItem('to-left');
	changeCurrentItem(n + 1);
	showItem('from-right');
}

function previousItem(n) {
	hideItem('to-right');
	changeCurrentItem(n - 1);
	showItem('from-left');
}

function goToItem(n) {
	if (n < currentItem) {
		hideItem('to-right');
		currentItem = n;
		showItem('from-left');
	} else {
		hideItem('to-left');
		currentItem = n;
		showItem('from-right');
	}
}

// 隐藏轮播
function hideItem(direction) {
	isEnabled = false;
	items[currentItem].classList.add(direction);
	dots[currentItem].classList.remove('active');
	items[currentItem].addEventListener('animationend', function() {
		this.classList.remove('active', direction);
	});
}

//显示轮播
function showItem(direction) {
	items[currentItem].classList.add('next', direction);
	dots[currentItem].classList.add('active');
	items[currentItem].addEventListener('animationend', function() {
		this.classList.remove('next', direction);
		this.classList.add('active');
		isEnabled = true;
	});
}

document.querySelector('.carousel-control.left').addEventListener('click', function() {
	if (isEnabled) {
		previousItem(currentItem);
	}
});

document.querySelector('.carousel-control.right').addEventListener('click', function() {
	if (isEnabled) {
		nextItem(currentItem);
	}
});

// 给小圆点绑定事件
document.querySelector('.carousel-indicators').addEventListener('click', function(e) {
	var target = [].slice.call(e.target.parentNode.children).indexOf(e.target);
	if (target !== currentItem && target < dots.length) {
		goToItem(target);
	}
});

```

我们可以再来看一种版本的轮播图

[在线demo](https://codepen.io/forbesg/pen/akGaqW)

![imgn](http://img.haoqiao.me/2017-carousel3.gif)

这个就是典型的自动播放型。我们可以来看下具体代码

它的自动播放代码很简单

```

		if (e.type !== 'autoClick') {
			clearInterval(scrollInterval);
			scrollInterval = setInterval(autoScroll, interval);
		}

```

通过定时器来执行播放的函数。

而且如果想要玩点新花样，比如`3d轮播旋转`，也非常简单。

[在线demo](https://codepen.io/zomgdomo/pen/MjNJPN)

![imgn](http://img.haoqiao.me/2017-carousel4.gif)

可以仔细看下它的代码:

```
html:
<base href="https://s3-us-west-2.amazonaws.com/s.cdpn.io/4273/wanaka-tree.jpg">
<div id="carousel">
  <section id="spinner">
    <img src="http://wallpapercave.com/wp/xO2qTSS.jpg" alt>
    <img src="https://s-media-cache-ak0.pinimg.com/originals/92/d4/40/92d44024c01f9fe0c19108678e3d1358.jpg" alt>
    <img src="https://lh3.googleusercontent.com/04xkqigdeXDOPtAIUtwqYoJhC-6xWIGPdZHPo_2xxJQqJ8k_zyFPIFNBqgiDQwiKZGo=h900" alt>
    <img src="https://pbs.twimg.com/media/CiLJCfqW0AE3crt.jpg" alt>
    <img src="http://hansgutknecht.com/blog/wp-content/uploads/2009/06/dn06-clouds1hg.jpg" alt>
    <img src="http://wallpapercave.com/wp/VpzSG78.jpg" alt>
    <img src="http://farm6.static.flickr.com/5024/5547942721_34fef447a1.jpg" alt>
    <img src="http://wallpapercave.com/wp/QFvJFwE.jpg" alt>
  </section>
</div>
<span style="float:left" class="ss-icon" onclick="galleryspin('-')">&lt;</span>
<span style="float:right" class="ss-icon" onclick="galleryspin('')">&gt;</span>

```

```
css:

div#carousel { 
  perspective: 1200px; 
  background: #040c19; 
  padding-top: 10%; 
  font-size:0; 
  margin-bottom: 3rem; 
  overflow: hidden; 
}
section#spinner { 
  transform-style: preserve-3d; 
  height: 300px; 
  transform-origin: 50% 50% -500px; 
  transition: 1s; 
} 
section#spinner img { 
  width: 40%; max-width: 425px; 
  position: absolute; left: 30%;
  transform-origin: 50% 50% -500px;
  outline:1px solid transparent; 
}
section#spinner img:nth-child(1) { transform:rotateY(0deg); 
}
section#spinner img:nth-child(2) { transform: rotateY(-45deg); }
section#spinner img:nth-child(3) { transform: rotateY(-90deg); }
section#spinner img:nth-child(4) { transform: rotateY(-135deg); }
section#spinner img:nth-child(5){ transform: rotateY(-180deg); }
section#spinner img:nth-child(6){ transform: rotateY(-225deg); }
section#spinner img:nth-child(7){ transform: rotateY(-270deg); }
section#spinner img:nth-child(8){ transform: rotateY(-315deg); }
div#carousel ~ span { 
  color: #fff; 
  margin: 5%; 
  display: inline-block; 
  text-decoration: none; 
  font-size: 4rem; 
  transition: 0.6s color; 
  position: relative; 
  margin-top: -6rem; 
  border-bottom: none; 
  line-height: 0; }
div#carousel ~ span:hover { color: #888; cursor: pointer; }

```

```
js:

var angle = 0;
function galleryspin(click) { 
spinner = document.querySelector("#spinner");
if (!click) { angle = angle + 45; } else { angle = angle - 45; }
spinner.setAttribute("style","-webkit-transform: rotateY("+ angle +"deg); -moz-transform: rotateY("+ angle +"deg); transform: rotateY("+ angle +"deg);");
}

```

其关键代码就是css的`transform`，将图片计算角度变化。
然后点击的事件是将整个容器进行旋转。这样不需要计算内部的旋转度数。但是效果看起来是每个图片都在旋转。



## 总结

轮播图现在大家基本考虑到实用性不会自己写，因为浏览器兼容是一块，移动端兼容又是一块。`swipe` `slider` `unslider`等轮播插件是大家主力使用的。
当然自己写个简单的根据以上内容也不难捣鼓出。关于轮播最主要的html结构和css样式的结合。关于js代码其实也只是起到一个辅助的作用。比如参数的传递，动画效果的切换，内容的隐藏。以及炫酷效果实现。



