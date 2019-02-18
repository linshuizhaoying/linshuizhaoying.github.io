---
layout: post
title: Chrome console tips 系列学习
category: 技术
tags: [总结,开发,前端,移动端]
keywords: 总结,开发,前端,移动端]
description: 
---
# 前言
先发一篇学习摘记。

## 描述

  来自[DevTools tips系列文章](https://medium.com/@tomsu)中摘取的Chrome tips。

  只摘取了认为对前端开发效率提升有帮助的部分。
  
  更多更新内容可以去[google developers](https://developers.google.com/web/updates/2018/12/nic71)。


### console中的'$'

在你还没有在app中定义 `$变量的情况下(例如 jQuery)`, $在console中是冗长的函数document.querySelector的一个别名。

但是 `$$ ` 能节省更多的时间，因为它不仅仅执行 `document.QuerySelectorAll`并且返回的是一个节点的数组，而不是一个`Node list`

从本质上说:

>Array.from(document.querySelectorAll('div')) === $$('div')

![1546427866568.jpg](http://img.haoqiao.me/blog1546427866568.jpg)

### $i

>引入 chrome 插件 `Console Importer`,可以快速的在console中引入和使用npm库。

典型的比如 `$i('lodash')` 或者 `$i('moment')`.

![chrome tips 1.gif](http://img.haoqiao.me/blogchrome%20tips%201.gif)


### copy 与 Store as global

> 通过全局的方法copy()在console里copy任何你能拿到的资源

> console中打印了一堆数据(例如 你在app中计算出来的一个数组),然后你想对这些数据做一些额外的操作,只需要右击它，并且选择“Store as global variable” (保存为全局变量)这个选项。

结合这两个，就可以不需要去`network`复制返回的数据还要自己处理格式。如果有写前端日志打印，就不需要自己去代码里输出手动断点了。

![chrome tips 2.gif](http://img.haoqiao.me/blogchrome%20tips%202.gif)


### console.table

>使用console.table方法将它以一个漂亮的表格的形式打印出来。它不仅仅会根据数组中包含的对象的所有属性，去计算出表中的列名，而且这些列都是可以缩放甚至...可以排序

>当列太多的时候，使用第二个参数，传入你想要展示的列对应的名字

![chrome tips 4.gif](http://img.haoqiao.me/blogchrome%20tips%204.gif)

### 监测执行时间

>console.time() — 开启一个计时器
>console.timeEnd() — 结束计时并且将结果在 console 中打印出来

###  Snippets

>在导航栏里面选中 Snippets 这栏，点击 “New snippet(新建一个代码块)” ,输入你的代码，保存
>通过右击菜单或者 [ctrl] + [enter] 快捷键来运行它

![chrome tips 5.gif](http://img.haoqiao.me/blogchrome%20tips%205.gif)

>一旦设置了一组很棒的代码块,使用 `Command Menu`。如果你输入 ! 在它的输入框中，你可以根据名字来选择你的代码块

![chrome tips 6.gif](http://img.haoqiao.me/blogchrome%20tips%206.gif)

## 其它

### H健

>按一下'h'来轻松隐藏你在元素面板中选择的元素。再次按下'h'可以使它出现。这在某些的时候是很有用的，例如你想截图，但是你又不想里面包含一些敏感信息的情况。

![1679379780c11ef3.gif](http://img.haoqiao.me/blog1679379780c11ef3.gif)

###  Command (命令) 菜单

>在 Chrome 的调试打开的情况下 按下 [ Ctrl]+[Shift]+[P] (or [⌘]+[Shift]+[P] on Mac)
>

或者

![1679a2adf8945253.jpg](http://img.haoqiao.me/blog1679a2adf8945253.jpg)

具体内容可以一个个尝试。它可以改变开发者模式外观，切换面板等。

> 开启command，键入time。可以让Log输出带上时间。

![167a467d9f9ff467.gif](http://img.haoqiao.me/blog167a467d9f9ff467.gif)


### console.log 加上 css 样式

![dutufunxcaahcmu.jpg](http://img.haoqiao.me/blogdutufunxcaahcmu.jpg)

