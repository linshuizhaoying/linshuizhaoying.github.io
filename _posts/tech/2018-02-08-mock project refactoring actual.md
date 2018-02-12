---
layout: post
title: 记某次项目的中期重构工作 - 实战篇
category: 技术
tags: [学习,开发,前端,Node,React,React,Mock,重构,typescript,rxjs,redux]
keywords: 学习,开发,前端,RxJS,React,React,Mock,重构,typescript,rxjs,redux
description: 
---

# 前言

花了一天学习和阅读源码，接下来就是在项目中实践了。

# 实践

## Typescript 重构

撇开其他不讲，我们先来看下现阶段的目录构造

```

── src
│   ├── actions
│   ├── components
│   ├── constants
│   ├── containers
│   ├── epics
│   ├── index.tsx
│   ├── logo.svg
│   ├── reducers
│   ├── registerServiceWorker.ts
│   ├── service
│   ├── store
│   ├── stories
│   ├── typing.d.ts
│   └── util

```

很明显我们需要先对 `typing.d.ts` 进行改造。

改造的思想基于以下几点:

1. 后端采用 Koa + Typescript，所以一些接口的定义是复用的，那么管理它们就必须统一。
2. 需要对命名做统一约定，这样数据的校验会很流畅。
3. 灵活利用泛型。

首先肯定是对 `.d.ts` 做归纳划分。

将外部引用的 module 用 `module.d.ts` 描述。
然后新建 `interfaces` 目录管理接口定义。

然后按照规范将 Number，String，Boolean或Object 改成小写。

从 `actions` 目录开始排查，看到 `dispatch:any` 这种用法，立刻改掉.

```

export function updateDocument(Document: Document) {
  return (dispatch: any) => {
    dispatch(update_document(Document))
  }
}


====>

export function updateDocument(Document: Document) {
  return (dispatch: Function) => {
    dispatch(update_document(Document))
  }
}


```

热更新后测试无报错，那么就全局修改。

接下来是典型的 any 问题

```

const add_document = (data: any) => ({
  type: ADD_DOCUMENT,
  data: data
})

...

const update_document = (data: any) => 

```

`data` 是开发时候为了图方便传入的参数，它可能是Id，可能是一个对象，也可能是字符串，这里不用泛型，只需要将之前定义的接口类型将其替换。增强可读性。

```

const add_document = (document: Document) => ({
  type: ADD_DOCUMENT,
  data: document
})

```

这样依次类推，将语义不明确的 `data` 转为语义明确的传参。

接下来是组件这块，从最基本的 `Loadingbar` 开始:

```

class LoadingBar extends React.Component<any, any> {
  constructor (props:any) {
    super(props);
    this.state = {
      className:'',
      show: true,
      // binding class when it end
      full: false,
      // state to animate the width of loading bar
      width: 0,
      // indicate the loading bar is in 100% ( so, wait it till gone )
      wait: false,
      // Error State
      myError: false,
      loadingerror: false,

    }
  }


```

对于 `React.Component` 和 `props`  都习惯性的用了 `any`。其实只要找到之前传入的参数，只要列出我们常用的，然后写进接口即可。 以上代码我们可以改为:

```


interface LoadingBarProps {
  progress: number,
  error: string,
  onErrorDone: Function,
  onProgressDone: Function,
  direction: string,
  className?: string,
  id?: string
}
interface LoadingBarState {
  show: boolean,
  full: boolean,
  wait: boolean,
  width: number,
  myError: boolean,
  className?: string,
  loadingerror?: boolean,
  progress: number
}

class LoadingBar extends React.Component<LoadingBarProps, LoadingBarState> {
  constructor (props:LoadingBarProps) {
    super(props);
    this.state = {
      className:'',
      show: true,
      // binding class when it end
      full: false,
      // state to animate the width of loading bar
      width: 0,
      // indicate the loading bar is in 100% ( so, wait it till gone )
      wait: false,
      // Error State
      myError: false,
      loadingerror: false,

    }
  }

```

因为 `Loadingbar` 不是自己写的插件，因此修改 `props` 和 `state` 接口定义发现很多错误，因为原作者对 `state` 的滥用，导致各种属性在编译的时候就报错。在把 `LoadingBarProps` 和 `LoadingBarState` 完善的过程中，其实也是将这个插件给修正了一遍。

