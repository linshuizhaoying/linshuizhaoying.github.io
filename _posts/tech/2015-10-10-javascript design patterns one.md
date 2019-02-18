---
layout: post
title: javascript 设计模式学习(1)
category: 技术
tags: [JS，基础,javascript,设计模式 ,设计,基础]
keywords: 前端,资料,学习
description: 
---

## 前 言
这几天开始学习javascript设计模式,并记录一些学习的东西。

## 正文

### 高级对象创建模式

###

show code:

```
/**
   * @param  {传入id,title,author参数}
   * @return {function}
   */
	var Book = (function(){
		//私有静态属性
		var numOfBooks = 0; //用于跟踪Book构造器的总调用次数。
		//私有静态函数 判断一个私用方法是否应该被设为静态方法，只需要看它是否需要访问实例数据，如果不需要
		//那么应该把它设为静态的，它只会被创建一份。
		function checkId(id){ //该函数被设为静态方法，因为对每一个实例都生成一个新副本是无意义的，浪费的。
			return id % 2 == 0; 
		}

		//返回一个函数,使Book变成一个构造函数，实例化Book时，调用
		return function(newId,newTitle,newAuthor){ //接口发布

    	//私有属性 
    	var id,title,author;

    	//私有函数 
    	this.getid = function(){
    		return id;
    	}

    	this.setid = function(newId){
    		if(!checkId(newId)) throw new Error('Book Id is error');
    		id =  newId;
    	}

    	this.getTitle = function(){
    		return title;
    	}

    	this.setTitle = function(){
    		title = newTitle || "No Title Here";
    	}

    	this.getAuthor = function(){
    		return author;
    	}

    	this.setAuthor = function(){
    		author = newAuthor || "No author Here";
    	}

    	numOfBooks++;

    	if(numOfBooks > 3) throw new Error("Books number exceed max");

    	this.setid(newId);
    	this.setTitle(newTitle);
    	this.setAuthor(newAuthor);
		}
	})(); //这个括号是匿名函数一部分，作用是代码一载入就执行这个函数

		//公开静态函数 这个可用于扩展新功能
		Book.print = function(){
			console.log("This is print");
		};

		//prototype属性的解释是：返回对象类型原型的引用。
		Book.prototype = {
			display: function(){
				console.log("我是公有函数，谁都能调用。")
			}
		};

		Book("2","测试1","linshui");

		console.log(Book.numOfBooks); //不能直接访问
		Book.print();
		var books = new Book("4","测试1","linshui");
		books.display();//引用不会出错。
		Book.display();//直接访问会出错
```


