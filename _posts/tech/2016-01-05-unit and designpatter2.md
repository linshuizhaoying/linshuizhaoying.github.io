---
layout: post
title: 单元测试与设计模式(2)
category: 技术
tags: [js,设计模式,should.js,karma,单元测试,进阶,基础,积累,Vue.js,Vue,Webpack]
keywords: 前端,资料,学习,javascript
description: 
---

# 前言
将延续上一章将一系列设计模式慢慢手敲出来。

# 正文

设计模式：

```
/**
     * [原型模式]
     * 原型模式是指将原型对象指向创建对象的类，使这些类共享原型对象的方法和属性
     * 创建基类时，将简单而且差异化的内容放在构造函数中，将消耗资源较多的方法放在基类的原型中。
     * 子类通过组合继承或者寄生组合式继承将方法继承下来。然后对于子类中需要重写的方法进行重写
     * @return {[result]} []
     */
    prototype:function(a,b){
      //图片轮播类
      var LoopImages = function(imgArr,container){
        this.imagesArray = imgArr;
        this.container = container;
      }
      LoopImages.prototype = {
        createImage : function(){
          console.log("I had Created new Image");
        },
        changeImage : function(){
          console.log("I had Changed Image");
        }
      }

      //上下滑动类
      var SlideLoopImg = function(imgArr,container){
        //构造函数继承图片轮播类
        LoopImages.call(this,imgArr,container);
      }
      SlideLoopImg.prototype = new LoopImages();
      SlideLoopImg.prototype.changeImage = function(){
        console.log("SlideLoopImg had Changed Image");
      }

      var test = new SlideLoopImg(a,b);
      return test.container;

    },
    /**
     * [Singleton]
     * 利用单例模式来创建命名空间
     */
    Singleton:function(type){
      var Linshui = {
        Util : {
          testA : function(){
            return "UtiltestA"
          }
        },
        Tool : {
          testA : function(){
            return "TooltestA"
          }
        }
      }

      var result = Linshui[type].testA();
      return result;
    }


```

单元测试:

```
describe('设计模式Three设计模式单元测试', function() {
  var myComponent = require('../src/vue/app.vue');  
  it('对原型模式正向测试,返回值应该是linshuizhaoying', function() {
    var defaultData = myComponent;  
    defaultData.methods.prototype("Javascript","linshuizhaoying").should.be.eql("linshuizhaoying");
    
  });
  it('对单例模式正向测试,返回值应该是TooltestA', function() {
    var defaultData = myComponent;  
    defaultData.methods.Singleton("Tool").should.be.eql("TooltestA");
    
  });
  it('对单例模式正向测试,返回值应该是UtiltestA', function() {
    var defaultData = myComponent;  
    defaultData.methods.Singleton("Util").should.be.eql("UtiltestA");
    
  });

});
```





