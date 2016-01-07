---
layout: post
title: 单元测试与设计模式(4)
category: 技术
tags: [js,设计模式,should.js,karma,单元测试,进阶,基础,积累,Vue.js,Vue,Webpack]
keywords: 前端,资料,学习,javascript
description: 
---

# 前言
按照前几日一样，继续学习-0-

# 正文

设计模式：

```
   /**
     * [State]
     * 状态模式，当一个对象内部状态发生改变时，会导致其行为的改变，这看起来像是改变了对象
     */
    State:function(num){
      var States = function(){
        var result;
        var State = {
          state0 : function(){
            result = "state0";
          },
          state1:function(){
            result = "state01";
          },
          state2:function(){
            result = "state2";
          }
        }
        //获取某一种状态并执行其对应的方法
        function show(test){
          State['state' + test] && State['state' + test]();
          return result;
        }
        return {
          //返回调用状态方法接口
          show : show,
        }
     }();
      
      return States.show(num);
    },

    /**
     * [Strategy]
     * 策略模式，将定义的一组算法封装起来，使其互相之间可以替换。封装的算法有一定独立性，不会随客户端变化而变化
     */
    Strategy:function(num){
      var PriceStrategy = function(){
        var strategy = {
          return30 : function(price){
            return + price + parseInt(price / 100) * 30;
          },
          return50 : function(price){
            return + price + parseInt(price / 100) * 50;
          },
          return50 : function(price){
            return + price + parseInt(price / 100) * 50;
          },
          percent50 : function(price){
            return price * 100 * 50 / 10000;
          }
        }
         return function(algorithm,price){
          return strategy[algorithm] && strategy[algorithm](price);
          /**扩展算法
           addStrategy : function(type,fn){
            strategy[type] = fn;
           }
          PriceStrategy.addStrategy('xx',function(value){
            return "xxx";
          });

           */
         } 
        
      }();

      var price = PriceStrategy('return50',num);
      return price.toString();
    },
```

单元测试:

```

  it('对状态模式正向测试,返回值应该是state2', function() {
    var defaultData = myComponent;  
    defaultData.methods.State("2").should.be.eql("state2");
  });

  it('对策略模式正向测试,返回值应该是state2', function() {
    var defaultData = myComponent;  
    defaultData.methods.Strategy("314.67").should.be.eql("464.67");
  });
  
```

