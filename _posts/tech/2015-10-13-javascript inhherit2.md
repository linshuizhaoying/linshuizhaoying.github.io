---
layout: post
title: javascript 设计模式学习(3)
category: 技术
tags: [JS，基础,javascript,设计模式 ,设计,基础]
keywords: 前端,资料,学习
description: 
---

## 前言

今天将继续昨天的学习。

### 原型式继承

使用原型式继承只需要直接创建对象，让后让其他新的对象重用。它给其他对象提供了一个原型原理是原型链查找的工作机制。

我们来看个例子:

```
var Linshui = {
	name:"linshui",
	getName:function(){
		return this.name;
	}
}
```

现在Linshui是一个对象字面量.嗯，新名词，学习一下.

```
名称:对象字面量
技能:对象字面量模式可以直接在创建对象时添加功能
实现:
  将对象主体包含在一对花括号内 { 和 }。
  
  对象内的属性或方法之间使用逗号分隔。最后一个名值对后也可以有逗号，但在IE下会报错，所以尽量不要在最后一个属性或方法后加逗号。
  
  属性名和值之间使用冒号分隔。
  
  如果将对象赋值给一个变量，不要忘了在右括号}之后补上分号。

```

此时Linshui定义了基本属性和方法，并提供了默认值。方法的默认值一般不会改变，属性默认值一般会改变.

来看下如何使用:

```
var testOne = clone(Linshui);
console.log(testOne.getName());
testOne.name = "Zhaoying";
console.log(testOne.getName());

```

这里又引出一个新的函数。我们来看下它的实现:

```
function clone(object){
	function F(){};
	F.prototype = object;
	return new F; //new F == new F()
}
```

可以看到如此简介的三行就实现了我们之前类继承的效果。

clone首先创建一个新函数F，将prototype来指向obejct原型对象，通过原型链接机制，获取所有继承而来的成员的链接。然后通过new运算符作用于F创建出一个新对象。函数所返回的是以给定对象为原型对象的空对象。

通过clone，你可以拿到原型对象的属性方法。然后你可以自由添加自己的属性方法。

比如

```
var testTwo = [];
testTwo[0] = clone(Linshui);
console.log(testTwo[0].getName()); //linshui
testTwo[0].name = "ZY";
console.log(testTwo[0].getName()); //ZY

```

克隆刚被创建时，testTwo[0].name 返回指向最初Linshui.name的链接。对于从原型对象继承而来的成员，其读和写存在不对等性。如果读取testTwo[0].name时,还没有为为其实例定义name属性，得到的将是从原型链找到的"linshui"。

这说明了为通过引用传递的数据类型的属性创建新副本的重要性.

可以用hasOwnProperty方法来区分对象的实际成员和它继承而来的成员。

有时候原型对象自己有子对象，想覆盖其中的一个属性值，你不得不重新创建整个子对象。为了弱化对象之间的耦合，任何复杂的子对象都应该用方法来创建。

可以用工厂方法写更优美的实现。

今天先到这里，明天继续。


