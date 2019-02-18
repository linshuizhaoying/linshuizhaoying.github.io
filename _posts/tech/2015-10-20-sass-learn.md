---
layout: post
title: Sass的深入学习
category: 技术
tags: [JS，Sass,Css,vue.js,vue]
keywords: 前端,资料,学习
description: 
---

## 前言

周末回来后整理了一下所得，然后这两天在思考如何将收获的用到项目里。因此这两天在整理基于vue.js的前端自动化流程。今天正好将大漠老师分享的东西整理一下。

## 整理学习

首先以前学习sass基础是从[这里](http://www.w3cplus.com/sassguide/syntax.html)

因为之前只用过嵌套，因此这次直接从比较生涩的内容开始。

### Sass 继承

```
sass:

.mt5{
  margin-top:5px;
}
.block{
  @extend .mt5;
}

output css:

.mt5,.block{
  margin-top:5px;
}

```

看上去extend是将继承来的类名也附加到后面。

看下说明`选择器继承可以让选择器继承另一个选择器的所有样式，并联合声明`

### Sass占位符

```
sass:
%mt5{
  margin-top:5px;
}
.block{
  @extend %mt5;
}

output css:

.block {
  margin-top:5px;
}

```

看上去是继承后把原来的类名替换掉。
看下说明`这种选择器的优势在于：如果不调用则不会有任何多余的css文件`

### 混合宏@mixin

```
sass:

@mixin mt($var){
  margin-top:$var;
}
.block{
  @include mt(5px);
}

output css:

.block {
  margin-top:5px;
}

```

这个一看起来就觉得最方便，当你想细调整某些组合的时候这个混合宏感觉可以起很大作用，

### 嵌套

这个好像如果觉得sass很棒都会夸这点，毕竟真的很方便，只需要记住`&`是表示父元素选择器

```
sass:
.header{
  padding:5px;
  margin:5px;
}
  nav{
    background-color:red;
  }
  
  &:after{
    clear:both;
  }
  
  .home & {
    background-color:green;
  }

output css:

.header{
  padding:5px;
  margin:5px;
}
.header nav {
  background-color:red;
}

.header:after {
  clear:both;
}

.home .header{
  background-color:green;
}

```

`&`还有一种用法，是这样的:

```
sass:
.block{
  color:blue;
}
  &__e{
    color:yellow;
  }
  &--m{
    color:green;
  }

output css:

.block{
  color:blue;
}
.block__e{
  color:yellow;
}

.block--m{
  color:green;
}

```

感觉这个比较适合写列表或者应用在组件中-0-

### @at-root

用来跳出选择器嵌套的。默认所有的嵌套，继承所有上级选择器，但有了这个就可以跳出所有上级选择器。

比如写着写着写歪楼了，用这个跳出来。

但是我感觉我更愿意自己重新另起一行来写。。。

### 循环

@for循环

```
sass:

@for $i from 1 through 3 {
  .item-#{$i} { width: 2em * $i; }
}

css:

.item-1 {
  width: 2em; 
}
.item-2 {
  width: 4em; 
}
.item-3 {
  width: 6em; 
}
```

@each

```
sass:

$animal-list: puma, sea-slug, egret, salamander;
@each $animal in $animal-list {
  .#{$animal}-icon {
    background-image: url('/images/#{$animal}.png');
  }
}

css:
.puma-icon {
  background-image: url('/images/puma.png'); 
}
.sea-slug-icon {
  background-image: url('/images/sea-slug.png'); 
}
.egret-icon {
  background-image: url('/images/egret.png'); 
}
.salamander-icon {
  background-image: url('/images/salamander.png'); 
}


```
list map 虽然感觉好像不知道什么时候能用到-0-万一要用呢，先记录一下。

list用法：

```
$animal-data: (puma, black, default),(sea-slug, blue, pointer),(egret, white, move);
@each $animal, $color, $cursor in $animal-data {
  .#{$animal}-icon {
    background-image: url('/images/#{$animal}.png');
    border: 2px solid $color;
    cursor: $cursor;
  }
}

css:

.puma-icon {
  background-image: url('/images/puma.png');
  border: 2px solid black;
  cursor: default; 
}
.sea-slug-icon {
  background-image: url('/images/sea-slug.png');
  border: 2px solid blue;
  cursor: pointer; 
}
.egret-icon {
  background-image: url('/images/egret.png');
  border: 2px solid white;
  cursor: move; 
}
```

map写法

```
//sass style
//-------------------------------
$headings: (h1: 2em, h2: 1.5em, h3: 1.2em);
@each $header, $size in $headings {
  #{$header} {
    font-size: $size;
  }
}

//css style
//-------------------------------
h1 {
  font-size: 2em; 
}
h2 {
  font-size: 1.5em; 
}
h3 {
  font-size: 1.2em; 
}
```

### sass的项目中使用


```
 modules目录是用来放置Sass文件的，他不会编译出CSS文件。主要放置了混合宏（mixins）、函数(functions)和变量(variables)这些东西。
 
 partials目录等组件（或者还有其他的）。将其化分更多的类别（typography, buttons, textboxes, selectboxes等等）。
 
vendor目录放的是第三方的CSS。放置了由其他人(或你自己为其他项目开发的其他组件)开发的预先封装的组件。比如说在vendor目录中放置了jQuery UI和Color picker组件。

```

这是别人的做法，然后我准备用到项目中是这样的:

```
因为我是用vue，因此我在vue目录建立assets

vue ->
    sass 自己模块化的sass 相当于上面的modules
    css 大型框架或者插件的css
    
```
其它我打算直接在组件中写，调用外置的sass目录下的文件来继承。



## 其它
这部分是关于我在构建前端自动化流程的时候所总结的。

```
1.对于vue.js 用webpack加载sass-loader模块，注意，直接在根目录 npm install sass-loader
因为它有需要Libsass依赖等等。

然后在loader里面写

test: /\.css$/,
loader: ExtractTextPlugin.extract("style-loader", "sass-loader")

2.之前有考虑是在组件内加载bootstrap这类,但后来发现其实适得其反，用sass的@important最后还是加载基础的局部样式。

```

今天就是整理一下深入学习一点，更多的想法打算实践后再总结。

