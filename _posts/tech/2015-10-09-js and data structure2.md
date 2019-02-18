---
layout: post
title: javascript 和 数据结构 (2)
category: 技术
tags: [JS，基础,javascript,数据结构,前端总结,基础]
keywords: 前端,资料,学习
description: 
---

## 前言
今天这篇内容比较基础,回到了当初学acm的时候。感觉什么都是那么新鲜。

## 列表

```
function List(){
  this.listSize = 0;
  this.pos = 0;
  this.dataStore = [];
  this.clear = clear;
  this.find = find;
  this.toString = toString;
  this.insert = insert;
  this.append = append;
  this.remove = remove;
  this.front = front;
  this.end = end;
  this.prev = prev;
  this.next = next;
  this.length = length;
  this.currPos = currPos;
  this.moveTo = moveTo;
  this.getElement = getElement;
  this.contains = contains;
}

```

这里有点javascript设计模式所说的门户大开型对象创建模式。

然后就是一一把内容补充起来

```
  //给列表添加元素，新元素就位后，变量listSize自动加1
  function append(element){
  	this.dataStore[this.listSize++] = element;
  }

  function find(element){
  	for (var i = this.dataStore.length - 1; i >= 0; i--) {
  		if(this.dataStore[i] == element){
  			return i;
  		}
  	}
  	return -1; //找不到返回-1
  }
  //
  function remove(element){
  	var foundAt = this.find(element);
  	if(foundAt > -1){
  		this.dataStore.splice(foundAt,1);
  		--this.listSize;
  		return true;
  	}
  	return false;
  }

  function length(){
  	return this.listSize;
  }

  function toString(){
  	return this.dataStore;
  }

  function insert(element,after){
    var insertPos = this.find(after);
    if(insertPos > -1){
    	this.dataStore.splice(insertPos + 1,0,element);
    	++this.listSize;
    	return true;
    }
    return false;
  }
  
  function clear(){
  	delete this.dataStore;
  	this.dataStore = [];
  	this.listSize = this.pos = 0; //当前位置也变为0
  }

  function contains(element){ //看列表中是否包含该元素
  	for (var i = this.dataStore.length - 1; i >= 0; i--) {
  		if(this.dataStore[i] == element){
  			return false;
  		}
  	}
  	return false;
  }

  //列表移动

  function front(){
  	this.pos = 0;
  }
  
  function end(){
  	this.pos = this.listSize - 1;
  }

  function prev(){
  	if(this.pos > 0){
  		--this.pos;
  	}
  }

  function next(){
  	if(this.pos < this.listSize -1){
  		++this.pos;
  	}
  }

  function currPos(){
  	return this.pos;
  }

  function moveTo(position){
  	this.pos = position;
  }

  function getElement(){
  	return this.dataStore[this.pos];
  }
```
然后我们随便写个例子来看下效果:

```
var news = new List();
news.append("Test1");
news.append("Test2");
news.append("Test3");

news.front();
console.log(news.getElement());//Test1
news.next();
news.next();
console.log(news.getElement());//Test3
news.prev();
console.log(news.getElement());//Test2
news.next();
news.next();
console.log(news.getElement());//Test3

```
其实有人会疑问那么简单的东西为什么要自己去实现一遍，原因其实已经在上面那个例子中了，当最后我们连续执行两次next()的时候，按理说应该是超出了边界，但是最后输出还是边界前一个。这是因为在函数中已经考虑到了越界这个情况。如果真让自己独立实现一个List，可能很多人会漏掉不少东西(事实上整个例子确实还是考虑不够严谨)。

当实现了以上功能，我们可以以迭代器的形式来访问整个列表数据。

```
for(news.front();news.currPoos() >= 0;news.prev()){
  console.log(news.getElement());
}
```

然后你会发现Bug=-=,好像逻辑都没问题,但其实溢出了。其实这本书我刚搜了一下被很多人吐槽，作者写的太不严谨。不过对于我们学习而已,解决Bug也是一个学习的方式。

因为是循环溢出我们先看一下for的定义

```
for (statement 1; statement 2; statement 3) {
    code block to be executed
}
Statement 1 is executed before the loop (the code block) starts.

Statement 2 defines the condition for running the loop (the code block).

Statement 3 is executed each time after the loop (the code block) has been executed.

```
然后我们可以发现问题应该大部分出现先statement2。

然后我们写个验证

```
 for(news.end();news.currPos() >= 0; news.prev()){
 	console.log(news.pos);
 	console.log(news.getElement());
 }
``
可以发现一大堆0,说明就是多写了个等号引起的-0-，该书作者怪不得被吐槽。。。

去掉等于。

但是问题又来了，少了一个输出。倒叙输出从3开始，判断在大于0的时候停止，那么dataStore[0]的值就无法输出，毕竟数组是从0开始计数。而且由于是列表，性质特殊不好删减，因此只能最后再附加一个输出。


今天就到这里。

