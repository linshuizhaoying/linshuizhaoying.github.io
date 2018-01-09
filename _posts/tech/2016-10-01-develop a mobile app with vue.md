---
layout: post
title: 使用Vue与koa开发移动端App
category: 技术
tags: [学习,整理,前端,实战总结,vue,koa]
keywords: web,代码,前端,学习总结,mobile,移动端,app,手机应用,koa,rest,api,vue
description: 
---

# 前言

当初大一的时候报了学校的一个项目，然后这个app是其中一部份功能衍生出来的。主要功能是提供在线图书漂流平台，以书换书或者仅仅只是纯粹的借阅。

![imgn](http://haoqiao.qiniudn.com/tushupiaoliu1.png)

先大概考虑了一下。

然后根据需求确定移动端这块是用`vux` + `JS计算Rem`的方式来做响应式。

然后一开始我打算尝试`测试驱动`，然后我发现，我没有设计稿啊，我无法确定整个产品的功能，如果一开始我就去想所有产品功能。。。中途再变一下，好像更麻烦的样子。

因此我打算先把`静态页面+交互`开发完毕，然后就能知道大概的接口，然后再写测试，接着再把代码写完。

## 静态页面+交互

交互这块打算简单直接用现成的`animated.css` 但是官网没有完全适配`vue`的，因此去github搜了一下有人已经整合了一下，直接拿过来用。

当然也可以直接在`index.html`加载`animated.css`
然后在app.vue里写上

```

// 注册
Vue.transition('fade', {
  enter: function () {},
  leave: function () {}
})


```

不过你用的效果越多代码越多，还不如在css这块下手解决。

然后这里提一下，当我这么写的时候

`<login class="animated" transition="rotateDownLeft" v-show="navShow == 'login' "></login>`

我发现当我切换组件的时候，效果会出现偏移，也就是应该是卡片式旋转切换的却变成了第一个组件旋转，然后第二个组件从底部旋转显示，再突兀的出现在第一个的位置。

后来经过排查发现是`postion`的问题。然后我看了下那块的css，改成`position:fixed`即可。

## 关于Mock

之前我推荐了一款`puer-mock` 虽然它的确能生成数据和文档，但是`它不能不能跨域`.这是非常麻烦的一件事，然后我看网上一些解决方法都无法帮助我解决，于是我就换成`mock-api`,虽然它不支持文档，但是作者处理了跨域的问题。而且它可以全局安装，不需要在生产环境里再建个文件夹了。
不过这个应用有个坑就是官方文档并没有写指定参数查询，它只提供了`纯静态json展示`。不过作者在文档里加了一句`另外，你可以使用nodejs能做到的所有功能。`是的，你没看错，这个意思就是我大概把基本功能写了，你们想要什么功能自己写呗。
良心作者，给使用者还提供了锻炼自我的机会~
按照那简朴的文档，把接受指定id显示对应的功能写了个模板

```

var store = [{
    id: 0,
    name: 'ssstom',

}, {
    id: 1,
    name: 'jerry'
}, {
    id: 2,
    name: 'lucy'
}, {
    id: 3,
    name: 'green'
}, {
    id: 4,
    name: 'white'
}, {
    id: 5,
    name: 'jay'
}];

module.exports = [{
    method: 'get',
    url: '/book/:id',
    response: function() {
        return  {
            "id|number": "{{this.params.id}}",
            "name": store[this.params.id].name,
        };
    }
}];

```

在目录下执行`mock-api serve`即可

## 遇到的一些问题

### JS 指定日期转换时间戳问题
在实体机测试的时候，微信端日期计算居然不正确，返回`NAN` 最终查到是日期格式问题，因此在计算的时候转换不再用`-`而是换成`\`

```

export const ComputeDate = (date1, date2) => {
  var aDate, oDate1, oDate2, iDays
  var sDate1 = date1
  var sDate2 = date2
  aDate = sDate1.split('-')
  oDate1 = new Date(aDate[1] + '/' + aDate[2] + '/' + aDate[0])
  aDate = sDate2.split('-')
  oDate2 = new Date(aDate[1] + '/' + aDate[2] + '/' + aDate[0])
  iDays = parseInt(Math.abs(oDate1 - oDate2) / 1000 / 60 / 60 / 24) + 1
  return iDays
}

```

## 后端开发

一开始就是打算用`koa`来作为提供数据的提供源，然后我就想可不可能实现一种就是用`koa`监听webpack打包前端的`vue`并且提供数据。但是当我翻了不少文档和看了不少别人的项目。这种想法是可行的，因为`koa`中其实是又类似的中间件来帮助管理`webpack`但是这会造成前后端开发的混乱。
比如我一开始需要模拟数据是用前端的`webpack`来监听打包，如果开发到后期需要后端的时候，要开始对其修改。因此现在比较好的处理方法大概还是后端独立监听已经打包后的页面。

磕磕碰碰看了一个多星期的koa项目源码，将github上的基本上都翻来覆去看过好几遍，主要是`koa` + `vue` + `mongoose`

经过多次调试终于总结出了一个现阶段koa适合的rest api方案

首先自然是数据库的加载

app.dev.js:

```

var db = require('./config/mongoose')()
db.on('error', console.error.bind(console, 'error: connect error!'))
db.once('open', function () {
  // 一次打开记录
  console.log('connect success!')
})

./config/mongoose.js:

const mongoose = require('mongoose')

const mongooseDB = function () {
  // mongoose.connect('mongodb://23.88.229.24:27017/drift')
  mongoose.connect('mongodb://127.0.0.1:27017/drift')
  return mongoose.connection
}

module.exports = mongooseDB

```

然后就是加载路由

```

  // 导入中间件
var koa = require('koa-router')()
var users = require('./api/users');
// 使路由生效
koa.use('/users',users.routes(),users.allowedMethods());
koa.get("/*", function *() {
  this.body = {status:'error',data:'404'};
});
app.use(koa.routes())

```

这里强调下为啥用`koa.get("/*"` 因为你在开发的时候调试很难判断你的路由是否真正的生效了，一般情况下post的都是返回一个`not found`(所有错误都是返回这个)，因此你需要在所有路由配置最后再加一句这个确保你至少有一个路由是生效的。

然后`./api/users` 其实是一个文件夹里面有三个文件 `index.js` 和 `user.controller.js` 和 `user.model.js`

```

user.model.js:

var mongoose = require('mongoose');

var userSchema = new mongoose.Schema({
  username: String,
  nickname: String,
  password: String,
  address: String,
  avatar: String
});
var User = mongoose.model('User', userSchema);
module.exports = User


user.controller.js:

// const mongoose = require('mongoose');
// const User = mongoose.model('User');
const User = require('./user.model.js');
//后台获取用户列表
exports.getUserList = function *() {
	try{
		const count = yield User.count();
		const userList = yield User.find({}).exec();
		this.status = 200;
		this.body = { status:"ok", data: userList, count:count };
	}catch(err){
		this.throw(err);
	}
}

//添加用户
exports.addUser = function *() {
	// const nickname = this.request.body.nickname;
	// const username = this.request.body.username;
	// const password = this.request.body.password;
	try{
		this.request.body.address = ""
		this.request.body.avatar = ""
		var newUser = new User(this.request.body);
		var result = yield User.find({username: this.request.body.username})
		if(result.length == 0){
			const user = yield newUser.save();
			this.status = 200;
			this.body = {status:"ok",user_id:user._id,message:"添加成功~"}
		} else {
			this.status = 200;
			this.body = {status:"error",message:"该用户已存在!"}
		}

	}catch(err){
		this.throw(err);
	}
}
//登录
exports.login = function *() {
	// const username = this.request.body.username;
	// const password = this.request.body.password;
	try{
		var result = yield User.find({username: this.request.body.username})
		console.log(result[0].password == this.request.body.password)
		if(result[0].username == this.request.body.username && result[0].password == this.request.body.password){
			console.log("2333")
			this.status = 200;
			this.body = {status:"ok",nickname:result[0].nickname,message:"登录成功~"}
		} else {
			this.status = 200;
			this.body = {status:"error",message:"账号或者密码错误,请重新登录!"}
		}

	}catch(err){
		this.throw(err);
	}
}



index.js:


const router = require("koa-router")();
const controller = require('./user.controller');

router.get('/getUserList',controller.getUserList);
router.post('/addUser',controller.addUser);
router.post('/login',controller.login);
module.exports = router;


```

按照这个套路走，你可以很快的用Koa搭建一个REST Api服务。

### 移动端图片上传
一开始没想那么多，以为就是普通的上传，然后koa写个方法去接收然后存储。但是在网上搜索资料的时候发现。好像移动端上传还有蛮多坑的。这是自己没想到的。

```

1.压缩并且按照比例自动裁切图片然后上传
2.图片扭曲、某些设备不自动旋转图片方向，没有jpeg压缩算法..、
3.不支持new Blob,formData构造的文件size为0..
4.还有某些机型和浏览器（例如QQX5浏览器）莫名其妙的BUG..

```

这些原理的解决方案大家可以参考[这篇文章](http://qiutc.me/post/uploading-image-file-in-mobile-fe.html)

出于偷懒我去找个插件 `npm i lrz --save`

因为大概我知道我显示的缩略图是宽度是400px，因此直接写死

```

    document.querySelector('#file').addEventListener('change', function () {
      lrz(this.files[0], { width: 400 })
            .then(function (rst) {
                // 处理成功会执行
              console.log(JSON.parse(window.localStorage.user)[0]._id)
              console.log(rst.formData)

              that.$http.post('http://192.168.1.102:3000/books/upload',
                {data: rst.base64
              }).then((response) => {
                if (response.data.status === 'ok') {
                  console.log(response.data.path)
                  document.querySelector('#demo').src = '../../../static/upload/' + response.data.path
                  that.$http.post('http://192.168.1.102:3000/books/upload',
                    {data: rst.base64
                  }).then((response) => {
                    if (response.data.status === 'ok') {
                      console.log(response.data.path)
                      document.querySelector('#demo').src = '../../../static/upload/' + response.data.path
                      // 保存上传图片名
                      that.img_url = response.data.path
                    } else {
                      that.$dispatch('child-loadShow', '上传出错,请重试!')
                    }
                  }, (response) => {
                    that.$dispatch('child-loadShow', '上传出错,请重试!')
                  })
                } else {
                  that.$dispatch('child-loadShow', '上传出错,请重试!')
                }
              }, (response) => {
                that.$dispatch('child-loadShow', '上传出错,请重试!')
              })
            })
            .catch(function (err) {
                // 处理失败会执行
              console.log(err)
            })
            
```

## 一些其它问题的解决方案
首先就是关于表单这块，我提交之后无法清空已经选中option的值，因为这是引用了vux里的`selector` 而且它并没有提供清空数据的方法。因此我一开始是想着怎么重新加载组件，因为单页面应用你为了重置提交模块整个页面刷新是不正常的。
后来翻阅文档，想出一个黑科技的用法那就是`v-if`

```
    <group v-on:click="changeType('book_age')" v-if="clearOption == true">
      <selector placeholder="请选择" title="新旧程度" :options="ageList" @on-change="onChange"></selector>
    </group>
    
    quit () {
      this.clear()
      this.clearOption = !this.clearOption
      setTimeout(() => {
        this.clearOption = !this.clearOption
      }, 100)
      this.$dispatch('child-changeNav', 'usermanage')
    }
    
```

设个短暂的定时器让它快速的重新加载组件即可~

## 结尾

最后效果图如下：

![imgn](http://haoqiao.qiniudn.com/drift1.gif)

原工程先不贴了，还有优化的步骤没做~

