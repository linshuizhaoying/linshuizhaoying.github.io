---
layout: post
title: 从零开始开发个人UI管理系统
category: 技术
tags: [学习,私货,前端,整站开发]
keywords: Javascript,代码,前端,学习总结
description: 
---

## 前言
整个暑假开始之前就已经在着手开发这款系统，它长这个样子.

![imgn](http://haoqiao.qiniudn.com/uimanage01.png)

恩，它现在还是一个半成品，原本打算开发完写总结，但是感觉这样到时候会漏掉一些细节，因此在开发后端的时候打算开始写一些思路和步骤。

## 整体介绍

这款系统开发的目的是为了解决平时收集漂亮的ui代码以及为了整理几套系统的UI，比如以后如果想要开发某个主题的应用，可以直接直接拉出一套已经成熟的UI来搭建。还有就是平时看到一些比较好的特效，可以对应搜集然后在需要的时候直接拉出来用。因此这个系统主要目的还是代码片段搜集以及组件化代码整理。

## 前端部分

前端部分也没啥多说的，基本就是把自己脑子里的页面先制作出静态页面然后加动效。这里直接引用animated.css方便开发。

## 后端部分

这部分是先设计数据库关系。

![imgn](http://haoqiao.qiniudn.com/uimanage02.png)

这是最初的设计稿，可能开发到后面会添加一些字段，但是基本上大局不会改变。

设计思路是用户注册需要邀请码，一个用户至多要求3个人（这个可以在管理员后台修改）

一个用户可以有多个propject，这是为了让用户能够根据自己需求制作几套风哥不同的UI，比如后台管理系列-蓝色/绿色/黑色。

每个Project对应的一整套UI代码。

其实思路这样看上去挺简单。需要解决的一个是邀请码的生成，一个是最基础的增删改操作。开发这种需求不大的系统只要开始把格局定下来，然后用思维导图做出前端分布和后端数据库就可以按部就班。因为作为一个前端，后端开发自然选择node.js。然后又因为个人最亲近的还是express，然后配合Mongodb既可以快速开发。
关于mongoose的介绍和使用,推荐
[这个网站](http://www.hubwiz.com/class/543b2e7788dba02718b5a4bd),比官方文档是更容易理解和上手的。

首先我们需要运行
`mongod` 来运行Mongodb数据库。对于mac/linux系统用户需要先跳到最高权限`sudo su`然后再运行。

然后我们需要创建一个数据库。然后定义schema和model，这里我写个例子:

```
var mongoose = require("mongoose"); //引用mongoose
var db = mongoose.connect("mongodb://127.0.0.1:27017/uimanage"); //链接数据库
// Schema —— 一种以文件形式存储的数据库模型骨架，
// 无法直接通往数据库端，也就是说它不具备对数据库的操作能力，
// 仅仅只是数据库模型在程序片段中的一种表现，可以说是数据属性模型(传统意义的表结构)，
// 又或着是“集合”的模型骨架。

//定义users
var UserSchema = new mongoose.Schema({
	  username : { type:String },
	  password : { type:String },
	  email: { type:String },
	  limit:{type:String},//user or admin
	  created_time : { type:Date, default:Date.now },
	  invitedcode_id  : { type:Number, default:0 },//邀请码
	  avatar : { type:String } //头像地址
	  // lastlogin_time : { type:Date, default:Date.now }, 如果需要监控用户最近登录时间可以添加
});
var UserModel = db.model("uimanage", UserSchema );
//定义方法


/**
 * [查找所有用户]
 * return allusers
 */
//返回只包含一个键值name、age的所有记录
UserModel.find({},{name:1, age:1}，function(err,docs){
   //docs 查询结果集
})
 /**
  * [查找指定用户]
  * return user || null
  */
UserModel.find({ "username": 'linshui' }, function (error, docs) {
  if(error){
    console.log("error :" + error);
  }else{
    console.log(docs); //docs: age为28的所有文档
  }
}); 
//添加管理员
UserModel.create([
                  { username:"linshuizhaoying",password:'test', age:20,email:'4799109@qq.com',invitedcode_id:'2333', avatar:'img/2.png',limit:'admin'},
                 ], function(error,docs) {
    if(error) {
        console.log(error);
    } else {
        console.log('save ok');
    }
});
 /**
  * [添加用户]
  * return success || error
  */
UserModel.create({ username:"linshui", age:22,}, function(error,doc){

});

// var conditions = {name : 'test_update'};
 
// var update = {$set : { age : 16 }};
 /**
  * [更新用户数据]
  * return success || error
  */
UserModel.update(conditions, update, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('Update success!');
    }
});

// var conditions = { name: 'tom' };
  /**
  * [删除用户]
  * return success || error
  */
UserModel.remove(conditions, function(error){
    if(error) {
        console.log(error);
    } else {
        console.log('Delete success!');
    }
});
```

大概了解如何操作数据之后我们需要配合express来修改部分代码。基于github十几个版本的例子之后，总结出了比较适合的方案.

首先建立一个data文件夹，建立_model.js 和_Controller.js文件，原本想要独立将连接数据库的代码单独存为一个link.js文件，然后在express的加载app.js中直接载入，但是经过测试发现这样无法对数据库进行访问操作。因此将连接数据库代码放在model.js当中.

```
userModel.js:

var mongoose = require("mongoose"); //引用mongoose
// 连接字符串格式为mongodb://主机/数据库名
mongoose.connect('mongodb://localhost/uimanage');

//定义users
var UserSchema = new mongoose.Schema({
      username : { type:String },
      password : { type:String },
      email: { type:String },
      limit:{type:String,default:'user'},//user or admin
      created_time : { type:Date, default:Date.now },
      invitedcode  : { type:String},//邀请码
      avatar : { type:String } //头像地址
      // lastlogin_time : { type:Date, default:Date.now }, 如果需要监控用户最近登录时间可以添加
});

var User = mongoose.model('User', UserSchema);
//倒出模型
module.exports = User

```

之后在userController.js当中引入并将方法都独立出来：

```

var User = require('./userModel.js')


exports.addUser = function ( req, res, next ){
  //console.log(req)
  var user = new User({
      username:    req.body.username,
      password:    req.body.password,
      email :    req.body.email,
      invitedcode:    req.body.invitedcode,
      created_time: Date.now(),
      limit: "user",
      avatar: "./img/1.png"

  })
  user.save(function(err) {
    if(err) {
      console.log('Error while saving User: ' + err);
      res.send({  status: 'error', message:error  });
      return;
    } else {
      console.log("User created");
      return res.send({ status: 'ok', message:"ok" });
    }
  });
};

```

最后只需要在route文件中直接引入即可

```

var express = require('express');
var router = express.Router();
var userController = require('../data/userController.js');
/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});
/* GET users listing. */
router.get('/login', function(req, res, next) {
  res.render('login', { title: '登录' });

});
router.post('/addUser', userController.addUser);

module.exports = router;

```

基础的框架打下来，接下来就是细节问题。
我们需要解决的细节如下:

```
1.用户密码必须加密保存
2.邀请码生成模块
3.用户登录状态验证机制

```

第一个很好解决，我们只需在载入Node模块md5即可,

```

npm install md5

```

第三个关于用户登录状态验证机制考虑到安全我决定使用redis。然而在mac上调试有些问题。经过排除问题得到一套比较详细的方案。
首先需要安装去redis的官网手动下载安装包并解压，然后

```
cd 你解压的目录
sudo su
make test
make install
redis-server

```
之后

```
/**
 * [session 管理]
 */

var session = require('express-session');
var RedisStore = require('connect-redis')(session);
var options = {
     "host": "127.0.0.1",
     "port": "6379",
     "ttl": 60 * 60 * 24  //Session的有效期为1天
};
app.use(session({
     store: new RedisStore(options),
     secret: 'linshuizhaoying session',
     proxy: true,
     resave: true,
     saveUninitialized: true
}));
// 经过中间件处理后，可以通过req.session访问session object。比如如果你在session中保存了session.userId就可以根据userId查找用户的信息了。

```

注意这段代码的排放顺序。

然后在登录代码里面添加`req.session.user = result.username;`

然后写一段路由控制管理页面登录

```

router.get('/admin', function(req, res, next) {
	  console.log(req.session)
		if(req.session.user == 'admin'){
	    res.render('admin', { title: '后台管理' });
		}else{
			res.redirect('/login');
		}
});

```
这样，状态管理基本就扔给服务器session来保存。

-------更新 8.29------------
之前由于考驾照+自己懒导致拖延了一个多月

然后重新上手的时候发现已经有点困难了，幸好之前写了注释，重新阅读然后继续写了一个星期。
基本功能已经写完了，剩下的就是边角料和重新以ES2015重构整个项目。
先来看效果：

![imgn](http://haoqiao.qiniudn.com/uimanage8291.gif)

ps:看不到图可能是图太大了，建议另外复制动态图链接打开。

首先续写第一件事情就是把数据库后台全部搞定，而且要考虑到各个系统之间的通讯，这里我是之间让后台来标记当前用户，并配合window.localStorage来控制，这就是复杂应用的一个麻烦点，然后我还选择最麻烦的就是自己处理一切，其实这个地方我应该考虑一下vuex的...但是项目也写了大半了我就没改了。

贴部分controller的代码：

```
/**
 * [addUser func]
 * 添加用户
 * 获取表单传来的参数
 * 返回 status 状态 message 信息
 */
exports.addUser = function ( req, res, next ){
  //console.log(req)
  var user = new User({
      username:    req.body.username,
      password:    md5(req.body.password),
      email :    req.body.email,
      invitedcode:    req.body.invitedcode,
      created_time: Date.now(),
      limit: "user",
      avatar: "./img/1.png"

  })
  console.log(user)
  /* 判断用户是否存在 */
  User.findOne({username:user.username},function(err,result){
     if(result != null){
        res.send({  status: 'error', message:"用户已存在"  });
     }else{
        user.save(function(err) {
          if(err) {
            console.log('Error while saving User: ' + err);
            res.send({  status: 'error', message:error  });
            return;
          } else {
            console.log("User created");
            res.send({ status: 'ok', message:"ok" });
            return 
          }
        });
     }
  })
};
/**
 * [getUsers func]
 * 需要管理员权限
 * 后台获取用户所有信息列表
 * 无接受参数
 * 直接返回 用户列表
 */
exports.getUsers = function ( req, res, next ){
  User.find({}, function(err, users) {
    if (err) {
      res.json({message: err});
    } else {
      res.json({users: users});
    }
  });
};
/**
 * [getUsers func]
 * 需要管理员权限
 * 获取指定用户所有信息
 * 接受url传来的id参数
 * 直接返回 用户信息
 */
exports.getUser = function ( req, res, next ){
  User.findById(req.params.id, function(err, user) {
    if (err) {
      res.json({message: err});
    } else {
      res.json({user: user});
    }
  });
};
/**
 * [getCurrentUser func]
 * 获取当前用户名
 * 直接返回 用户信息
 */
exports.getCurrentUser = function ( req, res, next ){
  if(req.session.user){
    res.send({ status: 'ok', message:req.session.user });
  }else{
     res.send({ status: 'error', message:"还未登录！" });
  }
  
};


```

之后就是模块之间的切换，这涉及几个地方，其中一个是UI管理部分点击选项中间内容改变，这里我是统一用`v-show`来控制。

这个项目的麻烦点其实在于在线代码编辑器。其中需要考虑的技术点分别有：

```
1.选择合适的开源代码编辑器
2.如何管理新建代码和查看UI这两个部分，如何将其组件化
3.如何解决多查看UI显示的冲突
4.如何管理不同类别的UI，比如我分成基础UI和高级UI,高级UI是包含JS和外部引入链接地址的，而且这两个编辑器界面应该是不同的

```
我们来一一解决，首先我选择的开源代码编辑器是`CodeMirror`，这已经是比较好的了，一开始我是打算拿之前自己写的那个直接来用。。。当然想了想立刻放弃2333 但是这个`CodeMirror`有个bug，就是一开始加载的时候它其实是空白的，即使里面有内容它需要点击一下才能显示，而且官方并没给出比较好的解决方法，只能参考stackoverflow的用个定时器来循环遍历refresh。

第二点组件化我是这么考虑的，因为一开始我构思的功能来看

![imgn](http://haoqiao.qiniudn.com/82911.png)

应该是加载初步的列表，然后点击右边的<- ->来打开配置界面。这样我就将其分成两个部分`basic-ui.vue` 和`basic-viewcode.vue` 并让`basic-ui.vue`传递参数给`basic-viewcode.vue`

就像这样

```
    <basicviewcode :basicviewcodeshow.sync="basicviewcodeshow" :sign="signnum" :title="title" :desc="desc" :htmlcode="htmlcode" :csscode="csscode" class="animated" transition="bounce"></basicviewcode>
    
```

第三个多个UI显示冲突我是这么考虑的，每个UI有个独立的project_id和ui_id，因此我在每个编辑器的html css js后面加一个ID标记sign.

```
<textarea v-model="htmlcode" id="htmlcode{{sign}}"></textarea>
```

因此ID可能会变成`basicHtmlCode57c3364bce3d899060e5093f`这样奇怪的样子，不过没关系，我们是直接后台生成的唯一ID，直接获取即可。

关于新建代码这个功能它其实是多个地方调用同一个，不需要每个地方都去生成，因此我将其修改为获取指定type然后自动修改提交内容。

在后台我是这么定义UI这个字段的

```

// //定义Uis

var UiSchema = new mongoose.Schema({
      title : { type:String },
      desc : { type:String },
      type : { type:String }, //btn panel form nav list other component plugin
      htmlcode:{type:String},
      csscode:{type:String},
      jscode:{type:String},
      desc: { type:String },
      project_id: { type:String },
      link:{ type:String },//用来保存高级UI的外链地址
      created_time : { type:Date, default:Date.now }
});

```

但是对于高级UI我又需要重新写一个与其对应的新建UI模块。不过这个改改即可。

整个项目结构如下：

![imgn](http://haoqiao.qiniudn.com/82911tree.png)

## 结尾

接下来需要做的工作是把其他边角料模块写完，项目结构优化，比如之前图方便写了三个路由...还有把ajax换成别的。把代码全部ES6化（这块会重点拿出来写篇总结）。
事实上看上去蛮复杂的系统搞清思路大概也能磨出来。但是一个暑假过去才写完一个感觉自己已经是咸鱼一个了，现在这篇文章也是午夜3点写的QAQ,白天没效率真可怕


