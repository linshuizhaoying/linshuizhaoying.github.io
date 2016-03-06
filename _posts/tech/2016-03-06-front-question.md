---
layout: post
title: 前端面试题错题集(一)
tags: [学习,私货,前端,面试]
keywords: Javascript,代码,CSS3,面试题,前端,学习总结
description: 
---

## 前言
很久没写博客了，因为最近零碎的事情做得比较多，静下心来学习思考的时间并不是很连贯。今天终于抽出一点时间来整理部分最近整理的前端面试题的错题集。

## 正文

首先是Html/Css3 与其他基础部分。

```
1.css属性overflow属性定义溢出元素内容区的内容会如何处理。如果值为 scroll，不论是否需要，用户代理都会提供一种滚动机制。

这句话是对的还是错的？

答案是对的。但是为什么呢？通过查w3c规范和mdn文档，可以得到以下资料：

/* 默认值。内容不会被修剪，会呈现在元素框之外 */
overflow: visible;

/* 内容会被修剪，并且其余内容不可见 */
overflow: hidden;

/* 内容会被修剪，浏览器会显示滚动条以便查看其余内容 */
overflow: scroll;

/* 由浏览器定夺，如果内容被修剪，就会显示滚动条 */
overflow: auto;

/* 规定从父元素继承overflow属性的值 */
overflow: inherit;

visible
默认值。内容不会被修剪，会呈现在元素框之外。默认值。内容不会被修剪，会呈现在元素框之外
hidden
内容会被修剪，并且其余内容是不可见的。
scroll
内容会被修剪，并且浏览器会使用滚动条，无论内容什么内容被裁减。这避免了在动态环境中滚动条的出现和消失问题。打印机会打印溢出的内容。
auto
取决于用户代理。浏览器，例如火狐，会在内容溢出时提供滚动条。

```

```
2.下述有关css属性position的属性值的描述，说法错误的是？

A.static：没有定位，元素出现在正常的流中
B.fixed：生成绝对定位的元素，相对于父元素进行定位
C.relative：生成相对定位的元素，相对于元素本身正常位置进行定位。
D.absolute：生成绝对定位的元素，相对于 static 定位以外的第一个祖先元素进行定位。

我一开始选的A，事实上好久没接触这些概念性的东西，一下子看到还有点蒙。于是妥妥的错了。
答案是B

查了下资料:
static
这个关键字使得这个元素使用正常的表现，即元素处在文档流中它当前的布局位置，top, right, bottom, left 和 z-index 属性无效。
relative
使用这个关键字来布局元素就好像这个元素没有被设置过定位一样。即会适应该元素的位置，并不改变布局（这样会在此元素原本所在的位置留下空白）。position:relative对table-*-group, table-row, table-column, table-cell, table-caption无效。
absolute
不为元素预留空间，元素位置通过指定其与它最近的非static定位的祖先元素的偏移来确定。绝对定位的元素可以设置外边距（margins），并且不会与其他边距合并。
fixed
不为元素预留空间。通过指定相对于屏幕视窗的位置来指定元素的空间，并且该元素的位置在屏幕滚动时不会发生改变。打印时元素会出现在的每页的固定位置。fixed属性通常会创建新的栈环境。

想了解更详细的可以看MDN：https://developer.mozilla.org/zh-CN/docs/Web/CSS/position

```

