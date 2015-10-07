---
layout: post
title: 前端查漏补缺计划第一波
category: 技术
tags: [JS，基础,javascript,Css3,前端总结,基础]
keywords: 前端,资料,学习
description: 
---
 
## 前言
最近屯了不少干货，有不少是面试题，还有一些是基础补充。今天整理一下。

大概应该是这样的结构

```
1. this详解
2. 有价值的面试题
3. 其他基础补充
```
## 正文

### this详解

首先，this是执行上下文的一个属性。

this值在进入上下文时确定，并且在上下文运行期间永久不变。

```
// 显示定义全局对象的属性
this.a = 10; // global.a = 10
alert(a); // 10

// 通过赋值给一个无标示符隐式
b = 20;
alert(this.b); // 20

// 也是通过变量声明隐式声明的
// 因为全局上下文的变量对象是全局对象自身
var c = 30;
alert(this.c); // 30

```

在代码运行时的this值是不变的，也就是说，因为它不是一个变量，就不可能为其分配一个新值.

了解了this的概念我们来看下随便写一个片段。比如:

```
var linshui = 621;
function test(){
  var linshui = 126;
  var b = {};
  b = {linshui:216};
  alert(this.linshui);
}
test();
```

这写的比较基础，可能一眼就看来答案。621。this.linshui的内容还是全局变量linshui的值。

再比如:

```

var x = 1;
var b = {x :233};
var test = {
  x : 2,
  z:function(){
   console.log( this === test ); //#1
   
   console.log(this.x);//#2
   

   alert(this.x);   // #3
  }
};

test.z();

b.z = test.z;
b.z(); //#5

```

按需执行下来，#1是true,因为此时this指向test,#2为2，因为此时this指向的x是test里面的x。
 #3也是2,事实上alert与console无什么区别。
 
 #5 分别是 false 233.因为此时z函数执行过程中指向的this已经改为b了。
 
 ```
 在通常的函数调用中，this是由激活上下文代码的调用者来提供的，即调用函数的父上下文(parent context )。this取决于调用函数的方式。
 ```
 
继续举例子

```
var foo = {
bar: function () {
console.log(this);
}
};

foo.bar(); 
var test = foo.bar;
test();

```
foo.bar() 时指向obejct{}，因为foo本身就是个空的对象，那么test()时输出什么？

答案是windows,问题来了,test不是已经指向为foo.bar了么？

原因是因为test作为标示符，生成了引用类型的其他值，其base（全局对象）用作this 值。
（PS:标识符其实就是变量名、函数名、函数参数名以及全局对象的未受限的属性。）
(再ps: `foo.say(); ` `foo['say']();` 两种是一样的)
为什么用表达式的不同形式激活同一个函数会不同的this值，答案在于引用类型（type Reference）不同的中间值。

在匿名函数中,this的调用:

```
(function () { alert(this); // null => global })();

```

由于括号()左边的匿名函数是非引用类型对象（它既不是标识符也不属于属性访问），因此，this的值设置为全局对象。

举个例子:

```
var foo = { bar: function () { alert(this); } };
 console.log((foo.bar)()); // obejct obejct. 
 console.log((foo.bar = foo.bar)()); // obejct windows 与表达式一组操作符不同的是，它会触发调用GetValue方法。 最后返回的时候就是一个函数对象了（而不是引用类型的值了），这就意味着this的值会设置为null，最终会变成全局对象。
 
 console.log( (false || foo.bar)()); // obejct windows
 console.log( (foo.bar, foo.bar)()); // obejct windows
 
```

还有例外的情况，当调用表达式左侧是引用类型的值，但是this的值却是null，最终变为全局对象（global object）。 发生这种情况的条件是当引用类型值的base对象恰好为活跃对象（activation object）。比如:

```
function foo() { 
  function bar() { 
    alert(this); // global 
  } 
  bar();
}
```

由于活跃对象（activation object）总是会返回this值为——null,当内部子函数在父函数中被调用的时候就会发生这种情况.

### 有价值的面试题

1.页面导入样式时，使用link和@import有什么区别？

```
	（1）link属于XHTML标签，除了加载CSS外，还能用于定义RSS, 定义rel连接属性等作用；而@import是CSS提供的，只能用于加载CSS;
	
	（2）页面被加载的时，link会同时被加载，而@import引用的CSS会等到页面被加载完再加载;
	
	（3）import是CSS2.1 提出的，只在IE5以上才能被识别，而link是XHTML标签，无兼容问题;
	
```

2.html5有哪些新特性、移除了那些元素？如何处理HTML5新标签的浏览器兼容问题？如何区分 HTML 和 HTML5？

