---
layout: post
title: 高性能javascript 读书笔记
category: 技术
tags: [js,读书笔记,学习整理]
keywords: 前端,资料,学习,javascript
description: 
---

## 前言
这是补基础第二波，买来的书终于看完一半了。《高性能javascript》的确很适合有一定基础的看，有不少东西都是以前忽略甚至是没听过的总结。以下将总结折页的内容。

## 正文

###原型

`javascript的对象是基于原型的。原型是其他对象的基础。它定义并实现了一个新创建的对象所必须包含的成员列表。`

原型对象为所有对象实例所共享，这些实例也共享了原型对象的成员。

对象有两种类型：实例成员（直接存在于对象实例中）和原型成员（从对象原型中继承而来）。

可以用`hasOwnProperty()`方法来判断对象是否包含特定的实例成员(传递给方法名即成员的名称)。要确定对象是否包含特定的属性，可以用In操作符(然而事实上In最好只用于对象内的遍历。其余遍历最好是知道length然后循环。)

来个例子:

```
var book = {
  title : "linshuizhaoying"
};
console.log(book.hasOwnProperty("title")); //true
console.log(book.hasOwnProperty("toString")); //false
```

### 访问集合元素时使用局部变量
这个应该是蛮实用的，当你不用Jquery而是原生自己去实现对同一个dom属性或方法多次访问时。或者遍历一个dom集合。用局部变量来缓存，我们直接来看最优的写法:

```
function collect(){
  var coll = document.getElementsBytagName('div'),
    len = coll.length,
    name = '',
    el = null;
  for(var count = 0; count < len ; count ++){
    el = coll[count];
    name = el.nodeName;
    name = el.nodeType;
  }
  return name;
}

```

可以看到先缓存了集合的长度，然后在遍历中，又缓存了当前集合元素。

### 选择器Api
这个是我以前看到过的，但是没怎么用。这里提到了可以提高效率而且简化写法。

```

var elements = document.querySelectorAll("#menu a");

```

querySelectorAll()方法使用css选择器作为参数并返回一个Nodelist（包含匹配点的数组对象）。这个方法不会返回Html集合，因此返回的节点不会对应实时的文档结构。

这个在某些情况下能代替jquery的选择器查询。

而且它可以同时获得不同class元素的集合.

```

var elements = document.querySelectorAll("div.error,div.notice");

```

而且可以用querySelector()获得第一个匹配的节点。


### 重排和重绘
提及WEB性能好像永远绕不开这两个。

当Dom变化影响了元素的几何高度（高或宽）,浏览器需要重新计算元素的几何属性。浏览器使渲染树中受影响的部分失效，重构渲染树。这个过程成为`重排(reflow)`. 完成重排后,浏览器会重新绘制受影响的部分，这叫`重绘(repaint)`。

重排的发生

```

添加或删除可见的Dom元素
元素位置改变
元素尺寸改变
内容改变（文本替换或者图片替换）
页面渲染器初始化
浏览器窗口尺寸改变

```

减少重排次数的方案:

这个是这本书里看到的干货，平时也没怎么接触这块来着。。。

`在文档之外创建并更新一个文档片段。并把它附近到原列表。`

文档片段的一个便利的语法特性是当你附加一个片段到节点中，实际上被添加的是该片段的子节点，而不是片段本身。

```
 
 var fragment  = document.createDocumentFragment();
 appendDataToElement(fragment,data);
 document.getElementById('mylist').appendChild(fragment);
 
```
appendDataToElement 是更新指定节点数据的通用函数，这里就步抄了，事实上对文档片段感兴趣可以去翻文档，应该有更详细的介绍与例子。我只是总结一下知识点。方便以后查找。

使用下列步骤可以避免`动画`中页面中的大部分重排:

1.使用绝对位置定位页面上的动画元素，让它脱离文档流。
2.让元素动起来。当它扩大时，会临时覆盖部分页面。当不会引起重排。只会引起小区域的重绘。
3.当动画结束时，恢复定位，从而只会下移一次文档的其他元素。

##结尾
今天先到这里。明天继续。





