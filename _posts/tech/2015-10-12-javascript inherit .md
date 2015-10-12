---
layout: post
title: javascript 设计模式学习(2)
category: 技术
tags: [JS，基础,javascript,设计模式 ,设计,基础]
keywords: 前端,资料,学习
description: 
---

## 前言
今天抽时间看了一章设计模式,对于继承有了点感觉，做下学习笔记。并为以后重构项目打下基础。

### 简单的类声明

```
function A(param){
	this.param = param;
}

A.prototype.getParam = function(){
	return this.param;
}

```

这里注意类名首字母大写。

调用方式如下:

```
var b = new A("It's B.");
console.log(b.getParam());

```

### 原型链

现在用一个C来继承

```
	function C(id,title){
		A.call(this,title);
		this.id = id;
	}

	C.prototype = new A();
	C.prototype.constructor = C;
	C.prototype.getID = function(){
		return this.id;
	}
```

这里你会发现，代码分开我都认识，合起来是什么意思？

这涉及到一些底层原理，也是我非常心水的部分。

我们先来列出其中的几个原理:

```
1.在使用new运算符时，系统会先创建一个空对象，然后调用构造函数。此过程中空对象处于作用域链的最顶端。
2.在C函数中调用超类的构造函数时，必须手工完成类似的任务。(超类不理解没关系，下面会有讲解)
3.让一个类继承另一个类，只需要将子类的prototype设置为指向超类的一个实例即可。
```

我们分别来对原理进行二次分析。
第一条我们可以无视。
第二条，超类的概念，我们举个例子

```
prototype由构造函数Object()创建,所以它本身也是一个Object实例,而任何Object实例都可以从Object.prototype中继承属性；于是prototype就也具有了这种能力。 

基于prototype的继承不仅仅局限于单一的prototype对象,访问沿着一条prototype链逐级向上执行：假设有个Complex实例，访问其中一属性，如果本身找不到，便访问Complex.prototype对象;还找不到即在Object实例中找不到，就接着访问Complex.prototype的上一级--Object.prototype。 

由于Complex.prototype默认是Object的实例(由Object()初始化)，于是Complex便继承了Object(即可以访问Object和Object.prototype的所有属性)。 
  
只要把Complex.prototype的构造函数换成其他的，而不是默认的Object()，那Complex便成为了那个类的子类；这就实现了自己定义的超类和子类关系。当然Complex还是Object的子类，但那是因为那个类最终也是Object的子类。 
 
```

第三点其实就是补充第二点，javascript中每个对象都有一个prototype的属性，它要么指向另一个对象，要么为null.

因此整个流程我们可以这么总结：
为了让C继承A，我们必须手工将A的prototype设置为Person的一个实例。
最后一步是为了让protype的constructor属性重设为A（因为prototype属性设置为A的属性时，其constructor属性被抹除了.）

这里我们又有个疑问，constructor是用来干啥的？
直接MDN之。

```
返回一个指向创建了该对象原型的函数引用。需要注意的是，该属性的值是那个函数本身，而不是一个包含函数名称的字符串。对于原始值（如1，true 或 "test"），该属性为只读。
```

从描述来看只看懂了创建引用，来继续看个例子:

```
所有对象都会从它的原型上继承一个 constructor 属性：

var o = new Object // 或者 o = {}
o.constructor == Object
var a = new Array  // 或者 a = []
a.constructor == Array
var n = new Number(3)
n.constructor == Number
```

看懂一点，但没怎么理解，再看下一个例子:

```
function Tree(name) {
   this.name = name;
}

var theTree = new Tree("Redwood");
console.log( "theTree.constructor is " + theTree.constructor );
/*

console: theTree.constructor is function Tree(name) {
   this.name = name;
}

*/

```
这下有点懂指向创建了该对象原型的函数引用的意思了。

接下来是创建这个子类的实例。和创建A类没什么区别

```
var c = [];
c[0] = new C("testtitle","testid");
c[1] = new C("testtitle2","testid2");
console.log(c[0].getID());
console.log(c[1].getParam());
```
看下结果就可以发现测试通过。建议都自己手打一遍加深印象.

### entend函数

javascript没有原生的extend的关键词，因此想要简化派生子类的过程，我们可以把写个extend函数封装起来。直接来看比较成熟的写法:

```
function extend(subClass,superClass){
	var F = new function(){};
	F.prototype = superClass.prototype;
	subClass.prototype = new F();
	subClass.prototype.constructor = subClass;

	subClass.superClass = superClass.prototype;
	if(superClass.prototype.constructor == Object.prototype.constructor){
		superClass.prototype.constructor = superClass;
	}
}

```

这个函数与之前手工做的一样，不过它引进了空函数F，利用它创建的一个对象实例插入原型链。可以避免创建超类的新实例,因为创建超类的新实例会造成一些不必要的消耗（比如大量计算，比如可能比较庞大创建浪费资源）

它还引入了superclass属性,用来弱化两个类直接的耦合。最后三行用来确保超类的constructor属性被正确设置(保证在子类中正确调用到父类的构造函数 ，而不是Object。)。

我们可以这样用它

```
function D(title,id){
	D.superClass.constructor.call(this,title);
	this.id = id;
}
extend(D,A);

D.prototype.getID = function(){
	return this.id;
}
var d1 = new D("TTT","VVV");
console.log(d1.getID()); //VVV
D.prototype.getParam = function(){
	var param = D.superClass.getParam.call(this);
	return "This is function redefined param:" + param;
}
var d2 = new D("TTT","VVV");
console.log(d2.getParam());
var a3 = new A("Linshui","Zhaoying"); //This is function redefined param:TTT
console.log(a3.getParam());//Linshui

```

在测试的时候发现书上有一点写错了，就是extend函数中你定义

```
	var F = new function(){};
```

然后再调用

```
subClass.prototype = new F();

```

会报错.

应该将其改为

```
function F(){}

```

有了superclass函数，可以直接调用超类的方法，可以重定义超累的某个方法而又想访问其在超类中的实现可以派上用场。(就是你想重写子类的getParam方法而且需要在子类的getParam中引用超类的getParam)

## 结尾
看书刷刷的很快，但是自己重写按照书中思想实现的时候发现有很多还是不懂，不过经过一个多小时学习，终于摸到了继承的边。下一章是原型式继承的进阶篇。

今天就到这里。