```
HTML5 现在已经不是 SGML 的子集，主要是关于图像，位置，存储，多任务等功能的增加。
      绘画 canvas;
      用于媒介回放的 video 和 audio 元素;
      本地离线存储 localStorage 长期存储数据，浏览器关闭后数据不丢失;
      sessionStorage 的数据在浏览器关闭后自动删除;
      语意化更好的内容元素，比如 article、footer、header、nav、section;
      表单控件，calendar、date、time、email、url、search;
      新的技术webworker, websockt, Geolocation;

  移除的元素：

      纯表现的元素：basefont，big，center，font, s，strike，tt，u;
      对可用性产生负面影响的元素：frame，frameset，noframes；

* 支持HTML5新标签：
     IE8/IE7/IE6支持通过document.createElement方法产生的标签，
     可以利用这一特性让这些浏览器支持HTML5新标签，
     浏览器支持新标签后，还需要添加标签默认的样式。

     当然最好的方式是直接使用成熟的框架、使用最多的是html5shim框架
     <!--[if lt IE 9]>
        <script> src="http://html5shim.googlecode.com/svn/trunk/html5.js"</script>
     <![endif]-->

* 如何区分： DOCTYPE声明\新增的结构元素\功能元素
```

3.HTML5的离线储存怎么使用，工作原理能不能解释一下？

```
原理：HTML5的离线存储是基于一个新建的.appcache文件的缓存机制(不是存储技术)，通过这个文件上的解析清单离线存储资源，这些资源就会像cookie一样被存储了下来。之后当网络在处于离线状态下时，浏览器会通过被离线存储的数据进行页面展示。


如何使用：
1、页面头部像下面一样加入一个manifest的属性；
2、在cache.manifest文件的编写离线存储的资源；
    CACHE MANIFEST
    #v0.11
    CACHE:
    js/app.js
    css/style.css
    NETWORK:
    resourse/logo.png
    FALLBACK:
    / /offline.html
3、在离线状态时，操作window.applicationCache进行需求实现。
```

4.请描述一下 cookies，sessionStorage 和 localStorage 的区别？

```
cookies 可选择过期时间
localStorage    长期存储数据，浏览器关闭后数据不丢失；
sessionStorage  数据在浏览器关闭后自动删除。

cookie在浏览器和服务器间来回传递。 sessionStorage和localStorage不会
sessionStorage和localStorage的存储空间更大；
sessionStorage和localStorage有更多丰富易用的接口；
sessionStorage和localStorage各自独立的存储空间；

```

5.iframe有那些缺点？

```
*iframe会阻塞主页面的Onload事件；

*iframe和主页面共享连接池，而浏览器对相同域的连接有限制，所以会影响页面的并行加载。

使用iframe之前需要考虑这两个缺点。如果需要使用iframe，最好是通过javascript
动态给iframe添加src属性值，这样可以可以绕开以上两个问题。

```

6.Label的作用是什么？是怎么用的？

```
label标签来定义表单控制间的关系,当用户选择(点击)该标签时，浏览器会自动将焦点转到和标签相关的表单控件上。

<label for="Name">Number:</label> 
<input type=“text“name="Name" id="Name"/> 

<label>Date:<input type="text" name="B" /></label>
```		

7.介绍一下CSS的盒子模型？

```
（1）有两种， IE 盒子模型、标准 W3C 盒子模型；IE的content部分包含了 border 和 pading;

（2）盒模型： 内容(content)、填充(padding)、边界(margin)、 边框(border).
```

8.如何居中div？

```
给div设置一个宽度，然后添加margin:0 auto属性

div{
    width:200px;
    margin:0 auto;
 }

```

9.列出display的值，说明他们的作用。position的值， relative和absolute定位原点是？

```
 1.
  block 象块类型元素一样显示。
  none 缺省值。象行内元素类型一样显示。
  inline-block 象行内元素一样显示，但其内容象块类型元素一样显示。
  list-item 象块类型元素一样显示，并添加样式列表标记。

  2.
  *absolute
        生成绝对定位的元素，相对于 static 定位以外的第一个父元素进行定位。

  *fixed （老IE不支持）
        生成绝对定位的元素，相对于浏览器窗口进行定位。

  *relative
        生成相对定位的元素，相对于其正常位置进行定位。

  * static  默认值。没有定位，元素出现在正常的流中
  *（忽略 top, bottom, left, right z-index 声明）。

  * inherit 规定从父元素继承 position 属性的值。
```

10.常见兼容性问题？

