---
layout: post
title: 将input玩出一朵花
category: 技术
tags: [js,vue.js,css3,input,基础,积累]
keywords: 前端,资料,学习,javascript
description: 
---

## 前言
这几天比较闲-0-，然后有一天晚上我下班回去后台发来一个图片，是Input里面包含一个路径，然后跟我嗦"还可以这样，修改不了"。一眼瞄过去我知道大概是某个属性起作用。今天突然脑(闲)洞(的)大(要)开(死)，打算深入一下。

## 正文
首先来看这张图

![imgn](http://haoqiao.qiniudn.com/input01.png)

它默认有个路径但是无法修改只能复制。

在[mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input)中可以看到一个readonly就能解决。

也就是下面这一句话就能解决

```

<input type="text"  value="/test/linshuizhaoying" readonly="">

```

考虑一下应用场景应该是在文件管理，那么一开始这个value应该是自动获取来的参数，我们可以用vue模拟一下。

![imgn](http://haoqiao.qiniudn.com/active54.gif)

非常简单的用法,代码也很简单：

```
<template>
  <input type="text"  value="{{data}}" readonly="">
  <button v-on:click="change"> 改变路径</button>
</template>

<script>
module.exports = {
  data:{
    data:"/vue/linshuizhaoying"
  },
  ready:function(){

  },
  methods: {
    change: function (e) {
      this.data = "/change/linshuizhaoying";
    }
  },

}
</script>

```

然后闲的没事继续看，有个autofocus属性。

这是之前没注意到的,它可以在页面加载的时候自动把光标移到你指定的Input.
经过测试，如果多个Input设置autofocus="true",那么只有第一个会发光。

![imgn](http://haoqiao.qiniudn.com/input2.png)

这个时候我突然想到vue官方文档中的一个例子。它可以定义一个过滤器，在把来自视图（<input> 元素）的值写回模型之前转化它.

![imgn](http://haoqiao.qiniudn.com/active55.gif)

因为之前没怎么用过filter,因此花了点时间去熟悉，后来找到了使用方法。

我们先来看官方的代码

```
Vue.filter('currencyDisplay', {
  // model -> view
  // 在更新 `<input>` 元素之前格式化值
  read: function(val) {
    return '$'+val.toFixed(2)
  },
  // view -> model
  // 在写回数据之前格式化值
  write: function(val, oldVal) {
    var number = +val.replace(/[^\d.]/g, '')
    return isNaN(number) ? 0 : parseFloat(number.toFixed(2))
  }
})

```

但这里有个问题，它会报错。

`Uncaught TypeError: t.toFixed is not a function`

在[stackoverflow](http://stackoverflow.com/questions/14059201/why-does-firebug-say-tofixed-is-not-a-function)找到根源。

我们需要强制转换一下类型。因此，具体可运行代码如下：


```

<template>

<input type="text" v-model="data | datachange">


</template>

<script>
var Vue = require("vue")

Vue.filter('datachange', {
  // model -> view
  // 在更新 `<input>` 元素之前格式化值
  read: function(val) {
  		//记得强制转换下类型
    return '$'+parseFloat(val).toFixed(2)
  },
  // view -> model
  // 在写回数据之前格式化值
  write: function(val, oldVal) {
    var number = +val.replace(/[^\d.]/g, '')
    return isNaN(number) ? 0 : parseFloat(number.toFixed(2))
  }
})

module.exports = {
  data:{
    data:"123"
  },
  ready:function(){

  },
  methods: {
    change: function (e) {
      this.data = "/change/linshuizhaoying";
    }
  },
}
</script>
```

这个应用我们可以用到很多地方，比如输入一连串的数字，在某个位置自动用斜线把它分开。

根据上面的思路我们写个简单的demo来看下：

```
Vue.filter('datachange', {
  // model -> view
  // 在更新 `<input>` 元素之前格式化值
  read: function(val) {
    return val;
  },
  // view -> model
  // 在写回数据之前格式化值
  write: function(val, oldVal) {
    var position = 4;
    var insert = "-";
    var s = val.toString().replace(/-/g, "");
    if(val.length <= position){
      return val
    }else{
      return  [s.slice(0, position), insert, s.slice(position)].join('');
    }
    
  }
})

```

定义`var s = val.toString().replace(/-/g, "");`这句是为了过滤原本有的横线。这样不会重复添加横线。

然后看到这个效果，我突然又想到了一个东东。很早之前别人给我看的一个效果。是这样的：

![imgn](http://haoqiao.qiniudn.com/active56.gif)

当初看到这个效果蛮惊艳的，后来经过查找问人，是[formatter.js](https://github.com/firstopinion/formatter.js)这个插件。

粗略的看了一遍源码。在配合官方给出的例子.

大概实现思路就是给一个pattern，给Input赋初始值。然后监听事件，对各个按键进行处理。

虽然讲的很简单，但是源码也有近千行了，感兴趣的可以去阅读。

## 结尾
好久没写点东东，最近在看高程和其他几本书。也在不断学习。