```
3.以下是行内元素的是？

A.span
B.input
C.ul
D.p

我好像选的是AD ,正确答案是AB

很奇怪，当时看题的时候对行内元素居然不甚了解。。。后来还是看了资料才回想起。。。

大多数 HTML 元素被定义为块级元素或内联元素。“块级元素”译为 block level element，“内联元素”译为 inline element。
1.块级元素 在浏览器显示时，通常会以新行来开始（和结束）。块级元素按照其应用于结构还是内容分为三种：结构化块状元素，终端块状元素，多目标块状元素。 
结构化块状元素： 这类元素用于构造文档的结构，没有语义上的含义，仅仅划分出了文档的组织方式，并没有体现文档的内容。 
终端块状元素： 这类元素用于从结构转向内容，拥有语义上的含义，能够表明内容的性质。终端块状元素属于结构的终点，它们不能再包含其他块级元素，只能包含文本或行级元素。
多目标块状元素： 多目标指的是可以自由的扩展或嵌套文档的结构，以可以终端的形式出现。当多目标块状元素以结构化的方式使用时就含有结构化的内涵，以终端的形式使用就含有语义的内涵。 
2.内联元素 (inline element)或称为行内元素 一般都是基于语义级(semantic)的基本元素，只能容纳文本或者其它内联元素。 

上述选项标签<ul></ul>、<p></p>在默认情况下会独占一行，属于块级元素；标签<span></span>、<input type="XXX"/>在默认情况下和相邻的行内元素排在同一行内，属于行内元素。

判断行内元素和块级元素的快捷方法就是判断是否能并列

```

```
4.HTML的Doctype作用? 严格模式与混杂模式如何区分？它们有何意义?
这一题我大概知道一些。不过我想知道更加提炼的答案我就扔到错题集了。

1.<!DOCTYPE> 声明位于文档中的最前面，处于 <html> 标签之前。告知浏览器的解析器，用什么文档类型 规范来解析这个文档。
2.严格模式的排版和 JS 运作模式是 以该浏览器支持的最高标准运行。在混杂模式中，页面以宽松的向后兼容的方式显示。模拟老式浏览器的行为以防止站点无法工作。
3.DOCTYPE不存在或格式不正确会导致文档以混杂模式呈现。

```

```
5.简述document.write和 innerHTML的区别。
这个题目我一开始看到真的有点懵，不知道它考察的是哪点。。
后来看了资料。。。
document.write只能重绘整个页面,
innerHTML可以重绘页面的一部分。

document.write是直接写入到页面的内容流，如果在写之前没有调用document.open, 浏览器会自动调用open。每次写完关闭之后重新调用该函数，会导致页面被重写。
innerHTML则是DOM页面元素的一个属性，代表该元素的html内容。你可以精确到某一个具体的元素来进行更改。如果想修改document的内容，则需要修改document.documentElement.innerElement。

innerHTML很多情况下都优于document.write，其原因在于其允许更精确的控制要刷新页面的那一个部分。


```

![imgn](http://haoqiao.qiniudn.com/cssjianjiao1.png)

```
6.css尖角的实现
请用CSS实现如上图的样式，相关尺寸如图示，其中dom结构为：
<div id=”demo”></div>

我一开始想的思路就是利用伪元素。不过我忘了具体写法。先记下两种写法：

方法1：
#demo {
    width: 100px;
    height: 100px;
    background-color: #fff;
    position: relative;
    border: 2px solid #333;
}
 
#demo:after, #demo:before {
    border: solid transparent;
    content: ' ';
    height: 0;
    left: 100%;
    position: absolute;
    width: 0;
}
 
#demo:after {
    border-width: 10px;
    border-left-color: #fff;
    top: 20px;
}
 
#demo:before {
    border-width: 12px;
    border-left-color: #000;
    top: 18px;
}



方法2：
#demo {
    width: 100px;
    height: 100px;
    border: 2px solid #000;
    position: relative;
}
#demo:after {
    content: '';
    display: block;
    width: 14.1421px;
    height: 14.1421px;
    border-top: 2px solid #000;
    border-right: 2px solid #000;
    position: absolute;
    right: -10px;
    top: 20px;
    transform: rotate(45deg);
    background-color: #fff;
}

```

## 结尾
这仅仅只是一部分面试题的错题集，但是感觉自己大概就是对一些基础的概念记忆不是很深刻，如果是做东西我相信给我时间我都能完成，但是对于理论更深刻的掌握会让人在各个项目中游刃有余。因此未来还需要多找到自己的知识薄弱点。。。


