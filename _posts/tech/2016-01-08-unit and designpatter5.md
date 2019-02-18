---
layout: post
title: 单元测试与设计模式(5)
category: 技术
tags: [js,设计模式,should.js,karma,单元测试,进阶,基础,积累,Vue.js,Vue,Webpack]
keywords: 前端,资料,学习,javascript
description: 
---

# 正文

设计模式：

```
    /**
     * [Command]
     * 命令模式 将请求与实现解耦并封装成独立对象，从而使不同请求对客户端实现参数化。
     * 
     */
    Command:function(){
      //实现模块
      var result = '';
      var LinshuiCommand = (function(){
        //创建方法
        var Action = {
          //创建方法
          create:function(msg){
            result +=  msg+"2333";
          },
          //展示方法
          display: function(msg){
            result +=  msg+"666"
          }
        }
        //命令接口
        return {
          excute: function(msg){
            //如果没有指令返回
            if(!msg)
              return;
            //如果命令是一个数组
            if(msg.length){
            //遍历执行多个命令
              for (var i = 0, len = msg.length; i < len ; i++) {
                  LinshuiCommand.excute(msg[i]);
              }
            }else{
            //如果msg.param不是一个数组，将其转换为数组，apply第二个参数要求格式
              msg.param = Object.prototype.toString.call(msg.param) === "[object Array]" ? msg.param : [msg.param];

              Action[msg.command].apply(Action,msg.param);
            }
          }
        } 
      })();

      LinshuiCommand.excute([
        {command:"create",param:"linshui"},
        {command:"display",param:"zhaoying"},
      ]);
     console.log(result);
     return result;
    
    },

    /**
     * [Mediator]
     * 中介者模式
     * 通过中介者对象封装一系列对象之间的交互，使对象之间不再互相引用，降低他们之间的耦合，有时中介者对象也可改变对象之间的交互.
     */
    Mediator:function(){
      //中介者对象
      var Mediator = function(){
        var _msg = {};
        return {
          //订阅消息方法
          register : function(type,action){
            //如果该消息存在
            if(_msg[type])
              //存入回调函数
              _msg[type].push(action);
            else{
              //不存在则建立消息容器并存入新消息回调函数
              _msg[type] = [];
              _msg[type].push(action);
            }
          },

          send : function(type){
            //如果该消息已被订阅
            if(_msg[type])
              for(var i = 0, len = _msg[type].length;i < len;i++){
                _msg[type][i] && _msg[type][i]();
              }
          }

        }
      }();

      var result = '';
      Mediator.register('demo',function(){
        result += "lin";
      })
      Mediator.register('demo',function(){
        result += "shui";
      })
      Mediator.send('demo');

      return result;
    }
```

单元测试:

```
 it('对命令模式正向测试,返回值应该是linshui2333zhaoying666', function() {
    var defaultData = myComponent;  
    defaultData.methods.Command().should.be.eql("linshui2333zhaoying666");
  });

  it('对中介者模式正向测试,返回值应该是linshui', function() {
    var defaultData = myComponent;  
    defaultData.methods.Mediator().should.be.eql("linshui");
  });
```


