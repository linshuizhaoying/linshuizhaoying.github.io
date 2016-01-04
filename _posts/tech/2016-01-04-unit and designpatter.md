---
layout: post
title: 单元测试与设计模式(1)
category: 技术
tags: [js,设计模式,should.js,karma,单元测试,进阶,基础,积累,Vue.js,Vue,Webpack]
keywords: 前端,资料,学习,javascript
description: 
---

# 前言

2016年开个好头,利用单元测试来辅助学习设计模式。其实买了挺多本设计模式的书，但个人感觉最好的还是张荣铭的《javascript设计模式》。

# 正文

因为之前已经有过如何搭建单元测试环境的内容就不再累述。

因为以后开发还是以vue.js为主，因此把测试例子写进了app.vue的methods中,然后直接在test.js中require就行了。

然后直接看第一部分基础篇:

```

     /**
     * [designpatter1 函数方法]
     * @return {[result]} [返回对应结果]
     */
   designpatter1:function(test){

      //Design Patterns One
      var checkName = function(){
        return "checkName";
      };

      var checkEmail = function(){
        return "checkEmail";
      };

      var checkPassword = function(){
        return "checkPassword";
      };

      return checkName();

   },

   /**
    * [designpatter2 对象方法]
    * @return {[result]} [返回对应内容]
    */
   designpatter2:function(type){
      var CheckObject = {
        checkName : function(){
          return "checkName2";
        },

        checkEmail : function(){
          return "checkEmail2";
        },

        checkPassword : function(){
          return "checkPassword2";
        },
      }

      return CheckObject[type]();

   },

   /**
    * [designpatter3 链式调用]
    * @return {[result]} [返回对应内容]
    */
   designpatter3:function(type1,type2,type3){
      var result = "";
      var CheckObject = {
        checkName : function(){
          result += "1";
          return this;
        },

        checkEmail : function(){
          result += "2";
          return this;
        },

        checkPassword : function(){
          result += "3";
          return this;
        },
      }
      CheckObject.checkName().checkEmail().checkPassword();
    
      console.log(result);
      return result;

   },
   
```

为了测试方便，每个测试例子最后都return了结果。

然后我们来看基础篇的测试例子是怎么写的:

```

describe('设计模式One基础单元测试', function() {
 	var myComponent = require('../src/vue/app.vue');  
  it('对函数方法调用正向测试', function() {
    var defaultData = myComponent;  
    defaultData.methods.designpatter1().should.be.eql("checkName");
    
  });

  it('对对象方法调用,判断是否是函数', function() {
    var defaultData = myComponent.methods.designpatter2;

    defaultData.should.be.an.instanceof(Function);
    //console.log(myComponent);
  });
  it('对对象方法调用，判断是否包含name属性', function() {
    var defaultData = myComponent.methods.designpatter2;
    defaultData.should.have.property("name");
    //console.log(myComponent);
  });

  it('对对象方法调用，判断是否返回对应值checkPassword2', function() {
   var defaultData = myComponent;  
    defaultData.methods.designpatter2("checkPassword").should.be.eql("checkPassword2");
  });

  it('对对象链式方法调用，判断是否返回对应值', function() {
   var defaultData = myComponent;  
    defaultData.methods.designpatter3("checkName,checkPassword").should.be.eql("123");
  });

});

```

这里我们结合两者可以看到其实我们还是采用了最简单的正向测试。也就是对比返回结果是不是预计的内容，如果是说明测试单元是没问题的，如果两者结果不一致那么说明肯定是哪部分有问题。一些都可以在karma输出的命令日志中看到回显。

然后我们具体来正式学习设计模式最基础的部分：工厂模式

来看代码：

```

/**
     * [安全模式下的工厂模式]
     * @return {[type]} [参数]
     * 在构造函数开始前先判断当前对象this指代是不是类，如果是则new。如果不是说明在全局
     * 作用域下执行，此时this指向window,因此需要重新返回新创建的对象。
     * 这样做的结果是可以直接 var test = Demo() 而不需要new关键词。
     */
    safeFactory:function(type,content){
      var Factory = function(type,content){
        if(this instanceof Factory){
          var temp = new this[type](content);
          return temp;
        }else{
          return new Factory(type,content);
        }
      }
      Factory.prototype = {
        Javascript :function(content){
          this.result = "Javascript" + content;
        },
        Node :function(content){
          this.result = "Node.js" + content;
        },
      }

      var test = Factory(type,content);
      console.log(test.result);
      return test.result;

    },
    
```

从例子中我们可以看到我设定了两个type,一个是`Javascript` 一个是`Node`

然后我们来看测试是怎么写的:

```

describe('设计模式Two基础单元测试', function() {
  var myComponent = require('../src/vue/app.vue');  
  it('对安全工厂模式正向测试,返回值应该是Javascriptzhaoying', function() {
    var defaultData = myComponent;  
    defaultData.methods.safeFactory("Javascript","zhaoying").should.be.eql("Javascriptzhaoying");
    
  });

});

```

如果我们将defaultData.methods.safeFactory("Javascript","zhaoying")里的参数改一改改为linshui.我们来运行`karma start`来看结果是什么

![imgn](http://haoqiao.qiniudn.com/designpatter1.png)

` this[type] is not a constructor`我们就可以很清楚的看见哪部分有问题。




# 结尾

事实上我通过单元测试来学习的只是书上的一些案例。真正要把设计模式用在项目中就需要不断的尝试。如何将单元测试应用在实际项目中也需要不断磨合。不过万事开头难，难的部分我们已经跨过了，剩下的就是水到渠成了。


