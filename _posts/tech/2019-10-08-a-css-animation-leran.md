---
layout: post
title: 提效小专题 v1.0
category: 技术
tags: [总结,开发,前端,开发, 提升效率]
keywords: 总结,开发,前端,开发, 提升效率]
description: 
---

## 前言
 
最近开发没什么总结的东西，原本想忽悠一个学习动效的经历，后来看了看源码败退。

为了输出一点不水的内容，想了想还是从提升开发效率这件事情说起吧。

## 正文

提升效率的手段肯定是要从解决问题开始。

我们面临的问题很多都是归结为： 是什么让我们一直在重复？

首先设定一个场景，比如新建一个新项目, 我们一般情况下如果依赖流行框架，首选的肯定是官方Cli，但是也可能是野路子的Cli，但是不管是什么我们需要一键创建项目开发环境。

但是在比较定制化的开发环境，我们一般会拷贝上一个项目的目录，然后删删改改捣鼓出新的开发环境。

这在之前的文章提到过利用`node` 和 `commander` 开发出小工具来提升项目环境的迁移。

但这个适合于你多个项目是同一套写法，如果是不同的，或者说需要增改的，需要换配置换插件换框架，那么就无异于重新在搞一份环境。

这种情况是很难避免的，但是开发环境的避免重复只是提效的一部分。

在开发过程中更多的是代码片段的复用。一般情况下`vscode`的`snippet`已经够用，或者搭配`atext`这种全局的快捷片段插入，也能提高我们的效率。

这种针对单个文件或者函数的重复是我们日常中开发用到最多的。也是比较好解决的。

那么还有什么会重复？

模块的重复是不怎么好解决的，因为它可能是变动大的，有时候也可能变动小。

比如开发某个项目，会遇到多个高度相似的流程与交互，尤其是开发后台管理系统的时候。

常见的我们会遇到

![imgn](http://img.haoqiao.me/blog661570707211_.pic_hd.jpg)

典型的表格交互，会需要查询过滤条件，会需要分页查询，会需要各种基础按钮交互，而且操作可能会需要编辑状态，需要删除等等功能。

它的变形交互如下

![671570707494_.pic.jpg](http://img.haoqiao.me/blog671570707494_.pic.jpg)

高度相似的交互与业务逻辑，面对这种情况，我们需要提取一份公共的母版出来，这个母版不需要高度抽象，毕竟抽象很容易造成一种情况，就是需要考虑各种方面，以至于捣鼓到最后四不像。而如果是一个已经开发好的模块就不会有这样的问题，我们只需要考虑如何更好地扩展以及迁移这份模块。

我们需要的母版就是一种利用关键词，比如`TestExample`作为命名前缀的功能模板。用到时候整个目录拷贝到对应位置，然后利用IDE的批量替换前缀即可。

这里需要注意的是每个功能模块记得有个开关开启或者说把对应逻辑独立出来，需要时开启，用不到就可以关闭甚至删除对应代码。

它可以节约大量的构造原型以及对接接口的时间，因为写母版模块的时候，用到的增删改接口与调用方法都是正式环境实践过的，这样大大降低调试时间。

这是针对高度相似交互逻辑的模块。

这种母版模块我们最重要的就是不断去维护，因为第一份写完，总会多出或者少一些功能逻辑，举个例子就是我迁移的第一个表格模板在一开始只有编辑单条项的逻辑，但是后来在开发其他功能的时候用这份母版发现还缺少删除逻辑，就需要开发完后，把功能逻辑补充上去。这里自己考虑通用性，如果不是通用的功能，建议是作为逻辑模块单独存到代码片段库。

如果是没什么相似度的模块，建议是将一些功能逻辑独立到单个文件，比如 表格的展示、带增删改的表格、可以增加和编辑的Modal, 这些都把接口联调的代码耦合。

正因为有了`ant design`这种组件库，我们需要的更多是已经耦合了交互逻辑和接口联调的代码，即时会和需求有不符的地方，但是修改总比新建来的快。


## 平常的积累

在开发时候我们往往会需要面对不同的需求写不同的代码，很多情况下会把一些工具函数给抽离出来作为备用。但是很多情况下，有不少代码都是别人已经总结过的，比如快速生成Id

```

Math.random().toString(36).substring(2)

```

点击文本自动粘贴到剪切板

```

  copyToClipboard = text => {
    // @ts-ignore
    if (window.clipboardData && window.clipboardData.setData) {
      // IE specific code path to prevent textarea being shown while dialog is visible.
      // @ts-ignore
      return window.clipboardData.setData('Text', text);
    } else if (document.queryCommandSupported && document.queryCommandSupported('copy')) {
      const textarea = document.createElement('textarea');
      textarea.textContent = text;
      textarea.style.position = 'fixed';  // Prevent scrolling to bottom of page in MS Edge.
      document.body.appendChild(textarea);
      textarea.select();
      try {
        message.success('已成功复制到粘贴板!');
        return document.execCommand('copy');  // Security exception may be thrown by some browsers.
      } catch (ex) {
        console.warn('Copy to clipboard failed.', ex);
        return false;
      } finally {
        document.body.removeChild(textarea);
      }
    }
  }

```

这类的代码只要你用到两次就应该把它作为代码片段积累起来，并起一个合适的语义化的名称，方便以后快捷使用。

## 结尾

水完~