```
png24位的图片在iE6浏览器上出现背景，解决方案是做成PNG8.

* 浏览器默认的margin和padding不同。解决方案是加一个全局的*{margin:0;padding:0;}来统一。

超链接访问过后hover样式就不出现了 被点击访问过的超链接样式不在具有hover和active了解决方法是改变CSS属性的排列顺序:
L-V-H-A :  a:link {} a:visited {} a:hover {} a:active {}
```

11.对BFC规范的理解？

```
（W3C CSS 2.1 规范中的一个概念,它决定了元素如何对其内容进行定位,以及与其他元素的关 系和相互作用。）
```

12.css定义的权重

```
以下是权重的规则：标签的权重为1，class的权重为10，id的权重为100，以下例子是演示各种定义的权重值：

/*权重为1*/
div{
}
/*权重为10*/
.class1{
}
/*权重为100*/
#id1{
}
/*权重为100+1=101*/
#id1 div{
}
/*权重为10+1=11*/
.class1 div{
}
/*权重为10+10+1=21*/
.class1 .class2 div{
}

```

13.如果需要手动写动画，你认为最小时间间隔是多久，为什么？

```
多数显示器默认频率是60Hz，即1秒刷新60次，所以理论上最小间隔为1/60＊1000ms ＝ 16.7ms
```

14.display:inline-block 什么时候会显示间隙？

```
移除空格、使用margin负值、使用font-size:0、letter-spacing、word-spacing
```

15.写一个通用的事件侦听器函数。

```
    // event(事件)工具集，来源：github.com/markyun
    markyun.Event = {
        // 页面加载完成后
        readyEvent : function(fn) {
            if (fn==null) {
                fn=document;
            }
            var oldonload = window.onload;
            if (typeof window.onload != 'function') {
                window.onload = fn;
            } else {
                window.onload = function() {
                    oldonload();
                    fn();
                };
            }
        },
        // 视能力分别使用dom0||dom2||IE方式 来绑定事件
        // 参数： 操作的元素,事件名称 ,事件处理程序
        addEvent : function(element, type, handler) {
            if (element.addEventListener) {
                //事件类型、需要执行的函数、是否捕捉
                element.addEventListener(type, handler, false);
            } else if (element.attachEvent) {
                element.attachEvent('on' + type, function() {
                    handler.call(element);
                });
            } else {
                element['on' + type] = handler;
            }
        },
        // 移除事件
        removeEvent : function(element, type, handler) {
            if (element.removeEventListener) {
                element.removeEventListener(type, handler, false);
            } else if (element.datachEvent) {
                element.detachEvent('on' + type, handler);
            } else {
                element['on' + type] = null;
            }
        },
        // 阻止事件 (主要是事件冒泡，因为IE不支持事件捕获)
        stopPropagation : function(ev) {
            if (ev.stopPropagation) {
                ev.stopPropagation();
            } else {
                ev.cancelBubble = true;
            }
        },
        // 取消事件的默认行为
        preventDefault : function(event) {
            if (event.preventDefault) {
                event.preventDefault();
            } else {
                event.returnValue = false;
            }
        },
        // 获取事件目标
        getTarget : function(event) {
            return event.target || event.srcElement;
        },
        // 获取event对象的引用，取到事件的所有信息，确保随时能使用event；
        getEvent : function(e) {
            var ev = e || window.event;
            if (!ev) {
                var c = this.getEvent.caller;
                while (c) {
                    ev = c.arguments[0];
                    if (ev && Event == ev.constructor) {
                        break;
                    }
                    c = c.caller;
                }
            }
            return ev;
        }
    };

```

16.js的基本数据类型

```
number,string,boolean,object,undefined
```

17.谈谈This对象的理解。

```
this是js的一个关键字，随着函数使用场合不同，this的值会发生变化。

但是有一个总原则，那就是this指的是调用函数的那个对象。

this一般情况下：是全局对象Global。 作为方法调用，那么this就是指这个对象
```

18.如何解决跨域问题?

```
jsonp、 iframe、window.name、window.postMessage、服务器上设置代理页面
```

19.模块化怎么做？

```
立即执行函数,不暴露私有成员

    var module1 = (function(){
    　　　　var _count = 0;
    　　　　var m1 = function(){
    　　　　　　//...
    　　　　};
    　　　　var m2 = function(){
    　　　　　　//...
    　　　　};
    　　　　return {
    　　　　　　m1 : m1,
    　　　　　　m2 : m2
    　　　　};
    　　})();
    　　
```

### 基础补充
发现内容有点多，等下一波再添加进去。今天就到这里-。-

