---
layout: post
title: 前端效率提升的一些思考（上）
category: 技术
tags: [总结,开发,前端,效率,提升]
keywords: 总结,开发,前端,效率,提升]
description: 
---

# 前言

前段时间，进入了一个项目组支援，其中艰辛过程不提，但是他们在项目里使用的一些提升效率的点我觉得还是可以挖掘并延伸的，结合之前在部门小伙伴分享的一些方法。因此本文主要是讲在实际工作中，如何一步步演进提升我们的效率。

# vscode

关于vscode其实很久之前已经有了一堆插件，然后其它的东西没有特意去折腾。实际上，日常用的工具中每天对着vscode是最多的。

## snippet

代码片段这个之所以提出来，是因为我以前折腾过代码片段，利用一些工具来储存，后来习惯用`atext`来做全局的代码片段管控。

![wechatimg564.png](http://img.haoqiao.me/wikiwechatimg564.png)

但是后来发先这样每次都需要加一个前缀标记，而且在实际项目中其实很少用到结构性的代码片段，而是都是用了git的缩写片段。因此代码片段在之前的项目中应用不多。

但是后来接触这个比较复杂的项目后发现，针对性的代码片段其实是可以起到一些不错的作用，比如当你使用了`styled-components`来写样式，那么可以这么操作:


```

// styled-component;
  "(const styled component with --styled: '') 不导出的常量样式组件, 并使用 --styled 标记;": {
    "prefix": "ss",
    "body": [
      "const $1 = styled(${2:'div'})`",
      "  --styled: '$1';",
      "  $0",
      "`;"
    ]
  },

```

![styled-components.gif](http://img.haoqiao.me/wikistyled-components.gif)

以及利用框架写标准的组件时，可以区分不同类型。比如在写`React`时:

```

 "(stateless function component page) 无状态组件的页面;": {
    "prefix": "sfcp",
    "body": [
      "import * as React from \"react\";",
      "import styled, { css } from \"styled-components\";",
      "",
      "export type $1Props = {}",
      "export const $1 : React.SFC<$1Props> = p => {",
      "  return (",
      "    $0",
      "  );",
      "};",
      ""
    ]
  },
  "(react class component) 单个富有状态的组件;": {
    "prefix": "rcc",
    "body": [
      "export type $1Props = {",
      "",
      "};",
      "export type $1State = {",
      "",
      "}",
      "export class $1 extends React.Component<$1Props, $1State> {",
      "  state :$1State = {",
      "",
      "  }",
      "  render() {",
      "    const s = this.state;",
      "    const p = this.props;",
      "    return (",
      "",
      "    )",
      "  }",
      "}"
    ]
  },

```

![snippet_react.gif](http://img.haoqiao.me/wikisnippet_react.gif)

在特定项目中加载不同的`snippet`能节省不少的时间。

# 自动替换文件名

这个操作是当开发的时候，更改了某个模块的文件名，那么依赖它的模块会自动把引入的名称更改。

这个操作也是在做项目中发现的，它属于vscode中typescript扩展的配置。因为之前也有使用ts开发但是并没有去看这些配置，实际上它的作用很大，因为项目后期开发的时候，如果某个模块需要更改名称，或者更改目录结构，如果一一去改依赖它模块的文件，是一件很麻烦的事情。而且这个功能并不属于冷门，经常会有这样的操作。

![wechatimg281.png](http://img.haoqiao.me/blogwechatimg281.png)


## 保存后根据tslint自动格式化

默认开启即可

```
  
"editor.formatOnSave": true
  
```

# 更多探索

突然发现以前没有仔细去遍历过最新版vscode的配置，实际上深度挖掘一下，肯定能带来不少效率上的提升，因此直接打开vscode的用户配置一个个看过来~

## 将粘贴的内容自动格式化

$![wechatimg574.png](http://img.haoqiao.me/wikiwechatimg574.png)

这个配置默认是关闭的。开启后，将减少去复制粘贴后还要手动格式化代码的步骤。
  
# 关于icon的几种引用方式

寻常开发中，引用切图中的`svg`或者`png`的, 我们会习惯用`css`中的`background`的`url`来引用。

类似这种写法:

```
 background-image: url('../xxx.svg');
 
```

如果是在`react`组件中需要用到类似的，我们也可以这么写：

```

import Xsvg from '../img/xx.svg';

<Img url={Xsvg} />

```

这都挺简单，唯一的确定就是不好统一管理，因为每写一个就需要重新引入地址。

之前就看到一种写法，只需要将地址传给一个组件，然后配置类型即可。这种写法是用到了`styled-components`的一种特性。这里可以看一下简单地`demo`

```

样式内js文件用props去接收
        
        export const RecommendItem = styled.div`
            background: url(${(props) => props.imgUrl});
        `;
        
        react组件内给样式JS文件传入需要的地址
        
        <RecommendItem imgUrl="http://xxxxx"/>

```

# 结尾

提升效率的方法多种，这里先简单的介绍了工具这方面，以及一种组件化的思路。其实更多就是平时多留意一些能够减少我们`DRY`的方式。这样才能在编码的时候稳步提升效率。


 

