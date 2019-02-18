---
layout: post
title: javascript 设计模式学习(5)
category: 技术
tags: [JS，基础,javascript,设计模式 ,设计,基础]
keywords: 前端,资料,学习
description: 
---

## 前言
因为中午的短路导致晚上有一小部分时间被占用。然后就是mac充电器放在公司未带回，导致最后电量只剩一点点=-=，现在断网学习，希望能在电没之前写完。。。

## 正文

今天正式步入设计模式的学习，单体模式是最基础而且最有用的模式。

它提供了一种将代码组织为逻辑单元的手段。

单体类可以作为命名空间，减少全局变量。或者用来将代码组织的更有条理，便于阅读和维护。

首先还是先来看个例子:

```
var Singleton = {
	attr1:true,
	attr2:10,

	method1:function(){

	},
	method2:function(){

	} 		
};
```

和我们之前接触到的很相似，可以直接通过`.`运算符来访问。你可以自由添加新成员而且可以通过`delete`来删除现有成员。但是这违反了面向对象设计一条原则:`类可以被扩展，但不应该被修改`

对象字面量只是用来创建单体的方法之一。单体是一个用来划分命名空间并将一批相关方法和属性组织在一起的对象。如果它能被实例化，它只能被实例化一次。


### 划分命名空间

```
//namespace

var Group ={}

Group.Common = {

};

Group.ObjectMethod = {

};

Group.Handler = {
	
}


```

函数中声明变量时使用的var关键词很重要，否则将被声明为全局的，会更容易干扰到全局命名空间的其他代码。

### 拥有私用成员的单体

在单体对象内创建私用成员方法是直接用下划线表示法。可以让其他程序员知道该方法或者属性是私用的，只能在对象内部用。

写法如下:

```
Group.TypeCheck = {
	//priave method
	_checkNumber:function(str){

	},
	_checkArray:function(obj){

	},
	_checkType:function(obj){

	},
	//public method
	getType:function(){
	var type = this._checkType();
	return this;

	}
}

```

一般按约定俗成，只能访问公开属性，如果因为访问私有方法而造成错误，那么是自己作死怨不得他人-0-

### 使用闭包

在单体对象中创建私用成员还可以利用闭包来实现。

```
  Group.Singleton2 = (function(){
  	var privateAttr = false;
  	var privateAttr2 = [1,2,3];

  	function privateMethod1(){

  	}

  	function privateMethod1(){

  	}

  	return{
  		publicAttr1 : true,
  		publicAttr2 : 10,

  		publicMethod:function(){
  			console.log("public1");
  		}
  	}

  })();

 Group.Singleton2.publicMethod();//public1
 console.log(Group.Singleton2.privateAttr); //undefined
 console.log(Group.Singleton2.publicAttr1);//true
 
```

从例子中你可以看到这种方式可以确保私用成员不会在单体对象之外被调用。你可以自由改变其中的实现细节。还可以用这种方法对数据和对象保护和封装。

### 惰性实例化

惰性加载用于对加载大量数据的单体。

将其实例化推迟到需要使用它的时候。

直接将上面的闭包进行修改。

```

  Group.Singleton3 = (function(){

  	var unique;
  	

  	function constructor(){
	  	var privateAttr = false;
	  	var privateAttr2 = [1,2,3];

	  	function privateMethod1(){

	  	}

	  	function privateMethod1(){

	  	}

	  	return{
	  		publicAttr1 : true,
	  		publicAttr2 : 20,

	  		publicMethod:function(){
	  			console.log("public233");
	  		}
	  	}
  	}

  	return {
  		getInstance:function(){
  			if(!unique){
  				unique = constructor();
  			}
  		  return unique;
  		}
  	}
  })();


```

用一个unique来控制

调用方式如下:

```
Group.Singleton3.getInstance().publicMethod();

```

## 结尾

单体模式的好处是它对代码的组织作用，把相关方法和属性组织在一个不会被多次实例化的但体重，可以让代码调试和维护更加轻松。

个人感觉单体模式可能很合适作为api的暴露，尤其是搭配上继承。应该能减少不少代码量。

题外话:写了一个小时，关网大概才费百分8的点=-= rmbp果然给力。。。





