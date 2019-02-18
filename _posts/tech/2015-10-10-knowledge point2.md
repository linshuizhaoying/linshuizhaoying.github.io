---
layout: post
title: 跨域问题再研究
category: 技术
tags: [JS，基础,javascript,Css3,前端总结,基础]
keywords: 前端,资料,学习
description: 
---

## 前言
之前在查漏补缺第一波的是时候发现对这个知识点有点小疑惑，因为有几个列出的项之前并没怎么接触过，因此在这篇博文中将深入学习。

```
1.jsonp
2.iframe

```

### jsonp

对于jsonp格式应该都不陌生，我之前在印象笔记有记录，如下:

```
$.ajax({
  type: "GET",
  url: "http://127.0.0.1:8000/ajaxdemo/serverjsonp.php?number=" + $("#keyword").val(),
  dataType: "jsonp",
  jsonp: "callback",
  success: function(data) {
      if (data.success) {
          $("#searchResult").html(data.msg);
      } else {
          $("#searchResult").html("出现错误：" + data.msg);
      }
  },
  error: function(jqXHR){
     alert("发生错误：" + jqXHR.status);
  },
});

```

### xh2
这是后台需要在输出的时候处理:

```
header('Access-Control-Allow-Origin:*');
header('Access-Control-Allow-Methods:POST,GET');
header('Access-Control-Allow-Credentials:true');
header("Content-Type: application/json;charset=utf-8");
```

### iframe

iframe，使用iframe其实相当于开了一个新的网页，具体跨域的方法大致是，域A打开的母页面嵌套一个指向域B的iframe，然后提交数据，完成之后，B的服务端可以：

返回一个302重定向响应，把结果重新指回A域；
在此iframe内部再嵌套一个指向A域的iframe。

这两者都最终实现了跨域的调用，这个方法功能上要比下面介绍到的JSONP更强，因为跨域完毕之后DOM操作和互相之间的JavaScript调用都是没有问题的，但是也有一些限制，比如结果要以URL参数传递，这就意味着在结果数据量很大的时候需要分割传递，甚是麻烦；还有一个麻烦是iframe本身带来的，母页面和iframe本身的交互本身就有安全性限制。

方案:

```
iframe跨域问题有点多，必须要得到iframe内部页面的配合才可能通信，方法也比较多：
1，假写hash值通信，父子页面各自建立轮询去检测iframe中url的hash值，通过值来通信
2，利用HTML5的postMessage，不过注意这个也是异步的
3，利用IE6\7中对navigator的bug，在ie6/7中，父子页面使用的window.navigator是同一个东西，父页面改了，子页面也会跟着变；
4，iframe中嵌套一层与顶层页面同域的页面，比如a中套b，b中套c，其中a、c同域，b做出改变后通过url给c传值，c中操作top对象也就是a，由于同域，所以不会有问题

```

例子

```
window.TUI = window.$ = {};
/**
* @public 通过iframe异步请求数据
* @param {string}  url是请求的地址
* @param {function}  cb是处理返回数据的回调函数
*/
TUI.getIframeData = function(url, cb){
    var f = document.getElementById('crossdomain');
    if(f)
        f.src = url;
    else{
        var t = document.createElement("DIV");
        t.innerHTML = '<iframe id="crossdomain" width="0" height="0" style="visibility:hidden;" src="' + url + '" ></iframe>';
        document.body.appendChild(t.firstChild);
    }

    (function(){
    try{
       cb(document.getElementById('crossdomain').contentWindow.document.body.getElementsByTagName("TEXTAREA")[0].value);
    }catch(e){
        setTimeout(arguments.callee,0);
        return;
    }   
    })();
};

//像这样执行
$.getIframeData("http://yoursite.com/request_url/", function(data){
    /* do something */
});

```

### window.name

window.name 跨域是一个巧妙的解决方案，一般情况下，我们使用 window.name 的情况如下：

```
使用window.frames[windowName]得到一个子窗口
将其设置为链接元素的target属性
加载任何页面 window.name 的值始终保持不变。由于 window.name 这个显著的特点，使其适用于在不同源之间进行跨域通信

```

具体方案

```
当页面 A 想要从另一个源获取资源或 Web 服务，首先在自己的页面上创建一个隐藏的 iframe B，将 B 指向外部资源或服务，B 加载完成之后，将把响应的数据附加到 window.name 上。由于现在 A 和 B 还不同源，A 依旧不能获取到 B 的 name 属性。当B 获取到数据之后，再将页面导航到任何一个与 A 同源的页面，这时 A 就可以直接获取到 B 的 name 属性值。当 A 获取到数据之后，就可以随时删掉 B。

```

```
主页面代码：

function sendMsg(msg) {  
  var state = 0, data;
  var frame = document.createElement("frame");
  var baseProxy = "http://www.otherapp.com:3000/demo4-req";
  var request = {data:msg};
  frame.src = baseProxy + "#" + encodeURI(JSON.stringify(request));
  frame.style.display = "none";
  frame.onload  = function(){
      if(state === 1){
       data = frame.contentWindow.name;
       document.querySelector('.res').innerHTML = "获得响应：" + data;
       //删除iframe
       frame.contentWindow.document.write('');
       frame.contentWindow.close();
       document.body.removeChild(frame);
      } else {
          state = 1;
          frame.src = "http://www.myapp.com:3000/demo4-res";
      }
  };
  document.body.appendChild(frame);
}

document.querySelector('button').onclick = function (){  
    var val = Math.random();
    sendMsg(val);
    document.querySelector('.val').innerHTML = "请求数据："+val;
}

目标页面代码：

var hash = window.location.hash;  
if(hash && hash.length>1){  
   var request = hash.substring(1, hash.length);
   var obj = JSON.parse(decodeURI(request));
   var data = obj.data;
   //数据乘以100
   window.name = data*100;
}

```

### postMessage

HTML5 规范中的新方法 window.postMessage(message, targetOrigin) 可以用于安全跨域通信。当该方法被调用时，将分发一个消息事件，如果窗口监听了相应的消息，窗口就可以获取到消息和消息来源。

具体方案:

```
如果 iframe 想要通知不同源的父窗口它已经加载完成，可以使用 window.postMessage 来发送消息。同时，它也将监听回馈消息：
function postMessage(msg){  
  var targetWindow = parent.window;
  targetWindow.postMessage(msg,"*");
}
function handleReceive(msg){  
 if(msg.data == "ok"){
  //要做的事在这
 }else{
  //重新发送消息
  postMessage(JSON.stringify({color:'red'}));
 }
}
window.addEventListener("message", handleReceive, false);  
window.onload = function(){  
  postMessage(JSON.stringify({color:'red'}));
}


父窗口监听了消息事件，当消息到达时，它首先检查消息是否是来 www.otherapp.com，如果是就发送一个反馈消息。

function handleReceive(event){  
  if(event.origin != "http://www.otherapp.com:3000") return;
  //处理数据
  var data = JSON.parse(event.data);
  document.querySelector('div').innerHTML = "来自iframe的颜色："+data.color;

  var otherAppFrame = window.frames["otherApp"]
  otherAppFrame.postMessage("ok","http://www.otherapp.com:3000");
}
window.addEventListener("message", handleReceive, false);  
```

## 结尾

事实上我一般跨域遇到的都是自己测试数据的时候ajax拿不到数据，后来自从接触Mock,便很少遇到这些情况。iframe跨域可以说是最复杂也最多变的。它能依赖旧特性来达到跨域也能结合Html5新api来跨域。

今天就到这里。

