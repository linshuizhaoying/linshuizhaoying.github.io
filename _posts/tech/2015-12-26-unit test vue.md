---
layout: post
title: 前端单元测试的摸索(Vue与Webpack)
category: 技术
tags: [js,mocha.js,should.js,karma,单元测试,进阶,基础,积累,Vue.js,Vue,Webpack]
keywords: 前端,资料,学习,javascript
description: 
---

# 前言
昨天整合了一下基础的单元测试环境，今天将正式工程化。

按照惯例这里放出今天搭建的环境的[github地址](https://github.com/linshuizhaoying/unit-test-vue-webpack)
# 正文

首先我们先确定一下需求，为了直接明了，用Ithoughts作了张思维导图。

![imgn](http://haoqiao.qiniudn.com/unitvue1.png)

看需求我们就知道我们需要解决的是vue文件的解析测试与webpack插件的应用。

因此在昨天的基础上我们先确定下载几个包:

```
package.json新增内容：


    "vue": "latest",
    "vue-loader": "latest",
    "vue-resource": "latest",
    "vue-router": "latest",
    "node-libs-browser": "latest"
    
    "karma-commonjs":"latest",
    "karma-webpack": "^1.7.0",

    "webpack": "^1.11.0",
    "css-loader": "^0.16.0",
    "html-loader": "^0.3.0",
    "extract-text-webpack-plugin": "^0.8.2",
    "style-loader": "^0.12.3",
    
```

然后我们要确定的就是我们不仅仅需要的是单元测试，而且还需要之前的自动化流程。

因此我们还需要将以前搭建webpack的那一套东西拿来。

这里就不赘述了,直接看我们将完成的目录结构：

```
├── README.md
├── coverage
│   └── Chrome\ 49.0.2593\ (Mac\ OS\ X\ 10.10.5)
├── dist
│   └── js
├── karma.conf.js
├── package.json
├── src
│   └── vue
├── test
│   └── linshui_test.js
├── unitTestResult
│   ├── assets
│   └── units.html
└── webpack.config.js

```

相比较上一篇的结果，我们可以发现改动了src下多了一个`vue`的文件夹，根目录下多了`dist`文件夹和`webpack.config.js`的文件。

因为侧重讲单元测试，至于如何搭建webpack自动化流程请看之前的文章或者直接参照github上的文件内容。

其实主要还是`karma.conf.js`这个文件里的配置内容。

```
module.exports = function (karma) {
    karma.set({

// base path, that will be used to resolve files and exclude
        basePath: './',

        frameworks: ['mocha'],
    
// list of files / patterns to load in the browser
        files: [
            {pattern: 'node_modules/should/should.js', include: true}, //对每一个测试文件都要加载should断言
            'src/*.js',
            'test/*_test.js' //这里与昨天不同，这是为了规范测试文件名。
        ],

        webpack:{
          module: {
            loaders: [
              { test: /\.vue$/, loader: 'vue' }, //调用vue的loader文件对vue进行解析
              { test:/\.css$/, loader: 'style!css!less' } //这里只用常规的，如果需要sass插件请自配
            ]
          }
        },

// list of files to exclude
        exclude: [
            'karma.conf.js'
        ],


// use dots reporter, as travis terminal does not support escaping sequences
// possible values: 'dots', 'progress', 'junit', 'teamcity'
// CLI --reporters progress
        reporters: ['progress', 'mocha', 'coverage','htmlalt'],

        mochaReporter: {
          output: 'autowatch'
        },

        preprocessors: {
          'src/vue/*.js':['webpack'], //用webpack插件对这些文件进行预处理
          'src/*.vue':['webpack'],
          'test/*.js':['webpack']
        },

        htmlReporter: {
          outputFile: 'unitTestResult/units.html',
                
          // Optional 
          pageTitle: '临水照影单元测试',
          subPageTitle: '项目描述'
        },



//Code Coverage options. report type available:
//- html (default)
//- lcov (lcov and html)
//- lcovonly
//- text (standard output)
//- text-summary (standard output)
//- cobertura (xml format supported by Jenkins)
        coverageReporter: {
            // cf. http://gotwarlost.github.com/istanbul/public/apidocs/
            type: 'lcov',
            dir: 'coverage/'
        },

// web server port
        port: 9876,


// cli runner port
        runnerPort: 9100,


// enable / disable colors in the output (reporters and logs)
        colors: true,


// level of logging
// possible values: LOG_DISABLE || LOG_ERROR || LOG_WARN || LOG_INFO || LOG_DEBUG
        logLevel: karma.LOG_DEBUG,


// enable / disable watching file and executing tests whenever any file changes
        autoWatch: true,


// Start these browsers, currently available:
// - Chrome
// - ChromeCanary
// - Firefox
// - Opera
// - Safari (only Mac)
// - PhantomJS
// - IE (only Windows)
// CLI --browsers Chrome,Firefox,Safari
        browsers: ['Chrome'],


// If browser does not capture in given timeout [ms], kill it
        captureTimeout: 6000,


// Continuous Integration mode
// if true, it capture browsers, run tests and exit
        singleRun: false,


        plugins: [
            require('karma-mocha'),
            require('karma-chrome-launcher'),
            require('karma-mocha-reporter'),
            require('karma-htmlfilealt-reporter'),
            require('karma-webpack'), //加载webpack插件
        ]
    });
}

```

如果仔细看了上篇文章会发现其实新增的内容不多,只要把握好思路，对其他环境比如react，比如angular等等都可以直接像套模板使用。

然后我们来简单写个测试例子:

```
linshui_test.js:


describe('Vue单元测试', function() {
 	var myComponent = require('../src/vue/app.vue');  
  it('正向测试,msg参数内容应该为linshi!', function() {
    var defaultData = myComponent.data();
    defaultData.msg.should.be.eql("linshui!");
    //console.log(myComponent);
  });
  it('method 内部函数toggle正向测试', function() {
    var defaultData = myComponent;
    defaultData.methods.toggle().should.be.eql("linshuizhaoying");
    //console.log(myComponent);
  });
});


```

然后我们看下我们测试的源文件内容：

```
<style>
body{
  margin:0;
}

</style>

<template>
</template>

<script>

module.exports = {
  data:function (){
    return {
      msg: 'linshui!'
    }
  },
  ready:function(){

  },
  methods: {
   toggle:function(){
     return "linshuizhaoying";
   }

  },
  components: {

  }
}
</script>
```
	
对照样例相信你已经能够写出基本的测试例子了。更多断言可以去should.js官网查看。

# 结尾

其实细心的少年可以看到其实我的思维导图里其实有postcss这项，但是因为现在还没摸熟就没放进去。对于新东西应该会单独写一篇来总结-0-事实上我最后的目标应该还是用Koa来重构我的私人项目,但是最近新技术都没尝鲜，因此可能要拖到2016年了QAQ.