比较纠结的其实还是 `react` 中的 event 类型。
在 `react` 中 经常会用到 `e.target.value`,但是在 typescript 中各种变化导致后来类型推导的时候各种麻烦。

社区中也有讨论 [Property 'value' does not exist on type 'EventTarget'](https://github.com/ant-design/ant-design/issues/6879)

也看了不少写法 [typescript-input-onchange-event-target-value](https://stackoverflow.com/questions/40676343/typescript-input-onchange-event-target-value)

尝试了一些写法发现还是不对，但我又不能容忍
`Type declaration of 'any' loses type-safety. Consider replacing it with a more precise type, the empty type ('{}'), or suppress this occurrence.`
的报错。

经过仔细研究，在报错信息中推敲最后找到了最终解决方法。

我们只需要引入

`import { ChangeEvent } from 'react';`

然后类型写为 

`ChangeEvent<HTMLInputElement>`

然我们就能愉快的这么写了

```

  changeRole = (e: ChangeEvent<HTMLInputElement>) => {
    this.setState({
      role: e.target.value
    })
    const { dispatch } = this.props;
    dispatch(updateUser(
      {
        role: e.target.value
      }
    ))
  }

```

类型符合语义。

然后是 `antd` 一系列类型的问题，简单的可以通过在官方的 `.d.ts`里寻找，有问题的比如

```

 handleChange = ( info: any) => {
    this.setState({ loading: true });
    if (info.file.status === 'done') {
      this.getBase64(info.file.originFileObj, () => this.setState({
        imgUrl: imgBaseUrl + info.file.response.image,
        loading: false,
      }));
      const { dispatch } = this.props;
      dispatch(updateUser(
        {
          avatar: info.file.response.image
        }
      ))
    }
  }

```

其中的 `info` 用了官方的类型各种报错，搞到后面还是用类似 `any` 的自写接口应付过去了。。。官方问题最为致命。。。

还有比如 `Modal` 组件的 `cancel event` 需要用 `React.FormEvent<HTMLFormElement>` 来匹配。

还有 antd 的 `Table` 组件在 Typescript中使用现在是需要做一些变动的，并不能直接引用使用。经过多次的迭代，现阶段应该是这样的

```

class MyTable extends Table<Team>{ }

const columns: ColumnProps<object>[] = [...]

     <MyTable columns={columns}  ...>
            </MyTable>

```

`Team` 的类型是我们传入的数据类型。

关于一些奇怪的问题比如 `JSX attributes must be on a line below the opening tag`

这些因为参数里面需要 render 新的布局，比如之前这么写

```

  <TreeNode title={
  	<div>
  	...
  	</div>
  } key={project._id} >

```

这在编译的时候就会提醒这么写不行。
我们可以把这个布局给提取出来，然后通过手动渲染的方式来替换

```

 <TreeNode title={this.renderTreeProjectTitle(project)} key={project._id} >

```

至于 `Exceeds maximum line length of 120` 的错误我们可以通过换行将参数下移来优化。

比如原本一行的

`  <Popconfirm title="确定克隆该接口么?" onConfirm={() => { this.cloneCurrentInterface(item._id) }} okText="确定克隆" cancelText="取消">`

换成

```

<Popconfirm
title="确定克隆该接口么?"
onConfirm={() => { this.cloneCurrentInterface(item._id) }}
okText="确定克隆"
cancelText="取消"
>

```

这里也有个小技巧，如果你实在找不到参数的类型，比如在 `antd` 中，我实在无法对 `Menu` 的点击事件做正确的判断，然后我就去 `node_modules` 目录里去翻看它 `Menu` 的 `index.d.ts`，往往能找到正确的定义。

最终从 `complied with warnings` 到 `Compiled successfully!`

![imgn](http://haoqiao.qiniudn.com/WechatIMG125.jpeg)

## 代码 重构

解决重复 Action 中类似的 Success 与 Error，
在开发前期没有考虑把中间件加进去，因此造成开发的时候 Action 里加了很多"脏代码"

```


export function updateDocumentSuccess(msg: string) {
  notification.success({
    message: '更新成功!',
    description: '更新成功!',
    duration: 1
  })
  return
}

export function updateDocumentError(msg: string) {
  notification.error({
    message: '更新失败!',
    description: '更新失败!',
    duration: 1
  })
  return
}
export function removeDocumentSuccess(msg: string) {
  notification.success({
    message: '移除成功!',
    description: '移除成功!',
    duration: 1
  })
  return
}

export function removeDocumentError(msg: string) {
  notification.error({
    message: '移除失败!',
    description: '移除失败!',
    duration: 1
  })
  return
}

......

```

这里我们可以把这些非关键的极其类似的代码统一管理，这里就用到了 redux middleware.

>Redux middleware 被用于解决不同的问题，但其中的概念是类似的。它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。 你可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

当然，按照接下来的思路就是做个 error & success middleware,然后去捕捉对应动作。

但是当我着手的时候我重新回顾了下我现在的代码，发现这样不妥。因为我以及用 rxjs 来对动作做过一层捕捉，然后我也对相应的结果做了更细致的处理。那么我似乎不需要去多此一举了。


```

export const EinvitedGroupMember = (action$: EpicAction) =>
  action$.ofType(INVITED_GROUPMEMBER)
    .mergeMap((action: Action) => {
      return fetch.post(invitedGroupMember, action.data)
        .map((response: Response) => {
          if (response.state.code === 1) {
            invitedGroupMemberSuccess(response.state.msg)
            return nothing();
          } else {
            invitedGroupMemberError(response.state.msg)
            return nothing();
          }
        })
        // 只有服务器崩溃才捕捉错误
        .catch((e: Error): Observable<Action> => {
          return Observable.of(({ type: ERROR_TEAM })).startWith(loadingError())
        })

    });

```

而且我以前定义的信息都是服务端传过来，那么我只需要做一个简单抽象然后更改所有类似的调用就行了。

```

import notification from 'antd/lib/notification';

// 简单的成功和错误处理
export function successMsg(msg: string) {
  notification.success({
    message: msg,
    description: msg,
    duration: 1
  })
  return
}

export function errorMsg(msg: string) {
  notification.error({
    message: msg,
    description: msg,
    duration: 1
  })
  return
}

```

而在后台定义的格式如下

```

// 返回正常数据
export const success = ( data: any, msg: string) => {
  return {
    'state': {
        'code': 1,
        'msg': msg
    },
    'data': {
       data
    }
 }
}
// 返回错误提醒
export const error = (msg: string) => {
  return{
    'state': {
        'code': 2,
        'msg':  msg
    }
  }
}

```

然后通过约定的接口就可以传递显示信息了。

```

export const baseModelList = async (ctx: any) => {
  const result = await BaseModelList()
  return ctx.body = success(result, '获取成功')
}


```



# 总结

通过重构收获还是很多的，首先是对 Typescript 理解更加深刻了，而且明白了如何处理一些奇怪的问题了。

在对组件的 Props 和 State 进行重构的时候，将之前为了快速开发所定义的数据比如之前会这么写

```

this.state = {
  projectMessagesList: ''
}

```

当通过接口定义之后，作为一个数组其实不应该这样置空，而且 Typescript 在我定义好接口后立刻提醒不能这样赋值。改成默认空数组后就解决了。

而且在之前初始化 state 的时候可能会漏掉某个属性，而接口定义后就会告诉你你有哪些属性不存在。

```

message: '类型“Readonly<InterfaceModeState>”上不存在属性“mode”。'

```

代码的可读性其实就是这么一点一点增加的。

以及在修改后，查询某个数据的类型(在 vscode 中)只需要按 ctrl 结合点击该数据就能立刻跳到该属性的定义，这样对开发人员来讲是很方便的一件事情。

![imgn](http://haoqiao.qiniudn.com/project%20refactoring%20actual%201.gif)

如果编译时期出错，在下方的问题中都会直接显示，这样可以在 热更新之前就对错误进行捕获。

重构了大概几十个组件和模块，工作量大是因为之前开发没注意，导致一批类似的问题，然后需要一个个加 接口定义。

重构的意义更多还是提醒自己在开发之前多思考，多想想，不然到后期各种问题，如果一开始逻辑清晰，代码可读性强，那么问题的定位将很方便。尤其在复杂的项目中，能不重构还是尽量不要。最后是写完一个模块就进行检验。遇到"脏代码"的情况下，能尽快解决就尽快，拖到后期免不得看见代码又是懵逼三连。




