---
layout: post
title:  Browser-sync 自动刷新
category: 技术
tags: [js,设计模式,Browser-sync,vue,基础,积累,Vue.js,Vue,Express]
keywords: 前端,资料,学习,javascript
description: 
---

# 前言

这只是一个工具使用记录。

# 正文

前端开发免不了繁琐的步骤，好在有很多工具能够帮助我们减轻负担。之前自动刷新是有考虑Browser-sync但是当初因为各种奇怪的原因导致安装失败。

现在考虑到没有自动刷新的不方便以及移动端测试的繁琐，因此重新捣鼓了一下。

安装方法还是官网的

`npm install -g browser-sync`

因为开发是基于vue的，原本是打算这样做的

`browser-sync start --server --files "src/*.vue"
`

但后来因为最终结果都是依赖webpack重新打包整理输出为index.App.js

因此又改成

`browser-sync start --server --files "dist/js/*.js
`

再后来考虑到自己的项目是基于express的，于是捣鼓了一下。在执行`supervisor ./bin/www`的时候能够自动监听。就没考虑写在webpack里。而是直接写在`www`这个文件内。

直接看我们修改的内容:

```
var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

/**
 * Create HTTP server.
 */

var server = http.createServer(app);

/**
 * Listen on provided port, on all network interfaces.
 */
var browserSync = require('browser-sync');

server.listen(port,listening);

function listening () {
  browserSync({
    proxy: 'localhost:' + port,
    files: ['../../../dist/**/*.{js,css}']
  });
}

```

对比以前只是加载了browserSync(已经全局安装).

然后加载的listening方法就是官网的配置方法。

之后就可以直接运行`supervisor ./bin/www`即可自动刷新。


