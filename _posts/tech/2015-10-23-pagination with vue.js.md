---
layout: post
title: 基于vue.js实现简单的pagination(分页)功能
category: 技术
tags: [JS，vue.js,前端总结,vue.js,vue学习]
keywords: 前端,资料,学习,vue
description: 
---

## 前言

最近开始学一些新东西，因此没啥时间总结，今天做了一些小东西，总结一下自己现阶段写的基于vue.js的分页。

## 正文
首先背景是需要在展示页加一个分页，后台分页我用的是另外一个插件，然而这个展示页撑死就几十条数据，而且自己也得去实现积累一些组件，因此就动手自己写了一个。

首先说明一下,vue.js似乎更新到1.0版本了，因为原来项目是基于0.12版本的，我也没看新特性是什么。不过应该是向下兼容的吧=-=

对于最基础的分页我们再熟悉不过，功能主要为以下几点:

```
1.基本的按钮跳转，比如第二页就显示第二页数据
2.支持自定义划分页数，per of page。
3.支持上一页 下一页功能。

```
组件懒得自己画，然而又不想直接加载bootstrap整个文件。因此直接去扒bootstrap分页组件的样式。
如下：

```
.pagination {
    display: inline-block;
    padding-left: 0;
    margin: 20px 0;
    border-radius: 4px;
}
.pagination>li {
    display: inline;
}
.pagination>li:first-child>a, .pagination>li:first-child>span {
    margin-left: 0;
    border-top-left-radius: 4px;
    border-bottom-left-radius: 4px;
}
.pagination>li>a:hover{
  background-color: #EAE8E8;
  cursor: pointer;
}
.pagination>li>a, .pagination>li>span {
    position: relative;
    float: left;
    padding: 6px 12px;
    margin-left: -1px;
    line-height: 1.42857143;
    color: #337ab7;
    text-decoration: none;
    background-color: #fff;
    border: 1px solid #ddd;
}
.pagination>li:last-child>a, .pagination>li:last-child>span {
    border-top-right-radius: 4px;
    border-bottom-right-radius: 4px;
}
.paginationactive{
  background-color: #EAE8E8!important;
}
.pagi{
  width: 100%;
  text-align: center;
}
```

因为扒下来是没有active状态的，因此自己写个类强制覆盖原先的就行了。

先来看基本Html

```
     <table  v-repeat="lists">
       <tr>
         <td>{{name}}</td>
         <td>{{project}}</td>
         <td>{{cycle}}</td>
         <td v-show="progress == '未完成' || progress != '已完成'">未完成</td>
         <td v-show="progress == '已完成'">已完成</td>
        </tr>
     </table>
```

因为之前显示是用v-repeat直接循环了，因此如果用别的插件还需要自己手写动态append内容，因此需要基于该v-repeat来考虑如何分页。

本来想直接用vue-grid这个插件，但是发现那么长的代码一大部分功能并不需要，因此只粗略看了一下整个源码就放弃使用外载vue-grid这个想法。

直接把最基础的分页组件放在一个包裹内，以后想在哪用直接可以扩展。

```
 <div class="pagi">
     <ul class="pagination">
          <li>
            <a aria-label="Previous" v-on="click:prev()" >
              <span aria-hidden="true">«</span>
            </a>
          </li>
          <li v-repeat="fullpagenum"><a v-on="click:jump($index)" v-class="paginationactive:checkcurrent($index)">{{ 1 + $index}}</a></li>
          <li>
            <a aria-label="Next" v-on="click:next()">
              <span aria-hidden="true">»</span>
            </a>
          </li>
     </ul>
   </div>
```

大概思路就是先在created生命周期时去获取所有数据A传给$fulldata，然后把数据的长度除以想要分页的数传给$fullpagenum,然后写一个获取指定范围内容函数来裁剪得到数据B，把数据B传给$lists。然后不用管别的vue会自动帮你把剩下的工作完成。

因此具体代码如下:

```
data:function () {
    return {
      lists: "",
      fulldata:"",
      fullpagenum:"",
      perofpage:6,
      currentper:0
    }
  },
  methods: {
    fetch: function() {
      var that = this;
      var lists = [];
      $.ajax({
            type: "GET",
            url: that.url,
            dataType: "json",
          //  async:false,//为了把数据赋值给上个作用域，需要同步进行
            success: function(data) {
           //  console.log(data);
             that.lists = data.content; 
             that.fulldata = data.content;
             that.fullpagenum = Math.ceil( data.content.length / that.perofpage);
             that.getPartData(0,that.perofpage - 1);
           //  localStorage.Cachemarquee = JSON.stringify(data); 
            },
            error: function(jqXHR){
               console.log("发生错误：" + jqXHR.status);
            }
        });
     },
     getPartData:function(start,end){
       var that = this;
       var fulldata = that.fulldata;
       var temp = [];
       if(end > that.fulldata.length - 1){
        end = that.fulldata.length - 1;
       }
       for (var i = start ; i <= end ; i++) {
         temp.push(fulldata[i]);
       };
       that.lists = temp;
     },
     jump:function(num){
       this.getPartData(num * this.perofpage,num * this.perofpage + this.perofpage -1);
       this.currentper = num;
     },
     prev:function(){
      //先判断是否越界
      var current = this.currentper;
      var last = current - 1;
      //console.log(last);
      if(last < 0){
        last = 0;//如果越界了就直接赋值为0
      } 
      this.currentper = last;
      this.jump(last);
     },
     next:function(){
      //先判断是否越界
      var current = this.currentper;
      var next = current + 1;
     // console.log(next);
      if(next > this.fullpagenum - 1){
        next = this.fullpagenum - 1;//如果越界了就直接赋值为0
      } 
      this.currentper = next;
      this.jump(next);
     },
     checkcurrent:function(num){
       console.log(num == this.currentper);
       return num == this.currentper; //点击哪个就给哪个添加active状态
     }
  },
  created:function(){ 
    this.fetch();
    console.log(this.fulldata);
  },
  ready:function(){ 
    console.log("test");
    this.getPartData();

  },
  
```

如果想要添加功能可以直接往里面扩展。

今天就到这-0-

