---
layout: post
title: 对javascript中apply和call的一点探索
tags: [javascript,apply,call]
keywords: 学习资料,前端,javascript
description: 
---
## 写在前面



## 正文

Function.prototype.bind = function(){
	var self = this;//保存函数
	context = [].shift().call(arguments),//需要绑定的this上下文
	args = [].this.call(arguments);//剩余的参数转为数组
	return function(){
		return self.apply(context.[].concat.call(args,[].slice.call(arguments)));
	}
	
	
};
var obj = {
	name = "Linshui";
}

var func = function(a,b,c,d){
	alert(this.name);
	alert([a,b,c,d]);
}.bind(obj,"x","y");

func("xyt","zhaoying");