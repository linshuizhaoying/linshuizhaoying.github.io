---
layout: post
title: contentEditable的Change判断
category: 技术
tags: [Vue.js，Vue.js学习,项目小结,前端学习,前端总结，Html5]
keywords: 前端,资料,学习
description: 
--- 
## 前言
最近项目用到一个`contentEditable`属性，它可以用于表明元素是否是可编辑的。也就是说，它配合事件监听能达到的效果是这样的：

![imgn](http://haoqiao.qiniudn.com/contenteditable.png)

这个效果我是用jquery 的contentEditable插件，它可以监控是否内容是否修改。用在后台的编辑功能是蛮实用的。但是这插件在我另一个项目中却"失联"了。因此只能考虑自己来实现监控。

## 思路
其实非常简单，因为数据是动态添加的，只需要用Jquery的on来监听事件。
首先在`focusin`事件中将内容先提前保存（在事件之前先定义一个变量来$lasttext）

```
$lasttext="";//为了防止点击某个字段然后点击其它字段造成判断误差

$lasttext = this.innerText;
```
然后失去焦点的时候，直接判断当前文本是否修改，如果修改那么就向服务器提交修改后的数据。

```
$(".xxx").on("blur",".xxx",function(event){
          	if(this.innerText != $lasttext){
          		console.log("changed!");
          		//从属item
          		var item = event.target.attributes.belong.value;
          		//从属字段
          		var field = event.target.attributes.field.value;
          		//改变后的内容
          		var change = this.innerText;
          		console.log("正在向服务器发送:" + "记录" + item + "字段" + field + "内容" + change);
          	}
 						$(this).attr('contentEditable','false');
          });
```
你会发现我这里直接根据event的target获取相关信息，这里你可以随意。

## 结尾
今天先补充个小技巧，还有我书单已经更新，感兴趣的可以去看看。


