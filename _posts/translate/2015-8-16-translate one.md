---
layout: post
title: 译文-React’s diff algorithm（React diff算法）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://calendar.perfplanet.com/2013/diff/)

### 作者:Christopher Chedeau

### 译者:临水照影


## 正文之前

这是第一篇正式的翻译文=-=，以后会陆续将看到好的文章放上来。毕竟打算做一个前端咨询站，内容包括国外前沿技术，国内优秀文章等等。

## 正文

React 是一个Facebook用来构建用户界面开发的javascript库。它是基于高性能而设计的。这篇文章我将阐述它的diff算法和渲染过程来让你优化你的项目。

### Diff Algorithm
在我阐述细节之前，先预览React是如何工作的非常作用。
    
    var MyComponent = React.createClass({ render: function() { if (this.props.first) { return <div className="first"><span>A Span</span></div>; } else { return <div className="second"><p>A Paragraph</p></div>; } } });

在任何时候，当你描述你所想要的UI界面。理解渲染的结果不是真实的Dom节点这个概念非常重要。这些都是轻便的javascript对象，我们称它们为（virtual DOM）虚拟节点。

React将用这种方式试图找到从渲染的上一步到下一步之间的最小数目。举个栗子，如果我们想要将
    
     <MyComponent first={true} />
     
用  
     
     <MyComponent first={false} />
     
来替换，然后移除它。这是Dom操作的过程：
    
    刚开始没有节点
      开始创建节点 <div className="first"><span>A Span</span></div>
    第一步到第二步
      替换属性：用className="second" 替换 className="first" 
      替换节点：用<p>A Paragraph</p> 替换 <span>A Span</span>
     第三步
       移除节点 <div className="second"><p>A Paragraph</p></div>
    
### 逐级
在两颗任意树(不理解树的童鞋可以看wiki的[介绍](https://en.wikipedia.org/wiki/Tree_(data_structure)))之间找到最小的差异是一个 O(n^3)级别（时间复杂度）的问题。你可以想象一下，这肯定不是我们需要的解决方案。React用了最简单也最实用的方式，这个解决方案近似于O(n)的复杂度。
    
React只是试图逐级对比两颗树。这大大降低了解决问题的复杂性而且在Web应用中将一个组件移到其他级的树上它不仅不会造成很大浪费而且会很节约。它们通常只移动子节点。

![img1](http://img.haoqiao.me//t-1-1.png)

### 列表
 让我们来假设我们有个组件它将迭代渲染5个组件并将在下一步中插入一个新的新的组件放到迭代列表中间。这很难知道两个组件列表之间的映射关系。
 
 默认的，React将上一步的第一个组件作为下一个列表中的第一个。你可以提供`key`属性来帮助React找出映射。实践中，这通常很简单的能找到其中列表中拥有唯一key的子组件。
 
![img2](http://img.haoqiao.me//t-1-2.png)

### 组件
一个React组件通常由用户自定义的组件构成，最后形成以 `div`为主的树。当React匹配相同组件的时候这些额外的信息将被diff算法用来查找具有相同类的组件。

举个栗子，一个 `<Header>`被一个`<ExampleBlock>`代替。React将会移除`<Header>`并创建新的块。我们不需要花费昂贵的时间试图匹配两个组件因为他们不可能有相同处。

![img3](http://img.haoqiao.me//t-1-3.png)

### 事件委派

对Dom节点附加事件监听是很浪费的行为。React用一种称为“事件委托”的流行的技术来代替。React走的远它实现了基于w3c兼容事件系统。这意味着IE8的事件处理的各种Bug将一去不复返，所有事件名将跨浏览器一致。

让我来解释如何实现的。一个简单地事件监听是附加在Document下的根节点。当事件发射（fired）时，浏览器会给我们目标Dom节点。为了通过DOM层级传播事件，React并没有在虚拟节点层迭代。

每一个React组件具有唯一的ID层级编码。我们能用简单地字符串操作来获取所有家长的的ID。通过储存事件监听的hash表，可以发现比将它们附加在虚拟节点表现的更加好。下面是一个将事件委派给虚拟DOM的例子。
    
    // dispatchEvent('click', 'a.b.c', event) clickCaptureListeners['a'](event); 		clickCaptureListeners['a.b'](event); clickCaptureListeners['a.b.c'](event); 		clickBubbleListeners['a.b.c'](event); clickBubbleListeners['a.b'](event); 		clickBubbleListeners['a'](event);

浏览器给每一个时间和监听者都创建一个新的事件对象。这具有很好的特性，你可以保存事件对象的引用，甚至可以修改它。然而，这意味着大量的内存消耗。React在启动时为这些对象分配内存池。当需要一个对象时，它从该池中重复使用，这大大降低了垃圾回收。

### 渲染

#### 批处理     
无论何时你在组件中调用`setState`方法，React都将标记它为`脏`（dirty）.在事件处理最末，React将会把所有脏标记的组件重新渲染。

批处理意味着在事件周期中，有DOM被恰好被更新一次。这个属性是编写高质量应用的关键，而且现在很难简单的用javascript来写编写类似功能。而在React，这已经被默认定义了。

![img4](http://img.haoqiao.me//t-1-4.png)

### 子树渲染
当`setState`被调用，组件重构子组件的虚拟DOM。如果你在根调用`setState`，整个组件将被重新渲染。所有的组件，即使它们没有改变，都会有它们自己的`Render`.这听起来很吓人，很低效，但是我们并没有接触真实的DOM。

首先，我们讨论用户界面的呈现。因为屏幕大小有限，你需要经常同时呈现成千上百个元素。Javascript足够快能够承担业务逻辑使界面为可控的。

另一个关键点是写React代码，你经常调用setState在根节点改变时。你在组件中调用它然后获得改变事件或者在多个组件之上调用。你很少能够直达到顶端。这意味着变化被定位在用户交互中。

![img5](http://img.haoqiao.me//t-1-5.png)

### 选择子树渲染

最后，你可能需要防止某些子树被渲染。如果你执行下面的方法：

    boolean shouldComponentUpdate(object nextProps, object nextState)

在组件中设定下一个props/state之前，你可以告诉React这个组件将不会变化而且没有必要重新渲染它。当被恰当的实施，它确实可以给你带来巨大的性能提升。

为了能够使用它，你需要能够比较javascript对象，因为浅拷贝会引起很多问题，你需要深拷贝或者用一直不变的数据结构。

如果你想要保持这个方法能被在任何时候调用，因此你需要确保计算时间少于渲染时间。即使子渲染不是非常严格的需要。

![img6](http://img.haoqiao.me//1-1-6.png)

## 结论

使React变得快的技术并不是最新的。我们都知道长时间接触DOM的代价是昂贵的，你需要批量写入和读取操作，事件委托更快。。。

人们依然在讨论它们因为在实际中，他们很难去实施常规的Javascript代码，正是这些都在React是默认定义的才使React脱颖而出。这使得它很难搬起石头砸自己的脚让你的应用变慢。

## 译者言

这是第一次翻译，花了大概半天的时间，把看到的翻译成中文还是蛮困难的，有些东西一看就意会，然后转成语言就很奇怪。这次算是练手，对着原文翻译，这是蛮差劲的。应该是用自己的话表达出来而且加入自己的看法这是我所追求。继续努力吧。
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      
      