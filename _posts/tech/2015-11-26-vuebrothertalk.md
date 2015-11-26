---
layout: post
title: vue兄弟组件通讯的实现
category: 技术
tags: [js,vue.js,css3,vue,组件,积累]
keywords: 前端,资料,学习,javascript
description: 
---

## 前言

最近把服务器换到阿里云，之前暂停的技能树也重新解析了。


## 正文

vue提供了很方便的父子组件通讯，基于Props我们可以做很多事情。在0.12版本中我是通过黑科技来实现兄弟组件的通讯。在1.0.8版本中，情况变的更加好。我们可以直接用官方的api的一些内容稍加组合就能实现。

因此我们来先讲明一下这里的`兄弟组件`指的是两个同级的组件需要同步一些相同的数据。

这里我将简述一个应用环境，具体代码不会给出，将把思路和关键代码阐述清楚。

首先是一个主页面app.vue

它包含两个子组件:

```
一个是checkbox.vue 它用来统计用户选择了多少个多选框，并显示.
一个是lists.vue 它里面还包含一个item.vue 用户的选择操作是在item.vue中定义的。数据也是在item.vue储存。单个Item.vue会将数据传给lists.vue。

```

目标是需要checkbox.vue能够获取item.vue里面每一个选项状况。

因为需要将item的数据传回lists。我们只需要在Item里的check操作中加入下列代码：

```

this.$dispatch('child-msg',"add",this.msg,this.id);

```

`msg和id 都是在props里面定义的。`


然后在Lists中监听child-msg事件：

```
events: {
    'child-msg': function (action,msg,no) {
      // 事件回调内的 `this` 自动绑定到注册它的实例上
      var that = this;
      if(action == "add"){
      	that.parentmsg.push(msg);  
      	that.parentno.push(no);
      }
      this.$parent.$data.alldata = that.parentmsg;
    }
  },
```

然后我们用$parent把数据传给app.vue

```
this.$parent.$data.alldata = that.parentmsg;

```

在app.vue里面:

```
  data:{
     alldata: []
  },
```

绕了一圈我们的数据到达了app.vue的data里面，然后我们直接给checkbox传过去:

```

<checkedbox v-bind:data=alldata></checkedbox>

```

记得用v-bind来绑定。

这里顺便提一句，如果你需要用v-for来将props传来的数据遍历，你需要换种形式。之前直接用`v-for(item in items)`是不可行的。

我们需要这么做：

```

	<li  transition="animate" class="checkeditem" v-for="n in data.length">
    <div class="name">{{ data[n] }}</div> 
  </li>
  
```

这里再提一句，如何循环用v-for给子组件传递props里的数据.

```

<li v-for="item in count">
	<voteitem :id.sync="item.uid" :name.sync="item.name" ></voteitem>
</li>

```

## 结尾
期间遇到一些坑之前也没仔细看文档，后来群里有人指出方法，又再次仔细去看了一下文档。1.0的文档比0.12的好读一些=-=。什么时候能进阶到去看源码分析的地步呢。还是继续努力吧。


