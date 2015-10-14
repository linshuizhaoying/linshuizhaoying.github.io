---
layout: post
title: javascript 设计模式学习(4)
category: 技术
tags: [JS，基础,javascript,设计模式 ,设计,基础]
keywords: 前端,资料,学习
description: 
---

## 前言
由于这两天课有点多,没多少空余时间。今天中午抽空把昨天剩下的部分学习。

### 工厂方法

昨天学到了用工厂方法来创建复杂的子对象。
来写个例子：

```
Base.childObejct = Base.creteChildObejct();

var baseClone = clone(Base);

baseClone.childObejct = Base.creteChildObejct();

baseClone.childObejct.num = "20";

console.log(Base.childObejct.num); //30
console.log(baseClone.childObejct.num);//20

```

这种写法能够在不知道base具有多少个属性时改变自己需要的属性的默认值。

这里又有一个新名词,工厂方法，来看下维基百科上它的简介:


```

工厂方法模式（英语：Factory method pattern）是一种实现了“工厂”概念的面向对象设计模式。就像其他创建型模式一样，它也是处理在不指定对象具体类型的情况下创建对象的问题。工厂方法模式的实质是“定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行。”

```

创建一个对象常常需要复杂的过程，所以不适合包含在一个复合对象中。创建对象可能会导致大量的重复代码，可能会需要复合对象访问不到的信息，也可能提供不了足够级别的抽象，还可能并不是复合对象概念的一部分。工厂方法模式通过定义一个单独的创建对象的方法来解决这些问题。由子类实现这个方法来创建具体类型的对象。

工厂方法也是一种设计模式，因此我们可以知道我们能够在后面更加深入的学习这个模式。

### 用类式继承还是原型式继承

如果你设计是API,建议用类式继承。

原型链继承更能节省内存。原型链成员的方式使所有克隆出来的对象都共享每个属性和方法的唯一一份实例。只有设置类某个克隆对象的属性和方法才会改变。而且它使用方法，只需要一个clone函数。

最重要的是理解两者的原理。至于最终用哪种都是看自己喜欢了。

## 掺元类
这是一种重用代码不需要严格的继承。如果把一个函数用到多个类中，可以通过扩充方式让这些类共享该函数。

大概方法如下:

```
先创建一个包含各种通用方法类，然后用它扩充其他类。它通常不会实例化或者直接调用，其存在的目的只是向其他类提供自己的方法。

```

来看个例子:

```
//掺元类
  
  var Mixin = function(){};
  Mixin.prototype = {
  	test1:function(){
  		console.log("你调用了test1方法");
  	},
  	test2:function(){
  		console.log("你调用了test2方法");
  	},
  	test3:function(){
  		console.log("你调用了test3方法");
  	}
  }



```

重点是argmet函数，我们直接来看成熟的写法:

```
function argment(receivingClass,givingClass){
	if(arguments[2]){//只给了主要函数
		for(var i = 2,len = arguments.length; i < len; i++ ) {
			 receivingClass.prototype[arguments[i]]=givingClass.prototype[arguments[i]];
		}
	}else{ //给了所有的函数
		for (methodName in givingClass.prototype) {
			if(!receivingClass.prototype[methodName]){
				receivingClass.prototype[methodName] = givingClass.prototype[methodName];
			}
		}
	}
}
```

然后你可以这样调用，在测试的时候由于书上没给例子加上脑子这几天机房辐射的短路，造成调试了很久才发现问题,那就是给传入方法的类新new一个出来=-=

```
  function testThree() {}
  
  function testFour() {}
  
  argment(testThree,Mixin,['test1']);//你调用了test1方法
  argment(testFour,Mixin);//你调用了test2方法
  
  var testNew = new testThree;
  var testNew2 = new testFour;

  testNew.test1();

  testNew2.test2();
```

这种方法只能适配于给一个类赋值一个方法或者全部方法。当你想多个比如

```
 argment(testThree,Mixin,['test1','test3']);
```

是不行的。因此你可以用采用以下写法

```
function augment(destClass, srcClass, methods) {
    var srcProto  = srcClass.prototype
    var destProto = destClass.prototype    
    for (var i=0; i<methods.length; i++) {
        var method = methods[i]
        if (!destProto[method]) {
            destProto[method] = srcProto[method]
        }
    }
}
```

从条理性看，严格的继承方案比扩充方案更加清楚。掺元类比较适合组织那些完全不同类之间共享方法。

## 小结

本来中午是可以写完的，但是因为忘记类还需要new一下再用，导致调试了一会，造成临近上课也没写完-0-。
 
接下来继续学习。



