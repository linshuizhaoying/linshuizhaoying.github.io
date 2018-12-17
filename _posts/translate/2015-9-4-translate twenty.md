---
layout: post
title:  Can We Do Better Than The Click? (浏览器输入事件，比click更棒的方法）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://www.smashingmagazine.com/2015/03/better-browser-input-events/)

### 作者:  Dustan Kasten


### 译者: 临水照影


## 正文之前

这是一篇科普文，里面彩蛋很多，因此会把一些彩蛋也放到本文的译者Ps中。

## 正文

响应用户输入是我们作为界面开发者的核心。为了能够创建响应式产品，理解`touch` , `mouse` ,`pointer`,`keyboard`在浏览器种协同工作是关键。你可能看过[在移动端浏览器中300ms延迟](http://blog.ionic.io/hybrid-apps-and-the-curse-of-the-300ms-delay/)(译者ps：这种延迟的触发主要是当用户触摸屏幕时，浏览器触发touchstart,用户停止触摸时，浏览器触发touched，然后浏览器会delay300ms来确定用户是否准备再次触摸。如果用户不再触摸，浏览器会触发click事件。)或者[纠结touchmove还是scrolling](https://docs.google.com/document/d/12k_LL_Ot9GjF8zGWP9eI_3IMbSizD72susba0frg44Y/edit#)

在这篇文章中，我们将介绍事件级联并用这个方法来让一个轻敲事件的demo能够支持大部分输入方法。

## 概况

现在有三种输入方法被web所接受：数字游标（鼠标），触摸以及键盘输入。我们通过javascript的`touch events`, `mouse events`, `pointer events` 和 `keyboard events`来获取这种输入。在这篇文章中我们将主要关注`touch-`和`Mouse-based`的互动。虽然有些事件是以标准键盘为基础，比如`click`和`submit`。

你很可能很久之前就已经用事件处理`touch`和`mouse`事件。这可能是之前的一种写法:
    
    /** DO NOT EVER DO THIS! */
    $('a', ('ontouchstart' in window) ? 'touchend' : 'click', handler);


微软一马当先，创建了`Pointer Events`标准。现在是w3c推荐的指针事件标准。

## 事件级联

当用户在移动设备上轻触一个元素，浏览器会触发一系列事件。如下：
   
     touchstart → touchend → mouseover → mousemove → mousedown → mouseup → click.

由于web向后兼容的特性，指针事件可以用另一种方法。
    
    mousemove → pointerover → mouseover → pointerdown → mousedown → gotpointercapture → pointerup → mouseup → lostpointercapture → pointerout → mouseout → focus → click.
    

下列图展示了事件级联跟随以下动作：
    
    1.初始轻触一个元素
    2.第二次轻触一个元素
    3.轻触结束

请注意：这个事件栈是忽略`focus`和`blur`事件的。

![imgn](http://img.haoqiao.me//01-ios-opt-small.png)

这是ios事件级联

![imgn](http://img.haoqiao.me//02-android-opt-small.png)

这是android事件级联

![imgn](http://img.haoqiao.me//03-pointer-opt-small.png)

这是ie11的事件级联

## 运用事件级联

避免300毫秒的方法：
   
     对与chrome和android     <meta name="viewport" content="width=device-width">

如果我们的目的是为了创建一个与native platforms竞争的产品，我们需要根据基础事件(down,move,up)来创建我们自己的复合事件(click,double-click) 当然为了更广泛的的支持和易用性我们还需要包含`native events`的`fallback handlers`。

这需要长时间知识的积累。为了避免300ms延迟，我们需要自己掌控事件所有的生命周期。我们需要自己绑定所需的事件，在事件结束后自己清除绑定。

在下面的链接中，你会发现一个小小的无依赖tap的demo来说明建立一个多输入低延迟点击事件所需的工作量。

当然也有现成的解决方案比如

[fastclick](https://github.com/linshuizhaoying/fastclick)

[hammerjs](http://hammerjs.github.io/)

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/qy9fz10o/)


## 重点

绑定你自己的事件处理是一切的开始，下面的代码是处理多设备输入：
    
    /**
     * If there are pointer events, let the platform handle the input 
     * mechanism abstraction. If not, then it’s on you to handle 
     * between mouse and touch events.
     */

    if (hasPointer) {
      tappable.addEventListener(POINTER_DOWN, tapStart, false);
      clickable.addEventListener(POINTER_DOWN, clickStart, false);
    }

    else {
      tappable.addEventListener('mousedown', tapStart, false);
      clickable.addEventListener('mousedown', clickStart, false);

      if (hasTouch) {
        tappable.addEventListener('touchstart', tapStart, false);
        clickable.addEventListener('touchstart', clickStart, false);
      }
    }

    clickable.addEventListener('click', clickEnd, false);

绑定触发事件处理器可能会对渲染造成影响，即使你什么也不做。为了降低这个影响，在开始事件处理的时候绑定跟踪事件是一种推荐的做法。不要忘了在你动作结束后清除跟踪事件的绑定。

    /**
     * On tapStart we want to bind our move and end events to detect 
     * whether this is a “tap” action.
     * @param {Event} event the browser event object
     */

    function tapStart(event) {
      // bind tracking events. “bindEventsFor” is a helper that automatically 
      // binds the appropriate pointer, touch or mouse events based on our 
      // current event type. Additionally, it saves the event target to give 
      // us similar behavior to pointer events’ “setPointerCapture” method.

      bindEventsFor(event.type, event.target);
      if (typeof event.setPointerCapture === 'function') {
        event.currentTarget.setPointerCapture(event.pointerId);
      }

      // prevent the cascade
      event.preventDefault();
  
      // start our profiler to track time between events
      set(event, 'tapStart', Date.now());
    }

    /**
     * tapEnd. Our work here is done. Let’s clean up our tracking events.
     * @param {Element} target the html element
     * @param {Event} event the browser event object
     */

    function tapEnd(target, event) {
      unbindEventsFor(event.type, target);
      var _id = idFor(event);
      log('Tap', diff(get(_id, 'tapStart'), Date.now()));
      setTimeout(function() {
        delete events[_id];
      });
    }

剩下的代码应该不言自明。事实上，这个工作量很大。你可以用现成的解决方案。

## 译者

对于想深入的同学可以看原文的结尾。里面有作者引用的各种资料。对于现阶段我并没进行移动端开发因此我就不深入引用了。



















