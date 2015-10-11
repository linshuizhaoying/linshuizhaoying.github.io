---
layout: post
title: modal 模态层的两种写法
category: 技术
tags: [JS，基础,javascript,Css3,前端总结,基础]
keywords: 前端,资料,学习
description: 
---

## 前言
今天有点小困，回寝室有点迟，来研究一下比较简单的两种模态写法就去睡觉(～﹃～)~zZ。

```
1.jquery插件
2.vue组件

```

## Jquery插件
将模拟层写成jquery插件是非常适合的，毕竟这类通用的自己抽象出来可以随时调用。

例子放在了[jsfiddle](http://jsfiddle.net/qcj6tt6h/)

来看下如何写:

```
(function($){
    $.fn.modalInfowindow = function(options){
     ...
    }
})(jQuery);

```
首先是jquery插件起手式.以这样的插件写法起手，你可以用下面这种方式调用。

```

function ShowModal(){
  $('body').modalInfowindow({
  ....
  });
}

```

之后就是填充内容:

```
var defaults = {};
        var options = $.extend(defaults, options); //参数设置
        var container=$(this); //引用调用插件的对象
        var width=options.width, height=options.height, title=options.title, content=options.content; //设定提醒框内容高宽
        
        //模态层要全屏撑满，因此动态设置百分比
        
        //模态层容器
        var modal=$("<div id='modal'></div>");
        modal.css("width","100%");
        modal.css("height","100%");
        //模态层
        var modal_div=$("<div class='modal'></div>");
        modal_div.css("width","100%");
        modal_div.css("height","100%");
        
        //信息框其实是随便定制的，因此，你可以自己定制自己的信息框
        
        //信息框
        var infoWin=$("<div class='infowin'></div>");
        infoWin.css("width",width+"px");
        infoWin.css("height",height+"px");
        infoWin.css("position","absolute");
        infoWin.css("top",(container.height()-height)/2+"px");
        infoWin.css("left",(container.width()-width)/2+"px");
        //标题
        var infoWin_title=$("<div class='title'></div>");
        
        //对关闭按钮绑定事件，关闭hide()等于display:none
        var infoWin_title_close=$("<div class='close'></div>")
        infoWin_title_close.on("click",function(){
            console.log("Close Modal!");
            modal.hide();
        });
        
        //添加内容生成信息框追加
        var infoWin_title_content=$("<div class='title_content'></div>")
        infoWin_title_content.append(title);
        infoWin_title.append(infoWin_title_close);
        infoWin_title.append(infoWin_title_content);
        
        
        //内容
        var infoWin_content=$("<div class='content'></div>");
        infoWin_content.append(content);
        //信息框添加标题和内容
        infoWin.append(infoWin_title);
        infoWin.append(infoWin_content );
        //模态层容器添加模态层和信息框
        modal.append(modal_div);
        modal.append(infoWin);
        //将模态层添加到容器
        container.append(modal);
```

然后你会发现，自己写一个模态层jquery插件是如此简单-0-.

接下来是对应的css样式:

```
html,body{
    font-size: 12px;
    font-family: "微软雅黑";
}
.modal{
    position: absolute; 
    top:0px;
    left: 0px;
    border: 1px solid #000;
    background: #555;
    opacity: 0.4;
}
.infowin{
    border: 1px solid #777777;
    background: #fff;
    box-shadow: 0 0 0.75em #777777;
    -moz-box-shadow: 0 0 0.75em #777777;
    -webkit-box-shadow: 0 0 0.75em #777777;
    -o-box-shadow: 0 0 0.75em #777777;
    border-radius: 5px;
    -moz-border-radius: 5px;
    -webkit-border-radius: 5px;
    -o-border-radius: 5px;
}
 .title{
    border-bottom: 1px solid #777777;
}
.title_content{
    padding: 5px;
    padding-left: 10px;
    font-size: 14px;
    font-family: "微软雅黑";
    font-weight: bold;
}
.close{
    background: url(close.png) no-repeat; //这边随意
    width: 25px;
    height: 25px;
    float: right;
}
.close:hover{
    cursor: pointer;
}
.content{
    padding-left: 10px;
    padding-top: 10px;
}

```
正式调用只要以如下格式就可以

```

function ShowModal(){
  $('body').modalInfowindow({
      width:600,
      height:500,
      title:"Linshuizhaoying",
      content:"临水照影"
  });
}

```

### Vue版modal

直接做成组件:

```
<template>
  <div class="overlay">
    <div class="overlay-mask" v-on="click: show=false"></div>
    <div class="overlay-inner">
      <content></content>
    </div>
  </div>
</template>

<script>
  module.exports = {
    props: ['show']
  };
</script>

<style>
  .overlay {
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    z-index: 9;
    height: 100%;
    width: 100%;
    background-color: white;
    overflow-y: scroll;
    -webkit-overflow-scrolling: touch;
  }
  .overlay-mask {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    z-index: 10;
    cursor: pointer;
  }
  .overlay-mask:after {
    content: '×';
    position: absolute;
    top: 16px;
    right: 18px;
    color: #ccc;
    font: 500 24px/1 "Helvetica Neue", "Arial", sans-serif;
  }
  .overlay-inner {
    position: absolute;
    top: 0;
    left: 50%;
    width: 860px;
    height: 100%;
    margin-left: -430px;
    z-index: 11;
  }
</style>

```
看以看到模态层的显示是以传过来的参数show为主。

我们看下如何调用:

先引用组件:

```
  components: {
    'overlay': require('../components/overlay.vue'),
    'adminaccountedit': require('./adminaccountedit.vue'),
  },
```

然后定义一个showlayer的data：

```
data: function() {
      return {
        showlayer: false,
      }
    },
    
```

在调用组件中写如下:

```

    <overlay v-if="showlayer" show="{{@ showlayer }}">
      <adminaccountedit userid={{userid}} username={{username}} ></adminaccountedit>
		</overlay>

```

最后在组件周期中写事件

```
	that.showlayer = true;
	
```

在需要关闭模态层的时候这么写

```
<div class="return-btn" v-on="click:close">返回</div>

methods: {
  close:function(e){
  	$(".overlay-mask").click();
  }
}
```

好了，今天就到这里。

