---
layout: post
title: 浅谈js动态生成html以及template与shadow Dom
tags: [学习,私货,前端,环境]
keywords: Javascript,代码,动态生成,html,前端,学习总结
description: 
---

## 前言

这篇是为了填之前的坑。

## 正文

web组件化是过去两年中提及非常多的。而它的衍生插件化是更古老的故事。对于现在不管写什么插件都要养成html,js合一的做法逐渐变成了插件的一个隐形规定。在之前写的日历插件中,为了方便并没有将html动态生成出,而是直接写在页面里,这带来了一些不好的后果。因此这章节我们将来初步接触一下相关技术。

首先，看到很多插件的写法是这样的:

```

var selectYear = document.querySelector(".selectYear");
for (var i = that.Config.minYear; i <= that.Config.maxYear; i++) {
	var li = document.createElement("li");
	var a = document.createElement("a");
	a.innerHTML = i;
	li.appendChild(a);
	selectYear.appendChild(li);
}
                
```

这种是先将html写好，然后转为js dom节点，用js的`createElement`方法来动态创建节点然后添加到页面里。这种写法的缺点在于你必须对html层级很清楚，然后写的时候要注意父子节点的关系，在很久之前我学习正则表达式的时候写过一个动态生成的工具，但是它仅仅只是判断父子节点的关系，而且还需要按照指定格式来编写。当然喜欢这种方式的童鞋可以不用再自己手动去写了，`github` 上有个开源的项目 [html2js](https://github.com/ArnonEilat/HTML2JS)已经帮我们可以实现这种形式。

![imgn](http://img.haoqiao.me/html2js1.png)

然后关于动态生成还有一种方法，我们拿我之前写的日历控件来看:

```

    //生成开始的内容
            initContent:function(){
                var content = ['    <div class="datePicker">',
                    '        <div class="datePicker-header">',
                    '            <b class="datePicker-pre"></b>',
                    '            <b class="datePicker-next"></b>',
                    '            <div class="datePicker-select yearPicker">',
                    '                <div class="date-hd">',
                    '                    <b class="date-icon"></b>',
                    '                    <span class="currYear">2016</span>',
                    '                </div>',
                    '                <div class="date-bd">',
                    '                    <ul class="selectYear">',
                    '                    </ul>',
                    '                </div>',
                    '            </div>',
                    '            <div class="datePicker-select mouthPicker">',
                    '                <div class="date-hd">',
                    '                    <b class="date-icon"></b>',
                    '                    <span class="currMouth">5</span>',
                    '                </div>',
                    '                <div class="date-bd">',
                    '                    <ul class="selectMouth">',
                    '                        <li><a>1</a></li>',
                    '                        <li><a>2</a></li>',
                    '                        <li><a>3</a></li>',
                    '                        <li><a>4</a></li>',
                    '                        <li><a>5</a></li>',
                    '                        <li><a>6</a></li>',
                    '                        <li><a>7</a></li>',
                    '                        <li><a>8</a></li>',
                    '                        <li><a>9</a></li>',
                    '                        <li><a>10</a></li>',
                    '                        <li><a>11</a></li>',
                    '                        <li><a>12</a></li>',
                    '                    </ul>',
                    '                </div>',
                    '            </div>',
                    '        </div>',
                    '        <div class="datePicker-title">',
                    '            <ul>',
                    '                <li>日</li>',
                    '                <li>一</li>',
                    '                <li>二</li>',
                    '                <li>三</li>',
                    '                <li>四</li>',
                    '                <li>五</li>',
                    '                <li>六</li>',
                    '            </ul>',
                    '        </div>',
                    '        <div class="datePicker-body">',
                    '            <ul class="clearfix dateContent"></ul>',
                    '            <div class="multiContent">',
                    '                <div class="multiOk">确定</div>',
                    '                <div class="multiCancel">取消</div>',
                    '            </div>',
                    '        </div>',
                    '    </div>'].join("");
                    var div = document.createElement("div");
                    div.innerHTML = content;

                    document.body.appendChild(div);
            },
            
```

我们可以从代码中看到我们将整个html代码用一个数组保存，里面每行都用单引号。这种方式已经很接近template了，因为我需要的是整个静态页面而不需要动态修改里面的内容因此不需要再加个解析引擎，直接动态创建一个div节点把内容填充进去即可。
当然这种方式看起来蛮清楚，但是如果要自己手打好像太累了，好在我们也有工具来实现类似功能，当然我感觉这种工具自己写也蛮方便的。。。

![imgn](http://img.haoqiao.me/html2js2.png)

工具戳[这里](http://www.css88.com/tool/html2js/)

接下来我们谈谈shadom dom，之前我看书的时候看到这个概念，然后我在思考在自己的插件中该如何使用，再后来考虑到我现在写的是单js插件，所有html代码需要放在js文件里面，这样对于使用shadom已经意义不大。因为shadom dom配合html5的template标签来实现。

我们来看个简单的例子：

```
html:

<div id="nameTag">Bob</div>
<template id="nameTagTemplate">
<style>
.outer {
  border: 2px solid brown;
  border-radius: 1em;
  background: red;
  font-size: 20pt;
  width: 12em;
  height: 7em;
  text-align: center;
}
.boilerplate {
  color: white;
  font-family: sans-serif;
  padding: 0.5em;
}
.name {
  color: black;
  background: white;
  font-family: "Marker Felt", cursive;
  font-size: 45pt;
  padding-top: 0.2em;
}
</style>
<div class="outer">
  <div class="boilerplate">
    Hi! My name is
  </div>
  <div class="name">
    <content></content>
  </div>
</div>
</template>


```

```
js:

var shadow = document.querySelector('#nameTag').createShadowRoot();
var template = document.querySelector('#nameTagTemplate');
var clone = document.importNode(template.content, true);
shadow.appendChild(clone);
document.querySelector('#nameTag').textContent = 'linshui';

```

![imgn](http://img.haoqiao.me/html2js3.png)

当然这种写法感觉上好像已经多此一举，因为你用各种mvvm框架都可以直接写组件比如vue。因此这个我们只需要看看就好了-0-

## 结尾

我标题是不是起的很到位，浅谈，分分钟看完。。。

