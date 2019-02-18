---
layout: post
title: javascript 设计模式学习(6)
category: 技术
tags: [JS，基础,javascript,设计模式 ,设计,基础]
keywords: 前端,资料,学习
description: 
---

## 前言

今天继续设计模式的学习，但是书上例子比较枯燥=-=,因此今天先整理概念。然后再找例子加深印象。

## 工厂模式

概念

```
一个类或者对象可能包含别的对象，创建这种成员对象时，可能习惯于用new关键词和类构造函数。工厂模式是一种有助于消除这两个类之间的依赖性的模式。它使用一个方法来决定究竟要实例化哪个具体的类。
```

### 简单工厂模式

这种模式把成员对象的创建工作转交给一个外部对象。这个外部对象可以有一个简单命名空间，可以是类的实例。或者把这个创建方法实现在一个类中，然后从这个类派生出一个子类.(用于创建实例的方法的逻辑会发生变化)

```简而言之就是用一个类或者对象封装实例化操作。```

### 工厂模式

真正的工厂模式与简单工厂模式的区别在于，它不是用一个类或对象来创建，而是用一个子类。按照正式定义:

```
工厂是一个将其成员对象的实例化推迟到子类进行的类。

```

```简而言之就是实现一个抽象的工厂并把实例化工作推迟到子类进行。```

### 为何选择工厂模式

1.当需要用一些不同方式实现同一接口的对象，那么可以用一个工程方法或者简单工厂对象来简化选择实现的过程。

2.用来集中设置代码或者检测代码。适用于对象需要进行复杂而且彼此相关的设置或检测，而且这种设置只需要为特定类型的所有实例执行一次。

3.用来将小型对象组成一个大对象。让子系统能与大对象解耦。

工厂模式的好处在于消除对象间的耦合，通过工厂方法而不是new关键词以及具体类。可以把所有实例化代码集中到一个位置。可以简化更换所用类或者运行期间动态选择所用的类的工资。在派生子类它提供更大的灵活性。
使用工厂模式，先创建一个抽象的父类。然后在子类中创建工厂方法。从而把成员对象实例化推迟到更专门化的子类中进行。

坚守两条面向对象设计原则:
1.弱化对象间的耦合
2.防止代码的重复。

### 例子
学完理论我们得来几个例子加深学习印象，列一下我们需要的例子：

```
1. 用一个类或者对象封装实例化操作的简单工厂模式
2. 将其成员对象的实例化推迟到子类进行的真正工厂模式。

```

从理论中我们可以得知简单工厂模式把成员对象的创建工作转交给一个外部对象,那么例子应该是这样:

```
function BuyBooks() {}

BuyBooks.prototype = {
  createBook: function(options) {
    var Book;

    switch(options.type) {
      case 'Science':
        Book = new ScienceBook();
        break;
      case 'Art':
        Book = new ArtBook();
        break;
      default:
        Book = new Financial();
    }

    return Book;
  }
};

```

在使用这个类生产对象的时候，传入option参数，在参数中的type属性规定我们需要的类型，构造函数就能够返回我们需要的对象类型了。比如我这里举的是买书,买书的动作不是我们这里面考虑的，我们考虑的是买什么书，然后选择让负责买这类书的人去买。

如果需要要添加新的书的类型也是很方便的，在工厂的switch中直接添加一个case就可以了。然后我们再在外面去创建一个新的类型的类。

简单工厂模式会把创建工作交给外部的类来做，这实际上会增加类的数量，并不利于代码的组织。真正的工厂模式会把创建工作交给子类来完成，父类只对创建过程中的一般性问题进行处理，这些处理会影响到每个子类，而子类之间相互独立，可以对创建过程进行一些定制化操作。

而且结合我们上面理论中提到的真正的工厂模式是先创建一个抽象的父类。然后在子类中创建工厂方法。从而把成员对象实例化推迟到更专门化的子类中进行。

我们先抽一个父类，根据我们上面举的简单工厂的例子，我们指出每个类型的书都需要专门的人买，那如果我负责一个区域的书店购买，那我肯定需要每个城市都有一个负责人，然后他来组织相应去买不同类型书的人.

因此我们需要这么写:


```
//父类抽象化
function Owner() {}

Owner.prototype = {
  createBuyBooks: function(options) {
    // 这里不直接指派，如果直接调用会抛出错误
    throw new Error('不能操控这个抽象类.')
  }
};


```

然后对于子类需要继承父类的所有属性，然后开辟自己的方法。

```
function BookBuyA() {}

// 继承方法
extend(BookBuyA, Owner);

// 实现自己的createBook方法
BookBuyA.prototype.createBuyBooks = function(options) {
  createBook: function(options) {
    var Book;

    switch(options.type) {
      case 'Science':
        Book = new ScienceBook();
        break;
      case 'Art':
        Book = new ArtBook();
        break;
      default:
        Book = new Financial();
    }

    return Book;
  }
}
```

这里你会看到一个extend方法，我们之前学过的，因此直接拿来主义.

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

如果一个类中包含了很多更小的子类作为自己的组成部分，那么替换这些子类的工作会很简单，因为工厂模式降低了模块之间的耦合度，一个模块并不会依赖于其某一组成部分。

今天就到这里。

