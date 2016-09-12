---
layout: post
title: 构建新的web开发环境
tags: [学习,整理,前端,实战总结]
keywords: web,代码,前端,学习总结,ci,单元测试,测试平台
description: 
---

## 前言

前段时间看了不少文章，发现优秀的前端工程师现在都是在github上玩连击啊什么的。。。心里还是向往的，而且感觉现在自己的开发环境好像不太正规(国际化)，因此打算学习并重新构建下自己的开发环境。

## 正文

说到开发环境，现在的感觉是不能只局限于某个项目什么的,应该先把最基础的搞好，比如

```

1.mock server
2.单元测试（对于单元测试我现在有点懵的，因为之前也写过文章介绍，但实际项目里写的很少，然后我发现别人的开源的项目里的单元测试我看不懂QAQ，因此这里的单元测试我只写函数测试和api测试。。。）
3.持续集成(CI)（这个得好好学习一个）
4.配合前端开发的小工具
5.代码质量监控
6.githook

```

而且2，3，6项应该是配合github来完成。

### mock server

因此首先我们来看看`mock server`

这个我之前提到过`jsonserver` 但是这个只适合于只需要单一api功能的项目，但是它不支持文档.然后找了一圈发现不少是国内外的在线工具，最有名的还是阿里的`rap`,然而它的本地配置有点麻烦，仅仅只是一个Mock我觉得还用不到那么大的。而且mock这种轻量级的东西配置到本地才合适。尤其是对于开发环境而言，它越小越好。因此最终我选择了[puer-mock](https://github.com/ufologist/puer-mock) 它非常小，只需要三个文件（当然自己配置需要五个）

```

├── README.md
├── _apidoc.html
├── _mockserver.js
├── _mockserver.json
├── api.js
└── api.json

```
对于基础配置我们可以参考它给出的mockserver文件。然后我自己新建了`api.js`和`api.json`。我们只需要对api.json进行修改就行。具体看文档即可。而且它编写Json的格式可以参考[mockjs](http://mockjs.com/examples.html#Image)

![imgn](http://haoqiao.qiniudn.com/puremock.png)



### 单元测试

要捣鼓单元测试自然需要先来个测试项目，这里我直接用`vue-cli`来快速搭建项目。直接生产的项目里面已经自带`Karma + Mocha`，而且还多了一个`e2e tests with Nightwatch` 然而我们并不需要，因此在配置的时候选择`n`。
项目一开始就已经自带了测试案例我们来看下:

先执行`karma start test/unit/karma.conf.js`

```

import Vue from 'vue'
import Hello from 'src/components/Hello'

describe('Hello.vue', () => {
  it('should render correct contents', () => {
    const vm = new Vue({
      template: '<div><hello></hello></div>',
      components: { Hello }
    }).$mount()
    expect(vm.$el.querySelector('.hello h1').textContent).to.contain('Hello World2333!')
  })
})


```

结果如图

![imgn](http://haoqiao.qiniudn.com/karma1.png)

对于vue而言，作者已经在文档中写明了`代码测试的最佳实践是导出组件模块的选项/函数`
而且在`vue-loader-example`中,作者写的测试案例也是类似差不多，比如:

```

import Vue from 'vue'
import App from '../../src/App.vue'

describe('App.vue', () => {
  it('should render correct contents', () => {
    const vm = new Vue({
      template: '<div><app></app></div>',
      components: { App }
    }).$mount()
    expect(vm.$el.querySelector('.logo')).toBeTruthy()
    expect(vm.$el.querySelector('h1').textContent).toBe('Hello from vue-loader!')
    expect(vm.$el.querySelectorAll('.container').length).toBe(2)
    expect(vm.$el.querySelectorAll('.counter').length).toBe(1)
  })
})

```

作者用了mocha自带的expect.js断言，我们把重要的api罗列一下：


```

.to.be.a，判断参数是否某个“类”的实例，例如expect(5).to.be.a("number")或者expect(5).to.be.a(Number)。
.to.match，判断参数是否匹配某个正则表达式。
.to.contain，判断参数是否包含某个项，调用的是indexOf方法。
.to.have.length，判断数组的长度是否某个特定的值。
.to.be.empty，判断数组是否为空。
.to.have.property，判断一个对象是否含有特定的属性。
.to.have.key，判断一个对象是否含有特定的键。
.to.throwException，判断执行一个方法是否抛出异常。
.to.be.within，判断一个数值是否在给定范围之内，例如expect(1).to.be.within(0, Infinity);。
.to.be.greaterThan和.to.be.lessThan，判断大小关系。

```

更详细的可以看[expect.js官方文档](https://github.com/Automattic/expect.js)

这里我贴一个小tip，当你想直接访问组件里的内容比如data()里面的msg内容，你不能用`tobe`，而是应该这么写

```

expect(Hello.data().msg).to.eql('ddd!')

```

而且经过我浏览了大部分github上他们在vue写测试的时候都是基本上先实例化，然后再配合`querySelectorAll`获取内容，然后匹配是否相等。

而且[vue.js官方论坛](http://forum.vuejs.org/) 关于单元测试也有不少值得借鉴的回答有需求可以去看下。


### 持续集成(CI)

```

持续集成指的是，频繁地（一天多次）将代码集成到主干。
它的好处主要有两个。
（1）快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。
（2）防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。
持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。

from http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html

```

`vue`官方现在是用`CircleCI`来做接入 Github 的持续集成工具，其核心是通过一个脚本，在代码 commit 的时候自动运行继承脚本，完成测试、构建等任务。

一开始接触`CircleCI`的时候我还在纳闷该怎么处理`node_modules`这个文件夹，后来搜了半天没看到答案我就思考是不是一开始我的思路是错的，于是我重新去看了下`vue`在github上的文件，后来发现在`.gitignore`文件里面已经过滤了。

```

.DS_Store
node_modules/
dist/
npm-debug.log
selenium-debug.log
test/unit/coverage
test/e2e/reports

```

#### 准备工作

使用你的Github账户 login https://travis-ci.org/，点击右上角的Accounts到达一个页面，点击其中的Sync account，同步一下Github上公开的仓库。其实Travis-CI的官网已经给出了详细的步奏，接下来就是选择你要持续集成的项目，打开开关，创建配置文件，触发第一次钩子。

![imgn](http://haoqiao.qiniudn.com/ci1.png)

然后选中你要持续集成的项目，让灰色的叉叉变成绿勾。

因为在项目的根目录创建.travis.yml文件，并且书写描述信息。因此我把`mock`文件夹移到了`project`文件夹内。

`.travis.yml`只需要这么写

```

# 语言
language: node_js

# 要测试的nodejs版本
node_js:
  - "5"

# 是否sudo身份
sudo: false

# 运行的脚本，这个脚本必须你本地能跑通，且通过
script:
    - "npm run unit"

```

其中`npm run unit`是在package.json里面的`scripts`中定义

```
    "dev": "node build/dev-server.js",
    "build": "node build/build.js",
    "unit": "karma start test/unit/karma.conf.js",
    "test": "npm run unit",
    "lint": "eslint --ext .js,.vue src test/unit/specs test/e2e/specs"

```

不过要注意一件事情，那就是如果你karma配置文件中没写明 `singleRun: true`那么持续集成将卡住超时。。。然后当你测试没通过时

![imgn](http://haoqiao.qiniudn.com/ci2-1.png)

![imgn](http://haoqiao.qiniudn.com/ci2.png)

而且你的邮箱将会受到一封邮件告诉你集成失败。

因此你这份提交将提交失败。并且在github上可以看到

![imgn](http://haoqiao.qiniudn.com/ci3.png)

这样最简单的集成测试就完成了，但是我们不能满足于仅仅只集成单元测试，我们需要将主流的功能一一涵盖,而我们接下来需要做的很简单。

首先加上eslint的检验,在`package.json`已经有了，因此只需要修改`.travis.yml`

```
# 语言
language: node_js

# 要测试的nodejs版本
node_js:
  - "5"

# 是否sudo身份
sudo: false

# 运行的脚本，这个脚本必须你本地能跑通，且通过
script:
    - "npm run unit"
    - "npm run lint"

```

测试一下，我直接在某段代码前多加个空格，提交编译后会爆出这样的错误

```

ERROR in ./test/unit/specs/test.spec.js
  ✘  http://eslint.org/docs/rules/indent  Expected indentation of 4 space characters but found 5  
  /home/travis/build/linshuizhaoying/web-ci-enviroment/test/unit/specs/test.spec.js:6:6
       expect(Hello.data).to.be.a('function')
        ^

```

然后我们把那个总所周知的图标放到`README.md`，点击 `build/failing` 然后选择`markdown`

![imgn](http://haoqiao.qiniudn.com/ci5.png)

访问github你就可以看到你的编译通过状态

![imgn](http://haoqiao.qiniudn.com/ci6.png)

接下来是代码测试覆盖率。

首先在根目录安装一个包`npm i coveralls --save-dev`

这里有几个小坑需要注意一下，不过首先我们还是打开[Coveralls](https://coveralls.io/) 官网，点击红框中的按钮再打开的页面中选择 `GITHUB SIGN UP`
然后在左边的侧边栏中选择`add repos` 之后就是选中自己的项目为`on`。它会给你一个`token`，你需要在根目录新建一个`.coveralls.yml` 然后里面内容这么填写

```

service_name: travis-ci
repo_token: 你的token

```

回到`.travis.yml`内容添加

```

# 语言
language: node_js

# 要测试的nodejs版本
node_js:
  - "5"

# 是否sudo身份
sudo: false

# 运行的脚本，这个脚本必须你本地能跑通，且通过
script:
    - "npm run unit"
    - "npm run lint"
after_script:
    - "npm run cover"
    - 
```

在 `package.json`中新加scirpt

```

 "cover": "cat ./test/unit/coverage/lcov.info | ./node_modules/.bin/coveralls"
 
```

这里要注意一下,我看到不少github项目里他的覆盖率和单元测试用两个karma配置文件来写，然而我因为用的`vue-cli`直接生成了一份`karma.conf.js`

```

    coverageReporter: {
      dir: './coverage',
      reporters: [
        { type: 'lcov', subdir: '.' },
        { type: 'text-summary' }
      ]
    }
    
```

它在相对于根目录的`test\unit\coverage`目录下新建的`lcov.info`文件。因此需要自己把这个路径加进去。


现在运行`npm run test` 然后 `npm run cover`，可以实现自动上传代码测试覆盖率报告。

为了确定代码测试覆盖率正确性，我新建了一个`Linshui.vue` 并且在`App.vue`中加载了这个组件。

现在运行`git add .` `git commit -m "ci cover"` `git push -u origin master`

然后再coveralls官网可以看到

![imgn](http://haoqiao.qiniudn.com/ci%20cover.png)

往`README.md`中添加

```

[![Coverage Status](https://coveralls.io/repos/github/linshuizhaoying/web-ci-enviroment/badge.svg?branch=master)](https://coveralls.io/github/linshuizhaoying/web-ci-enviroment?branch=master)

```

这样你就能看到你的代码测试覆盖率了。

![imgn](http://haoqiao.qiniudn.com/ci%20cover2.png)

关于跨浏览器集成测试我就不折腾了。。。原因就是折腾了很久也没有弄出个结果QAQ


### 配合前端开发的小工具

`PageSpeed`chrome套餐,都是在开发者应用中心搜索下载，具体效果如下：

![imgn](http://haoqiao.qiniudn.com/pagespeed1.png)

![imgn](http://haoqiao.qiniudn.com/pagespeed2.png)

这两者在开发完成后可以针对性的进行优化~

`wappalyzer` 这个chrome插件可以让你知道目标网站加载了啥，当初配这个是想偷师来着。。。后来发现自己已经习惯直接翻人家源码找东西了。。。

![imgn](http://haoqiao.qiniudn.com/wazzper1.png)

`alloydesigner` 绝对的神器，帮助你完成1px的设计稿完美对准

![imgn](http://haoqiao.qiniudn.com/alloydesigner.png)

### 代码质量监控

这里需要提到一个网站`codeclimate.com` 它可以帮你审查你github上的项目的代码质量，如图

![imgn](http://haoqiao.qiniudn.com/codejsclimate.png)

![imgn](http://haoqiao.qiniudn.com/codecssclimate.png)

而且能帮你分析整站的重复代码以及给出一些建议。但是关键是它只有14天的免费。。。然后就是20美元一个月...

不过官方有个本地搭建的方案有兴趣可以去尝试。






