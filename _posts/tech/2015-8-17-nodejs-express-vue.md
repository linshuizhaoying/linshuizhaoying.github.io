---
layout: post
title: Node.js + Express + Vue.js 自动化开发环境部署
tags: [ Node.js , Express ,Vue.js,Webpack]
keywords: 学习资料,前端,Node.js
description: 
---

## 正文
今天打算用vue.js来做练手项目。之前配置了前端开发的环境，因为我现阶段开发都是基于Node.js+Express4.x的。而且之前用gulp来配置环境，于是想把vue.j的自动化开发配置放在gulp里。

但是经过查找，发现vue.js的作者是支持用webpack的,我找到了勾股的文章,写的不错,但是坑大！直接套用他的方法是直接报错,而且错误提示很奇怪，找不到一个文件路径。

后来在github上找，发现基于vue gulp关键词的内容基本上还是很少。捣鼓了几个小时，最后放弃把vue.js的自动化流程写到gulp里。而是采用webpack。

最后目录结构是这样的
   
     vue-gulp
    ├── Gulpfile.js
    ├── app.js
    ├── bin
    │   └── www
    ├── node_modules

    ├── package.json
    ├── public
    │   ├── images
    │   │   ├── build
    │   │   └── src
    │   ├── javascripts
    │   │   ├── build
    │   │   └── src
    │   │       ├── components
    │   │       │   ├── app.vue
    │   │       │   ├── b.vue
    │   │       │   └── main.js
    │   │       └── jquery.js
    │   └── stylesheets
    │       ├── build
    │       └── src
    ├── routes
    │   ├── index.js
    │   └── users.js
    ├── views
    │   ├── error.jade
    │   ├── index.jade
    │   └── layout.jade
    └── webpack.config.js
 
Gulpfile.js的内容还是老样子，这是普通前端环境自动化。

webpack.config.js里的内容如下：
    
    module.exports = {
    entry: "./public/javascripts/src/components/main.js",
    output: {
    path: "./public/javascripts/build/",
    filename: "vueApp.js"
     },
    module: {
    loaders: [
      { test: /\.vue$/, loader: "vue" },
    ]
    },
     devtool: 'source-map'
    }
    
主入口文件为main.js，它来包含所有的组件并加到dom里。
  
    Main.js:
    
    var Vue = require('vue')
    var appOptions = require('./app.vue')
    var app = new Vue(appOptions).$mount('#app')
    
app.vue相当于一个组件，b.vue是第二个组件，被app所包含。
    
    app.vue:
    
    <style>
      comp-a h2 {
      color: #f00;
     }
    </style>

    <template>
      <h2 class="red">{{msg}}</h2>
      <comp-b></comp-b>
    </template>

    <script>
    var MyComponentB = require('./b.vue');
    module.exports = {
      data: function () {
        return {
          msg: 'Hello from Component A!' 
        }
      },
        components: {
        'comp-b': MyComponentB
      }
    }
    </script>

    b.vue:
    
    <style>
    .t {
      color: red;
    }

    </style>

    <template>
      <h2 class="t">{{msg}}</h2>
    </template>

    <script>
    module.exports = {
      data: function () {
        return {
          msg: 'Hello from Component B!'
        }
      }
    }
    </script>


之后组件是在components/目录下开发，写完后，用main.js包含进来或者用其他vue加载。

在views/index.jade里面加上#app 就可以确保让组件加到这个ID内。

在layout.jade里面把vueApp.js加载。

这样就完成了自动化流程。

开发开发的时候只要在根目录下 
     
     执行命令
     gulp 
     webpack -w
就可以自由的开发了。

我的vue的自动化流程就是通过Main.js入口来加载一些后缀.vue的组件。

    