---
layout: post
title: 利用StoryBook开发UI组件管理
category: 技术
tags: [学习,开发,前端,React,2018,StoryBook]
keywords: 学习,开发,前端,UI,2018,React
description: 
---

## 前言

最近掐指一算发现本月还有篇技术博文没写~,虽然随便拿一篇日常积累的文章，或者把最近重构的一些点拿出来讲都可以糊弄过去，但是我决定还是搞一点事情。。。

近期就有一个需求是这样的，我手里进行的一个重构项目里，有一些组件我想抽离，给未来其它项目使用，然后我还需要开发两个前端项目，他们有一些共同的组件需求。纯静态页面是不适合的，因为我现在技术栈上了React + Typescipt。我想做到即插即用。顺便把props,state这些东西定义好，以后改一改就能上项目。原本是立了个flag准备自己搞一点东西出来，但是在微信群里，有人扔了一个链接出来[storybook](https://github.com/storybooks/storybook),

粗略一看好像就说我需要的。因此今天目标就是捣鼓一份开发环境。

## 确定需求

>Storybook是UI组件的开发环境。它允许您浏览组件库，查看每个组件的不同状态，以及交互式开发和测试组件。

但是官方github的介绍非常贫瘠，因此建议大家看[Introduction to Storybook](https://blog.hichroma.com/introduction-to-storybook-5aca8cc643f7) 来了解更多。

以及[guide](https://storybook.js.org/addons/introduction/)


我们明确一下我们的需求：

1. 支持载入ant-design等UI库
2. 支持Typescript
3. 支持redux
4. 支持参数调试

## 正式开始

根据思路先创建一个支持ts的react项目

` create-react-app my-app --scripts-version=react-scripts-ts`

然后更新依赖包

`yarn upgrade`

然后按照 storybook

```

npm i -g @storybook/cli
cd my-app
getstorybook

```

之后直接运行`yarn run storybook`就可以看到界面

![imgn](http://haoqiao.qiniudn.com/WechatIMG32.jpeg)

然而事实并没有那么简单。因为支持ts的是项目本身，而storybook是独立出来的。因此你需要按照配置进行各种修改。

首先在`.storybook`目录下建立`webpack.config.js`

里面加载`typescript-loader`

```

// load the default config generator.
const genDefaultConfig = require('@storybook/react/dist/server/config/defaults/webpack.config.js');
module.exports = (baseConfig, env) => {
    const config = genDefaultConfig(baseConfig, env);
    // Extend it as you need.
    // For example, add typescript loader:
    config.module.rules.push({
        test: /\.(ts|tsx)$/,
        loader: require.resolve('awesome-typescript-loader')
    });
    config.resolve.extensions.push('.ts', '.tsx');
    return config;
};

```

然后对`package.json`进行改造

```

{ 
  "name": "my-app",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^16.0.0",
    "react-dom": "^16.0.0",
    "react-scripts-ts": "2.7.0"
  },
  "scripts": {
    "start": "react-scripts-ts start",
    "build": "react-scripts-ts build",
    "test": "react-scripts-ts test --env=jsdom",
    "eject": "react-scripts-ts eject",
    "storybook": "start-storybook -p 6006",
    "build-storybook": "build-storybook"
  },
  "devDependencies": {
    "@storybook/addon-actions": "^3.2.12",
    "@storybook/addon-links": "^3.2.12",
    "@storybook/react": "^3.2.12",
    "@types/jest": "^21.1.2",
    "@types/node": "^8.0.39",
    "@types/react": "^16.0.12",
    "@types/react-dom": "^16.0.1",
    "@types/storybook__react": "^3.0.5",
    "awesome-typescript-loader": "^3.2.3"
  }
}


```

之后最重要的一点是将根目录下的`stories`目录移到`src`目录之下.

里面写入一个`index.tsx`

```

import React from 'react';

import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';
import { linkTo } from '@storybook/addon-links';

import { Button, Welcome } from '@storybook/react/demo';

storiesOf('Welcome', module).add('to Storybook', () => <Welcome showApp={linkTo('Button')} />);

storiesOf('Button', module)
  .add('with text', () => <Button onClick={action('clicked')}>Hello Button</Button>)
  .add('with some emoji', () => <Button onClick={action('clicked')}>😀 😎 👍 💯</Button>);

```

之后再运行`yarn run storybook`就可以实现支持ts语法了。

之后我们需要考虑我们的ui组件该如何组织，通过大量翻看gitHub上的源码，大体上两种方式。
一种是在同名组件下直接添加`.stories.ts`的文件

```

./Button.jsx
./Button.stories.ts

```

一种是`stories`目录下建立index.ts,引用其他组件内容。

我们采取后一种，这是为了方便管理。而且直接在我们现有代码基础上就可以进行。

我们考虑做个demo，现在`react` + `redux` 的demo都是用`todolist`来完成。但是我们这里直接代入一个成熟的`redux`方案。

首先我们看`src`目录下现在的结构:

```

.
├── App.css
├── App.test.tsx
├── App.tsx
├── index.css
├── index.tsx
├── logo.svg
├── registerServiceWorker.ts
├── stories
│   └── index.jsx
├── webpack.config.1.js
└── webpack.config.js

```

很明显 典型`create-app`的结构。
然后我们直接看加入redux之后的项目结构:

```

── src
│   ├── actions
│   │   └── index.ts
│   ├── components
│   ├── constants
│   │   └── index.ts
│   ├── containers
│   │   └── App
│   ├── index.tsx
│   ├── logo.svg
│   ├── reducers
│   │   ├── index.ts
│   │   └── info.ts
│   ├── registerServiceWorker.ts
│   ├── store
│   │   └── index.ts
│   ├── stories
│   │   └── index.jsx
│   ├── typing.d.ts
│   ├── webpack.config.1.js
│   └── webpack.config.js

```

这里还需要注意的是你需要对`tsconfig`做一些整个，而且为了支持Less,我对webpack也做了一些修改。

之后我们写一段简单的`action to reducer`

action:

```

import { INFO_LIST } from '../constants/index'
const saveList = (data: Object) => ({
  type: INFO_LIST,
  data: data,
})

export function infoListRemote () {
  const info = {
    data: {
      item: 'Hello LinShuiZhaoYing',
      cnItem: '你好, 临水照影'
    }
  }
  return (dispatch: any) => {
    dispatch(saveList(info))
    return info
  }
}

```

reducer:

```

import { INFO_LIST } from '../constants';

const initialState = {
   info:''
}

const info = (state = initialState, action: any) => {
  // console.log(action)
  switch (action.type) {
    case INFO_LIST:
      return {
        ...state,
        info:action.data.data
      }
    default:
      return state
  }
}

export default info;

```

App:

```
index.tsx:


import * as React from 'react';
import { Button, Icon } from 'antd';
import { connect } from 'react-redux';
import { infoListRemote } from '../../actions/index';
import './index.css';

class App extends React.Component<any, any> {
  constructor (props: any) {
    super(props)
    this.state = {
      infoList: '',
    }
  }

  componentWillMount() {

  }
  componentDidMount() {
    // console.log(this.props)
  }
  componentWillReceiveProps(nextProps: any) {
    // console.log(nextProps)
    if (nextProps.info) {
      this.setState({
        infoList: nextProps.info.item
      })
    }
  }

  getInfo = () => {
    const { dispatch } = this.props;
    dispatch(infoListRemote())
  }

  render() {
    return (
      <div className="App">
        <div className="test"> {this.state.infoList} </div>
        <Button type="danger" onClick={this.getInfo}> Click Me</Button>
        <Icon type="play-circle-o" />
      </div>
    );
  }
}
const mapStateToProps = (state: any) => ({
  info: state.info.info,
})
let AppWrapper = App
AppWrapper = connect(mapStateToProps)(App);

export default AppWrapper;

```

然后来看下效果图：

![imgn](http://haoqiao.qiniudn.com/storybook2.gif)

可以看到数据已经传递成功。

接下来就是写`stories`,因为我们用了`redux` 所以我们需要用`addDecorator`来包装我们的组件


```

import React from 'react';

import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';
import { linkTo } from '@storybook/addon-links';
import { Provider } from 'react-redux';
import { Button, Welcome } from '@storybook/react/demo';
import { createBrowserHistory } from 'history';
import configureStore from '../store';

import  AppWrapper  from '../containers/App'

const store = configureStore(createBrowserHistory);

storiesOf('AppWrapper', module)
.addDecorator(story => <Provider store={store}>{story()}</Provider>)
.add('empty App', () => <AppWrapper />);

storiesOf('Button', module)
  .add('with text', () => <Button onClick={action('clicked')}>Hello Button</Button>)
  .add('with some emoji', () => <Button onClick={action('clicked')}>😀 😎 👍 💯</Button>);


```

![imgn](http://haoqiao.qiniudn.com/storybook3.gif)

最后一步就是加入参数调试，这里我们需要加在一些`addon` 来增强体验。

我们可以在[More addons](https://storybook.js.org/addons/addon-gallery/)这里看到addons列表。根据需求来增加。然后到对应的git网站上去看用法加入即可。

首先加入`addon-notes`，它用来写组件描述，而且经过测试，它是可以加入Html代码，因此可以先自己定义统一格式，然后加入内容。

![imgn](http://haoqiao.qiniudn.com/E6A65857-3411-49A0-BEEE-2CEBA15C7431.png)

还可以自定义一些信息，比如使用参数，暴露出来的接口等等。加载`Info Addon` 就可以实现。

![imgn](http://haoqiao.qiniudn.com/4A951A01-EBA9-4C5D-94C7-95D10FAA87C3.png)

接下来的核心就是增加参数调试功能

这里给段示例代码:

```

storiesOf('AppWrapper', module)
.addDecorator(withKnobs)
.addDecorator(story => <Provider store={store}>{story()}</Provider>)
.add('knobs App', () =><AppWrapper text={text('Label', 'Hello World')}></AppWrapper>)
.add('with all knobs', () => {
  const name = text('Name', 'Tom Cary');
  const dob = date('DOB', new Date('January 20 1887'));

  const bold = boolean('Bold', false);
  const selectedColor = color('Color', 'black');
  const favoriteNumber = number('Favorite Number', 42);
  const comfortTemp = number('Comfort Temp', 72, { range: true, min: 60, max: 90, step: 1 });

  const passions = array('Passions', ['Fishing', 'Skiing']);

  const customStyle = object('Style', {
    fontFamily: 'Arial',
    padding: 20,
  });

  const style = {
    ...customStyle,
    fontWeight: bold ? 800 : 400,
    favoriteNumber,
    color: selectedColor,
  };

  return (
    <div style={style}>
      I like: <ul>{passions.map((p, i) => <li key={i}>{p}</li>)}</ul>
      <p>My favorite number is {favoriteNumber}.</p>
      <p>My most comfortable room temperature is {comfortTemp} degrees Fahrenheit.</p>
    </div>
  );
});

```

需要在开发的时候把动态传递的参数给设定好。这样才能即时显示。效果图如下：

![imgn](http://haoqiao.qiniudn.com/storybook4.gif)

然后调用`addon options`

在config.js里加入

```

setOptions({
  downPanelInRight: true,
})

```

把横轴的显示板变成竖着的。

还有非常多有意思的`addons`，比如Info的提升版本`readme`.以及一键换背景的`backgrounds`。还有现成的`Material-UI`。还有直接显示你Jsx源码的`Storybook-addon-jsx`.以及控制版本显示的`storybook-addon-versions`，让你直接对比多个版本的区别。一键生成所有截图的`Storybook Chrome Screenshot Addon`。这些社区的`addons`都非常实用。感兴趣可以自己增加。

## 结尾

最后我们已经完美完成了之前的需求，还有了一些意外的惊喜。

根据我这份环境配置，可以自行进行扩展，虽然我现在基于React开发。但是`StoryBook`也是支持Vue的。

最后照惯例放出该工程的[github地址](https://github.com/linshuizhaoying/React-StoryBook-StartKit)


## 参考资料

>[前端组件化开发方案及其在React Native中的运用](http://insights.thoughtworkers.org/front-end-component-develop-and-application-in-react-native/)

>[react-template](https://github.com/ctco-dev/react-template)

>[playbook_ts_sample](https://github.com/m0a-mystudy/playbook_ts_sample)

>[d3-storybook-test](https://github.com/areologist/d3-storybook-test)

>[react-storybook-demo](https://github.com/kadira-samples/react-storybook-demo)

>[Introduction to Storybook](https://blog.hichroma.com/introduction-to-storybook-5aca8cc643f7)

>[StoryBook meets redux](https://medium.com/ingenious/storybook-meets-redux-6ab09a5be346)

