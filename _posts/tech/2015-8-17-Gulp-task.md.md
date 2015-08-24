---
layout: post
title: Gulp打造前端项目自动化构建流程环境
category: 技术
tags: [Gulp，Gulp学习,Gulp实例,前端学习,前端总结]
keywords: 前端,资料,学习
description: 
---

## 正文

这两天正在学习sass和gulp,Gulp的学习曲线非常平缓，api一看，例子看懂就能直接上手，sass需要以后继续熟悉。现在先简单的来贴学习结果。

### 目的

前端的资源大概分3类
    
    1.js文件
    2.css文件(如果是用sass需要把.scss转成css)
    3.图片等其他文件
    
其实如果把html算上也是一种，但是我一向用Node.js的express，因此好像没什么需要。因此流程应该是这样的：
    
    1.将javascripts/src目录下的js文件压缩然后写入javascripts/build目录下
    2.将stylesheets/src目录下的scss文件转成css，然后补全前缀然后再压缩，然后写入stylesheets/build
    3.将images/src目录下的图片都压缩然后写入images/build

事实上如果需要字体这类，直接放在build目录下即可。

### Css的Gulp写法

    var gulp = require('gulp');

    var sass = require('gulp-sass');
    var sassinput = './public/stylesheets/src/*.scss';
    var sassoutput = './public/stylesheets/build/';

    var sassOptions = {
     errLogToConsole: true,
    outputStyle: 'expanded'
    };
    var minifyCss = require('gulp-minify-css');
    var autoprefixer = require('gulp-autoprefixer');

    gulp.task('sass', function () {
      return gulp
    // Find all `.scss` files from the `stylesheets/` folder
    .src(sassinput)
    // Run Sass on those files
    .pipe(sass())
    .pipe(sass(sassOptions).on('error', sass.logError))
    .pipe(minifyCss())  //执行压缩
    .pipe(autoprefixer()) //自动补全前缀
    // Write the resulting CSS in the output folder

    .pipe(gulp.dest(sassoutput));
    });
    gulp.task('sass:watch', function () {
      gulp.watch(sassinput, ['sass']);
    });


    gulp.task('default', ['sass']);
    gulp.task('default', ['sass:watch']);

### JS的Gulp 写法
js文件压缩后应该价格.min的后缀，因此需要rename

    var uglify = require('gulp-uglify');
    var rename = require('gulp-rename');
    var jsinput = './public/javascripts/src/*.js';
    var jsoutput = './public/javascripts/build/';
    gulp.task('minifyjs', function() {
    	 return gulp.src(jsinput)
        .pipe(rename({suffix: '.min'}))   //rename压缩后的文件名
        .pipe(uglify())    //压缩
        .pipe(gulp.dest(jsoutput));  //输出
	});
	gulp.task('minifyjs:watch', function () {
    gulp.watch(jsinput, ['minifyjs']);
    });
    gulp.task('default', ['minifyjs']);
    gulp.task('default', ['minifyjs:watch']);
    
    
### image的压缩写法
  
看了下gulp的插件，很麻烦的使用方式，还要判断格式，写法在官网的插件有

因为我用图片要么直接就imageoptim，因此省去大图压缩的方式。

还有个插件不错适合小图集合的雪碧图插件。

[spritesmith](https://npmjs.org/package/gulp.spritesmith/)

## 后记
这样整个前端自动化流程就差不多了。之后就是用Bower简单的管理版本依赖。

因为我用Hbuilder来开发，因此只需要打包然后做成模板，之后就可以随调随用。
[参考资料1](http://www.sitepoint.com/simple-gulpy-workflow-sass/)
