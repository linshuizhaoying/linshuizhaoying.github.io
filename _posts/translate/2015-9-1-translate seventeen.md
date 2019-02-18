---
layout: post
title: CSS Shapes 101 (CSS 形状）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://alistapart.com/article/css-shapes-101)

### 作者: Sara Soueidan


### 译者: 临水照影


## 正文之前

月初果然还是得用sara女神的文章来震住整个月的学习动力。由于sara女神的文章一向很长。。。于是我会精简掉=-=反

## 正文

### 创建css形状

你可以用shaps属性来为一个元素创建形状。你可以通过shape函数来定义属性的值。

![img1](http://img.haoqiao.me//shape-rule.png)

你可以用一下的函数
    
    circle()
    ellipse()
    inset()
    polygon()

每个形状都是通过一系列点集合来定义的。有些函数将点集合作为参数，有些拿来偏移。但它们最后画出来的形状都是用点来表示。接下来我们将介绍每个函数。

形状属性接受下列值：
    
    shape-outside：包裹内容周围的形状
    shape-inside：包裹形状里面的内容
    
为一个元素申明一个形状是很简单的：
    
    .element {
	  shape-outside: circle(); /* content will flow around the circle defined on the element */
    }

或者

    .element {
	  shape-outside: url(path/to/image-with-shape.png);
    }

但是。。。

如果你直接应用这些到你的元素中，形状并不会显示除非：
    
    1.这个元素是浮动的。未来的css形状可能允许我们应用到非浮动的元素上。但现在不行。
    2.该元素必须有尺寸。该元素的高度和宽度将作为整个元素的坐标系。
    
正如你上面所见，所有形状都是由点定义的。因为这些点有坐标，浏览器想定位元素上的点必须知道该坐标系。所以，上面例子想要正常运行需要像这样：
    
    .element {
    	float: left;
	    height: 10em;
    	width: 15em;
    	shape-outside: circle();
    }

因为每一个形状都是由一些点定义来表明坐标，因此如果改变坐标系中的点将直接改变形状。举个例子，下面图中用`polygon()`来创建多边形。这个形状有六个点，改变坐标系中的点将改变形状。而且将会影响内部浮动的内容。

![img2](http://img.haoqiao.me//shape-points.png)

### 用形状函数来定义的形状

我们将开始通过围绕一个圆形的头像和一些信息来演示。

![img3](http://img.haoqiao.me//demo-user-profile-screenshot.png)


我们用`circle()`函数来让图片应用圆形，用下面的html：
    
    <img src="http://api.randomuser.me/0.3.2/portraits/men/7.jpg" alt="profile image" />

    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Harum itaque nam blanditiis eveniet enim eligendi quae adipisci?</p>

    <p>Assumenda blanditiis voluptas tempore porro quibusdam beatae deleniti quod asperiores sapiente dolorem error! Quo nam quasi soluta reprehenderit laudantium optio ipsam ducimus consequatur enim fuga quibusdam mollitia nesciunt modi.</p>

你可能会问，为什么我们不用`boder-radius`。这是因为`boder-radius`对内容周边浮动没有影响。它只影响元素本身边界。而且元素本身还是被视为一个矩形。如果我们对上图的例子中用下面css:
   
    img {
    	float: left;
    	width: 150px;
    	height: 150px;
    	border-radius: 50%;
    	margin-right: 15px;
    }

![img4](http://img.haoqiao.me//demo-user-profile-screenshot-incomplete.png)

对比一下你就能发现差别。

为了改变内容的浮动以适应特定的形状，用形状属性：

    img {
    	float: left;
    	width: 150px;
	    height: 150px;
	    border-radius: 50%;

	    shape-outside: circle();
	    shape-margin: 15px;
    }

这样你可以看到上面的圆形环绕的显示效果，如果在不支持css形状的浏览器，将显示上面无环绕的效果。

[Click Demo Here](http://jsfiddle.net/linshuizhaoying/7fmeoev1/)

你可以像这样用circle（），这是官方的语法:
    
    circle() = circle( [<shape-radius>]? [at <position>]? )

当你省略参数，它会默认定义到你元素的中心。

你可以用任意长度单位来指定半径(px,em,pt,etc.)。也可以用`closest-side`或者`furthest-side`来指定半径，不过
`closest-side`是默认的。它意味着浏览器将用元素离中心点最近的长度作为半径。`furthest-side`是元素离中心最长的长度作为半径。
    
    shape-outside: circle(farthest-side at 25% 25%); /* defines a circle whose radius is half the length of the longest side, positioned at the point of coordinates 25% 25% on the element’s coordinate system*/

    shape-inside: circle(250px at 500px 300px); /* defines a circle whose center is positioned at 500px horizontally and 300px vertically, with a radius of 250px */

![img5](http://img.haoqiao.me//closest-side-farthest-side.png)

` ellipse()`函数和`circle()`函数类似。接受同样的值，不过它不是用一个半径作为参数，而是两个，一个x轴方向，一个y轴方向。

    ellipse() = ellipse( [<shape-radius>{2}]? [at <position>]? )

当不是直接用` ellipse()`函数和`circle()`函数，`insert()`用来创建多边形。它可以帮助我们创建圆角的内容浮动在圆角边的元素。


![img6](http://img.haoqiao.me//inset-example.png)

`insert()`还能接受四个偏移值。用于指定从引用框的边缘向内偏移。它还能通过round关键词接受四个圆角的值。

    inset() = inset( offset{1,4} [round <border-radius>]? )
    
下面将为一个浮动元素创建一个圆角边框：
    
    .element {
	    float: left;
	    width: 250px;
	    height: 150px;
	    shape-outside: inset(0px round 100px) border-box;
    }
    
最后一个形状函数是polygon()，它很复杂，可以接受任意数量的点的坐标。

来看个例子：

![img7](http://img.haoqiao.me//polygon-example-incomplete.png)

然后添加css：
    
    img.right {
    	float: right;
	    height: 100vh;
	    width: calc(100vh + 100vh/4);
	    shape-outside: polygon(40% 0, 100% 0, 100% 100%, 40% 100%, 45% 60%, 45% 40%);
    }

你可以用任意长度单位，我这里用百分比。

为了显示我们创建的多边形，我们需要用路径裁剪。

    img.right {
    	float: right;
    	height: 100vh;
    	width: calc(100vh + 100vh/4);
    	shape-outside: polygon(40% 0, 100% 0, 100% 100%, 40% 100%, 45% 60%, 45% 40%);
	    /* clip the image to the defined shape */
	    clip-path: polygon(40% 0, 100% 0, 100% 100%, 40% 100%, 45% 60%, 45% 40%);
    }

然后结果如下：

![img8](http://img.haoqiao.me//polygon-example-finished.png)

[Demo click here](http://jsfiddle.net/linshuizhaoying/7fmeoev1/1/)

`polygon() `函数也有一个可选的填充关键字。更多详情[点这里](http://alistapart.com/article/css-shapes-101)

### 用图片定义形状

这个图片必须有个透明层能够让浏览器提取形状。

举个例子：

![img9](http://img.haoqiao.me//leaf.png)

用`shape-outside`属性和`url()`来获取图片的点，我们可以用这个叶子环绕内容。

    .leaf-shaped-element {
    	float: left;
    	width: 400px;
    	height: 400px;
    	shape-outside: url(leaf.png);
	    shape-margin: 15px;
    	shape-image-threshold: 0.5;
	    background: #009966 url(path/to/background-image.jpg);
	    mask-image: url(leaf.png);
    }

当然如果你定义了一个背景，它将被裁剪成定义的形状。结果如下：

![img11](http://img.haoqiao.me//shape-image-example.png)

如果你像创建负责的图形，你可用ps中透明图层图像来代替。

### css Shape的响应式设计

`shape-outside`允许你用任意的长度单位。它是响应式的。
`shape-inside `现在并不是响应式的。

### 图形工具

[创建多边形](http://betravis.github.io/shape-tools/)

译者Ps:建议自己把它fork到仓库、

[Brackets 编辑器自带插件](http://brackets.io/)


## 译者

这篇文章是精简版，建议英文好的去看原文，会有很大收获。Sara女神每篇文章都是这么棒，干货满满还赠送各种彩蛋。








