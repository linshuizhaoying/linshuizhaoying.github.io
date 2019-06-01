---
layout: post
title: 前端通用组件开发的思考
category: 技术
tags: [总结,开发,前端,通用组件,积累]
keywords: 总结,开发,前端,通用组件,积累]
description: 
---

## 前言

最近重心有部分在开发通用的选择组件。具体背景不详细展开，主要是分享开发通用组件的一些思考。

## 正文

在开发通用组件的时候，是比较迷茫的，因为不知道一开始需要做什么。搭建环境也遇到一些问题，技术栈一开始就设想用`typescript + antd 3.16` 开发。但是一直陷入一个误区就是认为需要把将其用webpack打包成js让其更加通用。因此在开发的时候是花时间准备了一套配置，也能够使用。但是开发过程中会比较麻烦，修改一些内容就需要打包一次。通过静态html引用js的方式很明显是不符合开发效率的。

因此当时是想着要么自己找个现成的项目，将组件以模块的形式开发，开发完再迁移出来。要么直接用cli搭建一个壳子。

先是通过第一种方式开发了第一版。在做文档和example的时候去github搜了一下现有一些组件的做法，才猛然发现，开发组件其实和我们开发模块的步骤其实还是有些不同的。

尤其是开发一种通用组件，我们需要把许多测试例子给完善。因此我们需要工具来帮助我们完成基础配置和基础文档的构建，让我们能专心组件本身。

## 引入包

在第一版开发过程中需要把开发中的组件引用到现有的项目中，直接把目录建立在现有项目是一种方式，另一种更灵活的是利用`npm link`

```

$ # 先去到模块目录，把它 link 到全局
$ cd path/to/my-utils
$ npm link
$
$ # 再去项目目录通过包名来 link
$ cd path/to/my-project
$ npm link my-utils
想去掉 link 也很简单：
$ npm unlink my-utils

```

## 搭建基础


一开始开发的时候还没想法要怎么打包。但是想着先把功能完成。

第三方组件开发阶段的解读网上还是很少，一般都是拍脑袋就把初期的事情定好。

一开始其实是建立了一个目录放代码，一个目录放 build 后的代码。是打算用html来引用打包后的js文件。

但是后来想了想这对开发不利，每次的编译都会降低开发的效率。

因此后来使用的是利用 create react app 来创建一个typescript的开发环境，然后引用一些包，然后在app.tsx引用组件，把第三方组件当做内部的组件进行开发。到时候再迁移出来。

但这个其实有个问题就是，迁移出来的东西还是需要做处理，才能作为独立的第三方组件使用。

后来开发完了，需要做迁移处理了，想了想还是需要有个正规的第三方组件开发的流程，实在没思路我就想到了github上应该有不少第三方组件，只要看看排名高的react 组件 它们是怎么封装怎么暴露组件的就可以照搬。

后来挑选了 `rc-select` 这个组件。

源代码管理差不多，但是它利用了rc-tools 与 storybook的思路让我思路一下子清晰。

照着利用storybook。基本上就解决了整个第三方组件开发流程。

```

├── README.md 
├── examples // 这里直接写引用的例子
├── index.js // 这里暴露组件
├── src // 源代码 这里把所有需要加的类型都写到Index.d.ts中
├── tsconfig.test.json
├── tslint.json

```

关键是引用了 `"rc-tools": “9.4.0”`
配合脚本

```

  "scripts": {
    "build": "webpack --config webpack.config.js --progress --colors",
    "test": "jest ./test/index.test.js",
    "start": "rc-tools run storybook",
    "storybook": "rc-tools run storybook",
    "init-storybook": "rc-tools run genStorybook"
  },

开发阶段直接运行 npm run start 即可。

```

## 冲突解决

在开发通用组件的时候，遇到一个问题，就是新旧 `antd` 样式的兼容问题。因为需要用到这个组件的大部分系统都是旧版的antd。因此需要一套能解决这个问题的方案。

## antd 样式冲突问题

不要直接引入 antd.css

每个组件配对引对应的样式

import Menu from 'antd/lib/menu';
import 'antd/lib/menu/style/css’;

### 第三方组件，不同版本antd 样式兼容方案

