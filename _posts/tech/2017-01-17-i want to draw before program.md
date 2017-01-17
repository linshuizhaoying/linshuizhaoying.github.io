---
layout: post
title: 在编码之前先画个图
tags: [程序流图,类图,前端,Graphviz,]
keywords: web,代码,前端
description: 
---

## 正文之前
逛博客的时候发现一个工具`Graphviz`,因为看到文中作者提到了用`Graphviz`画一个图，对此忽然联想到我们程序员开发之前画图大概会有几种方法。为此本文为此做出一些探讨。

## 正文

我们先罗列下我们需要用到各种各样图的场景

```

1.UML的各种图
2.程序流程图
3.数据展示图
4.思维导图

```
画图目的是为了更直观的展示，在描述复杂的系统/需求/对象关系/思路的时候，有时候一图胜过千言万语，比如我本人就常常用思维导图来整理一些点子，一些项目结构，一些读书笔记。但是画其它的图思维导图就不是很适合了。
下面我们来一一寻找适合以上需求的软件。

## Graphviz

### 下载
先去[官网](http://www.graphviz.org/)下载适合自己系统版本的，这里我down了mac版。

### 使用方式

这里我们主要研究生成层级图的`dot`。
官网文档在[这里](http://www.graphviz.org/content/dot-language)
一般是执行`dot`命令来执行`.dot`文档来生成图，但这样太慢了。
因此更好的方法直接把`.dot`文档拖到`Graphviz`程序上，但有时候这样也觉得太慢。
这时候你可以考虑用`sublime`的插件`GraphvizPreview`
只要你全选你的代码就能然后按快捷键就能快速显示图。如图:
![imgn](http://haoqiao.qiniudn.com/Graphviz1-1.png)


graphviz 有两种图，一种是无向图graph，边用--连接，一种是有向图digraph，边用->连接。当然你可以通过代码设置将边框设置为你想要的，还有虚线，子图等等功能，下面我们看网上摘抄的一段代码

```
digraph G{

	size = "4, 4";//图片大小
	main[shape=box];/*形状*/

	main->parse;
	parse->execute;

	main->init[style = dotted];//虚线

	main->cleanup;

	execute->{make_string; printf}//连接两个

	init->make_string;

	edge[color = red]; // 连接线的颜色

	main->printf[style=bold, label="100 times"];//线的 label

	make_string[label = "make a\nstring"]// \n, 这个node的label，注意和上一行的区别

	node[shape = box, style = filled, color = ".7.3 1.0"];//一个node的属性

	execute->compare;
}
```

![imgn](http://haoqiao.qiniudn.com/Graphviz1-2.png)

喜欢更深入学习的童鞋可以戳[这里](http://icodeit.org/2015/11/using-graphviz-drawing/) 这篇博文是我搜到一篇讲解比较详细的。

## OmniGraffle && Sketch && Keynote

### OmniGraffle

`Omini`家族的软件都是非常有特色，产品概念性非常完整，上节我们用`Graphviz`来用脚本生成图片，但是习惯了拖来拖去生成图片的我们如果想边思考边绘制原型或者流程图，那么`OmniGraffle`就是非常好的选择。

![imgn](http://haoqiao.qiniudn.com/OmniGraffle1-1.png)

完全随心所欲的画图，而且根据绘制的类型不同可以自己调整风格。更方便的是你还可以去下载它的模板`stencil`，来达到快速画图的目的。
![imgn](http://haoqiao.qiniudn.com/OmniGraffle1-2.png)
这里提供一些`stencil`的网站
```
1.https://stenciltown.omnigroup.com
2.https://www.graffletopia.com/ （收费）
```
`OmniGraffle1`的优势我觉得是对于不怎么会画图的程序员，如果看到一些比较好的风格，可以通过直接编辑属性达到快速模仿的目的。

### sketch

同样的软件还有设计师神器`sketch`,但是个人觉得画程序图不怎么好使，虽然也能画，但是更偏向于设计向。

![imgn](http://haoqiao.qiniudn.com/sketch1-1.png)

当然`sketch`的模板更加的多，主要是设计师们比较给力。而且还有很多丰富的插件，具体可以点[这里](https://www.zhihu.com/question/27495264)

### keynote

在还没有`OmniGraffle`的时候，如果遇到需要简便画图的情况下，大多数人可能会选择`画图工具`或者`Ps`
但是我当时选择了`keynote` 感觉在没有工具的情况下，效果还行。

![imgn](http://haoqiao.qiniudn.com/keynote1-1.png)

同理，可以替代的是`PPT`和`WPS`。

## echarts

现在数据表示最强大的图表制作软件之一就是`echarts`,直接去[官网](http://echarts.baidu.com/)
![imgn](http://haoqiao.qiniudn.com/9AB1D834-0F14-40BD-B10B-1EFBB317E319.png)

这个文档这里就不给出了，直接官网查询并使用。
对于数据展示我现阶段还没很深入的学习因此这里就不展开说明。

## D3.JS

前端数据图形化展示鼻祖[D3.js](https://github.com/d3/d3/wiki/Gallery)
对于这个我只能说只有你想不到，没有它做不到~
有兴趣的可以深入去学习。

## 思维导图
 
关于思维导图我推荐两个，一个是`ithoughtsX` ，我一直用它来复习学习以及制作读书笔记。
![imgn](http://haoqiao.qiniudn.com/ithoughtsX.png)
win下也有类似的软件需要大家自己寻找。

一个是在线版的`百度脑图`。
![imgn](http://haoqiao.qiniudn.com/baidunaotu.png)

# 总结

就我个人而言，我还是更加推荐`OmniGraffle`，当然大家看各自喜好选择。我觉得在开发之前，先画图梳理一下思路，然后再配合详细的文档说明，在开发中把百分50的时间用在设计思考上，我们在编码的时候才会事半功倍。

以上一些资料我放到了Github上的[瞎折腾](https://github.com/linshuizhaoying/toss)

