---
layout: post
title: 单元测试与设计模式(3)
category: 技术
tags: [js,设计模式,should.js,karma,单元测试,进阶,基础,积累,Vue.js,Vue,Webpack]
keywords: 前端,资料,学习,javascript
description: 
---

# 正文
设计模式:

```
/**
     * [Decorator]
     * 装饰者模式
     *在不改变源对象的基础上，通过对其进行包装扩展（添加属性或者方法）使原有对象能够满足用户的更发杂的需求。
     */
    Decorator:function(name){
      var result = "";
      var linshui = {
        Util : {
          testA : function(){
             result = "linshui";
          }
        }
      }
      //装饰者
      var decorator = function(fn){
        //如果事件源存在
        console.log(typeof linshui.Util[fn]);
        if(typeof linshui.Util[fn] === 'function'){
           //缓存事件原有回调函数
           var oldFn = linshui.Util[fn];
           function zhaoying(){
             result += "zhaoying";
           }
           //执行原有回调函数
           oldFn();
           //执行新增的回调函数
           zhaoying();
        }else{
          result = "new function";
        }
      }
      decorator(name);
      return result;

    },

    /**
     * [Observer]
     * 观察者模式
     * 又称为发布-订阅者模式或者消息机制，定义了一种依赖关系，解决了主体对象与观察者之间功能的耦合。
     */
    Observer:function(type,msg){
      //将观察着放在闭包,立即调用
      var Observer = (function(){
        //防止消息队列暴漏而被篡改因此将消息容器作为静态私有变量保存
        var _messages = {};
        return {
          //订阅消息方法
          regist : function(type,fn){
            //如果此消息不存在则创建一个该消息类型
            if(typeof _messages[type] === 'undefined'){
              _messages[type] = [fn];
            }else{
              //将动作方法推入该消息对应的动作执行序列中
              _messages[type].push(fn);
            }
          },
          //发布消息方法
          fire : function(type,args){
            //如果该类消息没有被注册则返回
            if(!_messages[type])
              return;
            var events = {
              type : type,  //消息类型
              args : args || {}  //消息携带数据
            },
            i = 0,  //消息动作循环变量
            len = _messages[type].length;  //消息动作长度
            //遍历消息
            for(; i < len ;i++){
              //依次执行注册的消息对应的动作序列
              _messages[type][i].call(this,events);
            }
          },
          //取消订阅方法
          remove : function(type,fn){
            //如果消息动作队列存在
            if(_messages[type] instanceof Array){
              //从最后一个动作遍历
              var i = _messages[type].length - 1;
              for (; i > 0; i--) {
              //如果存在该动作则在消息动作序列中移除相应动作
               _messages[type][i] === fn && _messages[type].splice(i,1);
              };
            }
          }
        }
      })();
      
      var result;
      console.log(Observer);
      Observer.regist(type,function(e){
        result = e.args.msg+"2333";
      });

      Observer.fire(type,{msg:msg});

      return result;

    }
```

单元测试:

```

  it('对装饰者模式正向测试,返回值应该是linshuizhaoying', function() {
    var defaultData = myComponent;  
    defaultData.methods.Decorator("testA").should.be.eql("linshuizhaoying");
  });

  it('对装饰者模式正向测试,返回值应该是linshuizhaoying', function() {
    var defaultData = myComponent;  
    defaultData.methods.Decorator("testB").should.be.eql("new function");
  });

  it('对观察者模式正向测试,返回值应该是linshuizhaoying', function() {
    var defaultData = myComponent;  
    defaultData.methods.Observer("linshui","linshuizhaoying").should.be.eql("linshuizhaoying2333");
  });
  
```