在大版本比如 3.8 与 3.16 之间，组件的样式变更非常大，尤其是icon的引用方式与整体的reset都有很大程度的变化。

首先如果直接 引入
`import 'antd/lib/menu/style/css’`
这种类型的会导致icon的不兼容，因为3.8版本的`icon`是`web-focnt` ，而3.9之后的都是`svg`
这样会导致两种情况出现，一种是3.8版本旧项目的`icon`全部消失，这个原因是因`import 'antd/lib/menu/style/css’`这样的写法会自动去载入`reset.css`把之前的覆盖。导致旧项目的`icon`消失

另一种情况是多icon的重叠。

![6783fe72fd7a2864f9205779801cebfc.jpg](http://img.haoqiao.me/blog6783fe72fd7a2864f9205779801cebfc.jpg)

而且还会有个隐藏的问题，即使你很小心的引用了样式，你以为不会覆盖的比如Button之类的
但实际上已经有了冲突

![dcec1bc158641b8ae0cd2baa94cdd4be.jpg](http://img.haoqiao.me/blogdcec1bc158641b8ae0cd2baa94cdd4be.jpg)

![5c4c96261768d3333b0adf1d8a21c850.jpg](http://img.haoqiao.me/blog5c4c96261768d3333b0adf1d8a21c850.jpg)

因此为了处理这些奇奇怪怪的样式冲突问题。

整个兼容方案的核心思路就是 `避免不同版本样式问题，将会产生冲突的样式给独立出来`。

#### 1、将组件需要的样式全部从 antd/lib/组件 单独拎出来
![c7ab59a11e34913f7a0427966a078a07.jpg](http://img.haoqiao.me/blogc7ab59a11e34913f7a0427966a078a07.jpg)

这里可能会有人觉得麻烦，想要 直接用官方的 `import 'antd/lib/checkbox/style/css’;`这种形式

但是请注意 这样的写法会导致让你把 该版本的  初始化样式 也同样包含进来

因为 `import 'antd/lib/checkbox/style/css’`  这目录下的结构是这样

```

   ├── css.js
   ├── index.css
   ├── index.d.ts
   ├── index.js
   ├── index.less
   └── mixin.less
   
```

而 index.js的代码

```

"use strict";

require("../../style/index.less");

require("./index.less”);


```


index.less 的代码里都会包含

```

@import '../../style/themes/default';
@import '../../style/mixins/index';

```

从而导致你想象的引入的纯净的css里面多了很多你不需要的东西，而这些东西就是引起冲突的关键
比如 在3.8 与 3.16 版本，icon的表现形式是不一样的，3.9之后都是svg 导致如果加载了3.16版本初始化样式，会让3.8的所有icon都挂掉。

#### 2、利用新版的 ConfigProvider 将样式前缀全部替换掉

在新版的antd中
ConfigProvider全局化配置

有个prefixCls 用于设置统一样式前缀。

在使用的组件最前面套上后，利用vscode全局替换 .ant

```


 <ConfigProvider prefixCls={'cmSelect’}>。。。
  </ConfigProvider>


```
![66715676cf8d8e90af457142a960e104.jpg](http://img.haoqiao.me/blog66715676cf8d8e90af457142a960e104.jpg)


然后你的样式全部会替换掉。
这样你的组件 所用的样式将会和 antd 区分开
这样旧版本antd项目引入 你利用新版本antd开发的组件，将完全避免样式冲突。

#### 3、处理icon细节

之前讲过 因为3.8版本的icon 是web-focnt ，而3.9之后的都是svg

因此你利用新版本antd开发的组件 里 某些样式里的icon 会出现重叠
比如 notification


这个出现的原因是 `.anticon-close:before`

旧版本的 `.anticon-close:before`里面包含了web-font

新版本的icon直接引入了svg

这样就出现了重复的

解决方法很简单

就是哪里重复就去哪里的样式加上覆盖代码

```

.ant-notification .anticon-close:before {
  content: "" !important;
}

```

这就是为什么要把样式全部提出来，这样可以最方便的进行样式的替换。


## 结尾

这里就零散记录一些思路，实际上自己的笔记本已经总结了整个的开发过程，但感觉还没成体系，就先不放了。

