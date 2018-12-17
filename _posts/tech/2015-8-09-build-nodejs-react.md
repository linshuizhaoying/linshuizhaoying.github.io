---
layout: post
title: 建立Node.js+Express4.x+React+Gulp的开发环境
category: 技术
tags: [react，react学习,Node.js,Express,Express4.0,Gulp]
keywords: 开发环境搭建
description: 
---



## 正文
俗话说，工欲善其事，必先利其器。一个好的开发环境必将带来效率的成本提升。React的开发环境多种多样，配合不同的环境有不同的写法。由于我之前在学Node.js，对Node.js的Web框架Express恋恋不舍，因此google了很久才找到想要的东东。

## 开始搭建
开头第一句应该就是很熟悉的
     
     express env
     cd env
这样就建立了一个env的目录，并切进去

由于我们已经有了node.js应用，我们就继续添加相应的工具来将jsx脚本编译成标准JavaScript。接下来有两种选择，一种是手动编译，一种是自动化。
    
    提起自动化，我很久以前就听过这个，但是一直觉得没啥用，因为自己当初水平比较低，写的东西都是静态居多，或者加个jqeury插件就差不多了，但是随着知识的积累，一个效率的开发应该是部署到线上能稳定的运行，并且它所占用的资源应该尽可能少。这需要压缩图片，js,css。以及打包发布等等过程。

### 手动编译

我们需要安装react-tools这个node包，使用如下命令：

	$ npm install react-tools --save-dev
	
现在我们可以使用这样的命令将jsx脚本编译成标准JavaScript：

	$ ./node_modules/react-tools/bin/jsx public/javascripts/src/ public/javascripts/build/
	记得使用-x标志，因为我们使用.jsx作为代码文件扩展名

这样的缺点很明显，每次你写完之后都需要重新编译。
### 自动化构建	
Gulp是一个非常棒的自动化构建工具，而且它基于Node.js，这样就显得很亲切。而且它的写法非常的优美，简洁明了，加上社区的强大，强有力的插件支持，使得自动化构建非常简洁。对于我们的目标来说我们首先需要这么做：

	npm install --save-dev gulp
	npm install --save-dev react browserify reactify vinyl-source-stream
这两条命令一条安装Gulp，一条安装Gulp插件，记得在执行这两条之前执行
    
    npm install gulp -g
全局安装Gulp    
    
browserify是一个工具，可以将node模块编译成浏览器可执行的commonjs模块。
为了开发方便，需要在public/javascript/目录下面建立
	  
	/build
	/src
这两个目录，然后在src里建立app.jsx，里面的内容如下:

    var React = require('react');
	var HelloWorld = require('./helloworld.jsx');

	React.render(
    	<HelloWorld />,
    	document.getElementById('example')
	);
这就是我们主程序的入口，之后我们删除一些express脚手架建立的多余的内容，比如app.js中users部分，还有routes目录下users.js。
看app.jsx里面的代码，我们可以发现，组件的编写已经继承了Node.js的写法，把组件全部单文件化，这样需要哪个加载哪个。之后我们在src里面建立helloworld.jsx，里面内容如下:
    
    var React = require('react');

    module.exports = React.createClass({
     render: function() {
        return (<h1>Hello, world from a React.js linshuizhaoying!</h1>)
        }
	   });
之后回到根目录，我们可以开始自动化构建了，新建一个gulpfile.js,里面内容如下：
    
    var gulp = require('gulp');

	var browserify = require('browserify');
	var reactify = require('reactify');
	var source = require('vinyl-source-stream');

	gulp.task('js', function(){
    browserify('./public/javascripts/src/app.jsx')
        .transform(reactify)
        .bundle()
        .pipe(source('app.js'))
        .pipe(gulp.dest('public/javascripts/build/'));
	});

	gulp.task('watch', function() {
       gulp.watch("public/javascripts/src/*.jsx", ["js"])
	});
	gulp.task('default', ['js']);
	gulp.task('default', ['js', 'watch']);

先是加载需要的插件。gulp.task建立一个gulp的任务，然后
    
    browserify('./public/javascripts/src/app.jsx')

传入入口文件的路径。
    打包之前有jsx代码，因此我们需要用reactify来翻译因此用到了
    
    .transform(reactify)
然后bundle打包，然后输出app.js到bulid目录下。

下面的watch是监视目录下jsx文件的改动，如果有改动，那么就重新打包刷新。
其实还可以压缩文件啊什么的，但是由于我们只是示例，这些以后正式部署再提。
直接执行 gulp 命令会报错，因为我们之前执行的
    
    	npm install --save-dev react browserify reactify vinyl-source-stream
它其实是残缺的，还需要我们自己手工把缺失的模块npm install，不过我已经把整个项目打包并上传到github上了，需要的可以直接下载 [下载地址](https://github.com/linshuizhaoying/Node.js-Express4.x-React-Glup)
之后我们可以运行supervisor ./bin/www 来看我们建立的项目已经成功运行了。



That's all.
![pic1](http://img.haoqiao.me/n2.png?imageView2/2/w/600/h/600/q/100|watermark/2/text/Qnkg5Li05rC054Wn5b2x/font/5a6L5L2T/fontsize/500/fill/IzAwRkZGRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
