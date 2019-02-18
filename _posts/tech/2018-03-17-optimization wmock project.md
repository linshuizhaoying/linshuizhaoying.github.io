---
layout: post
title: 记某项目的收尾工作 - 实战篇
category: 技术
tags: [学习,开发,前端,Node,React,React,Mock,重构,typescript,rxjs,redux]
keywords: 学习,开发,前端,RxJS,React,React,Mock,重构,typescript,rxjs,redux
description: 
---

# 前言

三月份想了想自己干了好多事情，但是归类一起还是在赶毕设以及写论文(悲伤/(ㄒoㄒ)/~~)

因此本篇文章就是作为一次收尾的一些思考。

# 首先，还是解决Bug吧

我曾经以为项目经验多了以后，设计的结构就会合理，Bug会少。后来我发现项目经验多了，提高的是我解决Bug的能力QAQ

首先一些交互的小Bug不提，直接来怼曾经瞎设计的自己。

因为项目一开始采用Mongodb，当初设想是单独表加外键字段。

比如 Team 表我可以这么设计：

```

_id: stirng,
teamInfo ...
projectId
member:Array...
masterId:string...

```

后来写API接口文档的时候我发现team需要的数据还要有用户头像。当时脑抽了一下想着要不走个捷径，直接加个字段(事实证明这种捷径从来都是坑自己)

于是字段变成了

```

_id: stirng,
teamInfo ...
projectId
member:Array...
masterId:string
masterAvatar:string

```

这里是其实犯了一个很大的错误就是我前端开发久了，想着前端返回数据需要那么多字段，那么我后端设计的时候尽量往这方面靠拢。这样我就只查询一次就能拿到数据。
这个给后面我最终测试的时候带来了一些麻烦。
首先就是当用户更新个人信息的时候，它头像是变动的。
而我在team表里写死的，这样会造成不同步。
当时我的思考是就增加一个需要的字段需要多查询一次很麻烦，却没考虑清楚整个需求其实还有点小漏洞。
因此这里需要把bug补上。

首先 team 这个表其实一开始有个设计就是利用mongoDb的 ref 来保存动态的 member。现在回过头想想我当初都能用ref来引用一个数组，多引用一个字段其实并没有增加多少工作量。。。

这里还需要多做一个考虑是因为我现在优化的后端代码，前端其实是已经Ok了的，那么我返回的数据是最好还是原来的字段，不能做改动。

因此我们重新设计 Team 表

```

const ObjectId = mongoose.Schema.Types.ObjectId;
const stringId = mongoose.Schema.Types.String;
  masterId: { type: String },
  role: { type: String },
  masterName: {
    type: stringId,
    ref: "User"
  },
  masterAvatar: {
    type: stringId,
    ref: "User"
  },
  projectId: { type: String },
  projectName: {
    type: stringId,
    ref: "Project"
  },
  member: [
    {
      type: ObjectId,
      ref: "User",
      default: []
    }
  ]
});

```

从上面的更改中，将只需要字符串的用 stringId 表示，返回整个对象的是 objectId.

然后我们要将之前的那些插入团队名称等等字段的内容全部改成插入用户id(顺便也改造了一下project)

```

  team.masterAvatar = userData._id;
  team.masterId = userData._id;
  team.role = userData.role;
  team.masterName = userData._id;
  team.projectId = result;
  team.projectName = result;

```

之后是回到 controller 里面对数据库的查询，我们需要利用到 mongoDb 提供的 `populate` 方法。

```

export const FindTeamByProjectId = async (id: string) => {
  const result = await Team.findOne({ projectId: id })
    .populate("member")
    .populate({ path: "masterAvatar", select: "-_id avatar" })
    .populate({ path: "masterName", select: "-_id userName" })
    .populate({ path: "projectName", select: "-_id projectName" });
  result.masterAvatar = result.masterAvatar.avatar;
  result.masterName = result.masterName.userName;
  result.projectName = result.projectName.projectName;
  return result;
};

```

这里有几个点，一个是通过 selete 参数我们将默认传进来的 `_id` 去掉，然后通过 populte 自动将关联的字段返回给查询的内容。

当然这还不够因为我们最终 populte 返回的是一个对象，它类似 `{avatar: '...'}` 而我们之前约定的接口名称是 masterAvatar， 需要做个简单的处理然后再返回。以此类推，将剩余的内容都按照这个处理。

当然整个核心流程是这样，具体操作还是比较复杂，需要删表测试，再删再测试。

然后就是将之前写死的这些动态更改的字段全部替换掉。

当然我们之前替换的单个查询 findOne

如果遇到 需要返回整个列表的情况，我们需要做后续处理

```

export const AllDocument = async () => {
  return await Document.find()
    .populate({
      path: "ownerName",
      select: "-_id userName"
    })
    .exec((err: any, documents: any) => {
      documents = documents.map((document: any) => {
        document.ownerName = document.ownerName.userName;
      });
    });
};

```

通过对返回的数据再加工，成为符合api接口文档格式的内容。

最大的问题解决之后，就是对一些逻辑不严谨的地方增加判断，比如用户修改名称的时候需要做是否重复的校验等等，这些都是在测试中找到后可以直接修改的。

因为后端我设计了以下的结构

```

router 路由调用对应 service 函数

service 处理逻辑

controller 数据处理

model 数据模型

```

所有逻辑判断，业务处理都在 servic层，controller 只需做数据操作处理即可。因此维护起来还是很顺手的。

# 然后我们考虑一下优化

首先是 利用工具 `whyDidYouUpdate`

使用方式就是直接在你的主页面入口 `index.js/tsx` 里加入

```

if (process.env.NODE_ENV !== "production") {
  const { whyDidYouUpdate } = require("why-did-you-update");
  whyDidYouUpdate(React);
}

```

这里发现因为之前一直写代码的时候有做重复校验，没检查出来问题。

因为没有明显的卡顿，主要的优化其实应该是在于加载的时候这个速度会比较慢，原因是打包下来的代码即使压缩了还是很大，需要做按需加载。

这里就不展开，到时候有空再开个专题。









