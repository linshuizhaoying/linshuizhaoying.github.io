---
layout: post
title: Demystifying Closures, Callbacks and IIFEs (揭秘javascript的闭包，回调，IIFEs）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://iamvdo.me/en/blog/css-element-function)

### 作者: iamvdo

### 译者: 临水照影


## 正文之前

这是今天的翻译文-0-。。。我发现我有种突然想把[gsap](http://greensock.com/)的官方文档认真的翻译或者系统的写一些列学习文章（⊙﹏⊙其实是最近在练习一个demo，但是没有系统的学习总感觉不得劲。。）。不过自己已经作死的每日刷一个翻译日常，=-=也不知道哪天能开坑。


## 正文

### 闭包

在javascript中，一个闭包是任何函数保存对其父作用域中变量的引用，即使这个父类已经返回结束。

这意味着几乎所有函数都是一个闭包，因为在[变量作用域](http://www.sitepoint.com/demystifying-javascript-variable-scope-hoisting/#variable-scope)中我们明白了，一个函数能被引用，或者可以访问
    
    任何在它们自己的作用域中的变量和参数
    任何在外部（父）函数中的变量和参数
    任何在全局作用域中的变量

所以，你甚至不知道你已经在不经意间使用过闭包，但是我们不仅仅追求能够使用它，更要理解它。如果我们不知道它是如何工作的，我们无法完整的使用它。因此，我们把闭包定义成三个知识点。

### 知识点1：你可以引用当前函数的父函数中的变量

    function setLocation(city) {
      var country = "France"; 

      function printLocation() {       
        console.log("You are in " + city + ", " + country);  
      }

      printLocation();
    }

    setLocation ("Paris");  // output: You are in Paris, France
    
[在js bin中的例子](http://jsbin.com/sakiqepato/1/edit?js,console,output)

在这例子中，printLocation函数调用了包含它的setLocation函数的country变量

### 知识点2:内部函数可以引用定义外部函数中的变量，即使它们已经执行返回
    
    function setLocation(city) {
      var country = "France"; 

      function printLocation() {       
        console.log("You are in " + city + ", " + country);  
      }

      return printLocation;
    }

    var currentLocation = setLocation ("Paris");   

    currentLocation();   // output: You are in Paris, France
    
[在js bin中的例子](http://jsbin.com/vajihuwixu/edit?js,console,output)

### 知识点3:内部函数可以通过引用存储外部函数的变量，而不是保存值

    function cityLocation() {
      var city = "Paris";

      return {
        get: function() { console.log(city); },  
        set: function(newCity) { city = newCity; }
      };
    }

    var myLocation = cityLocation();

    myLocation.get();           // output: Paris
    myLocation.set('Sydney');
    myLocation.get();           // output: Sydney

[在js bin中的例子](http://jsbin.com/sakiqepato/3/edit?js,console,output)
    
这个例子说明闭包可以读和更新它们存储的变量，任何闭包都能够看到这个更新。这意味着闭包是引用它们外部函数的变量而不是拷贝它们的值。这是很重要的一点，因为我们等会会看到在立即执行函数表达式中看到一些复杂的逻辑问题。

闭包有个有趣的是特性就是它内部的函数都会自动隐藏。唯一改变内部存储的值方法只有间接修改。比如我们刚刚的例子。

## 回调

在javascript中，函数是第一类对象。函数可以作为参数传递或者作为返回值。

一个函数作为其它函数的参数或者返回值这叫做高阶函数。作为参数传递的函数叫做回调函数。
    
回调函数有很多常规用途。其中一项就是使用浏览器的windows对象中的setTimeout()和setInterval()。
    
    function showMessage(message){
      setTimeout(function(){
        alert(message);
      }, 3000);  
    }

    showMessage('Function called 3 seconds ago');
    
[在js bin中的例子](http://jsbin.com/sakiqepato/4/edit?js,console,output)

另一个例子就是在页面中对元素添加事件监听
     
    // HTML

    &lt;button id='btn'&gt;Click me&lt;/button&gt;

    // JavaScript

    function showMessage(){
      alert('Woohoo!');
    }

    var el = document.getElementById("btn");
    el.addEventListener("click", showMessage);    


[在js bin中的例子](http://jsbin.com/sakiqepato/5/edit?html,js,console,output)  
    
建立一个理解高级函数和回调函数的工作原理的例子是很简单的



    function fullName(firstName, lastName, callback){
      console.log("My name is " + firstName + " " + lastName);
      callback(lastName);
    }

    var greeting = function(ln){
      console.log('Welcome Mr. ' + ln);
    };
    
    
[在js bin中的例子](http://jsbin.com/sakiqepato/6/edit?js,console,output)

我们传送的是函数的定义而不是调用函数。这是为了防止函数立刻执行，只是在回调函数中需要注意的一点。回调函数就像一个运行中的闭包。他们能访问任何包含他们的外部函数的变量和参数，甚至从全局作用域中获取变量。

下面这个回调函数可以在上个例子中存在，或者它可以在任何匿名函数中。
    
    function fullName(firstName, lastName, callback){
      console.log("My name is " + firstName + " " + lastName);
      callback(lastName);
    }

    fullName("Jackie", "Chan", function(ln){console.log('Welcome Mr. ' + ln);});

[在js bin中的例子](http://jsbin.com/sakiqepato/7/edit?js,console,output)
    
回调在javascript库中提供管理和复用中非常重要。它们让库方法很容易定制和扩展。而且，代码更容易维护，更容易阅读。

让我们写两个函数，一个输出关于发布的文章的信息，另一个输出发送的信息。

    function publish(item, author, callback){   // Generic function with common data
      console.log(item);
      var date = new Date();

      callback(author, date);
    }

    function messages(author, time){   // Callback function with specific data
      var sendTime = time.toLocaleTimeString();
      console.log("Sent from " + author + " at " + sendTime);
    }

    function articles(author, date){   // Callback function with specific data
      var pubDate = date.toDateString();
      console.log("Written by " + author);
      console.log("Published " + pubDate);
    }

    publish("How are you?", "Monique", messages);

    publish("10 Tips for JavaScript Developers", "Jane Doe", articles);

[在js bin中的例子](http://jsbin.com/sakiqepato/8/edit?js,console,output)


我们把代码抽象然后复用，可以发现最后仅需要一个Publish函数即可搞定(译者PS：这已经有点策略模式的味道了。)



## 立即执行函数表达式

这是两种有点不同的写法：

    // variant 1

    (function () {
      alert('Woohoo!');
    })();

    // variant 2

    (function () {
      alert('Woohoo!');
    }());

将一个常规的的函数放入立即执行函数表达式你需要做两步：
   
    1.你需要将整个函数包含在一对括号中。立即执行函数表达式必须是一个表达式而不是函数定义。用一对括号是将一个函数定义转化为函数表达。
    2.你需要在末尾加一对括号，如变式1或者变式2.

还有三件事你需要记住：

首先，如果你将一个函数赋值给变量，你不需要将整个函数用括号包住.因为它已经是一个表达式了
    
    var sayWoohoo = function () {
      alert('Woohoo!');
     }();

第二，必须在末尾加分号，否则可能不会正常工作

最后，你可以传递参数给立即执行函数表达式，像这样：

    (function (name, profession) {
      console.log("My name is " + name + ". I'm an " + profession + ".");
    })("Jackie Chan", "actor");   // output: My name is Jackie Chan. I'm an actor.
    
[在js bin中的例子](http://jsbin.com/sakiqepato/9/edit?js,console,output)

传递一个全局对象给立即执行函数表达式是很常见的，因为它可以让函数内部不需要用window对象。这让代码在浏览器环境中更加独立。

    (function (global) {
      // access the global object via 'global'
    })(this);
    </code></pre>

    <p>This code will work both in the browser (where the global object is <code>window</    code>), or in a Node.js environment (where we refer to the global object with the special     variable <code>global</code>). </p>

    <p>One of the great benefits of an IIFE is that, when using it, you don’t have to worry about polluting the global space with temporary variables. All the variables you define inside an IIFE will be local. Let’s check this out:</p>

    [code language="javascript"](function(){

      var today = new Date();
      var currentTime = today.toLocaleTimeString();
      console.log(currentTime);   // output: the current local time (e.g. 7:08:52 PM)

    })();

    console.log(currentTime);   // output: undefined

[在js bin中的例子](http://jsbin.com/sakiqepato/10/edit?js,console,output)

在这个例子中，第一个console工作正常，第二个无法输出，因为调用的参数是一个立即执行函数表达式中的局部变量。

我们已经知道闭包保存与外部函数之间变量的引用，然后它们返回最新的值。你认为下列输出会是什么？
    
    function printFruits(fruits){
      for (var i = 0; i < fruits.length; i++) {
        setTimeout( function(){
          console.log( fruits[i] );
        }, i * 1000 );
      }
    }

    printFruits(["Lemon", "Orange", "Mango", "Banana"]);

[在js bin中的例子](http://jsbin.com/sakiqepato/11/edit?js,console,output)

最终结果都是Undefined，这是为什么呢？

首先这个函数迭代四遍，由于一开始我们fruits中什么值都没有，它输出undefined。然后当i < fruits.length时返回false。所以这个时候i等于4.这是最近版本中变量在函数中因为在循环中使用，闭包只链接变量本身而不是它的值。(译者PS:其实就是printFruits函数一开始定义的时候已经内部开始循环了，然后因为循环到最后i等于4，这时候你传递参数进来它是输出fruits[4]，注意因为数组是从0开始计数，所以你只能看到输出undfined.你可以输出i的值试试)

为了解决这个问题，我们需要提供一个新的作用域，让每个函数被循环创建，这样能追踪到i变量的状态。

    function printFruits(fruits){
      for (var i = 0; i &lt; fruits.length; i++) {
        (function(){
          var current = i;                    // define new variable that will hold the     current value of "i"
          setTimeout( function(){
            console.log( fruits[current] );   // this time the value of "current" will be different for each iteration
          }, current * 1000 );
        })();
      }
    }

    printFruits(["Lemon", "Orange", "Mango", "Banana"]);

[在js bin中的例子](http://jsbin.com/sakiqepato/12/edit?js,console,output)

当然我们可以用下面这种变体

    function printFruits(fruits){
      for (var i = 0; i &lt; fruits.length; i++) {
        (function(current){
          setTimeout( function(){
            console.log( fruits[current] );
          }, current * 1000 );
        })( i );
      }
    }

    printFruits(["Lemon", "Orange", "Mango", "Banana"]);

立即执行函数表达式通常用在封装模块。来防止被修改其中的内容。这衍生了一种技术：[模块模式](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)它被用在很多现代javascript的库中(举个例子：Jquery和underscore)

## 结尾

这个教程主要目的是为了将这些概念介绍清楚，简要的浓缩为一系列规则。理解它们是成为一个成功的优秀的javascript开发者的必要元素。

如果你想深入了解这些内容，可以点[这里](http://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)


## 译者
前面两个挺简单，第三块内容很有嚼头。建议去看作者推荐的深入内容。
 









