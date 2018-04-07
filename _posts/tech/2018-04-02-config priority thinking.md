---
layout: post
title: 学习配置优先思想
category: 技术
tags: [学习,开发,前端,Node,React,React,config]
keywords: 学习,开发,前端,RxJS,React,React,config
description: 
---

## 前言

以前听过配置优先这个概念，但是从未在开发生产环节中运用这种思想。当然那个时候还是菜鸟，一心想着功能实现。最近的工作中接触到了这个，而且在方方面面都运用了这个概念，本着学习总结的目的就有了这篇文章。

## 从路由配置开始

说起来我们前端开发最常接触的配置其实就是各个框架的路由配置，原理我们也不深究，我们就以 react-router为例子。

```

<Router history={history}>
  <Switch>
    <Route path="/" component={App} />
  </Switch>
</Router>

```

其中 `App` 是之前 import 进来的。这就是简单的配置。

我们只要关注一下这个配置还可以怎么分离。以上这个已经是最简模式了，我们考虑的是如何将其独立出来。像 `create-react-app` 是这样组织的:

```

const store = configureStore();
const history = createBrowserHistory();

ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      <Switch>
        <Route path="/" component={App} />
      </Switch>
    </Router>
  </Provider>,
  document.getElementById("root") as HTMLElement
);
registerServiceWorker();

```

看上去没什么不对，但其实它将配置耦合进来了。

而分离的方式是创建新的 configSore

```

import routes from '../routes';
import request from '../middlewares/request';
import rootReducer from '../rootReducer';

const middlewares = [thunk, request];
if (process.env.NODE_ENV !== `production`) {
  middlewares.push(createLogger());
}

const finalCreateStore = compose(
  applyMiddleware(...middlewares),
  reduxReactRouter({
    routes,
    createHistory
  })
)(createStore);

export default function configureStore(initialState) {
  const store = finalCreateStore(rootReducer, initialState);
  return store;
}

```

然后在 `routes` 里面写路由配置

```

import AllListPage from './pages/AllListPage'
。。。

export default(
  <Route path="/">
    <Route path="page/login" component={LoginPage} />
    。。。
        <Route path="*" component={NOTFOUND} />
  </Route>
);


```

可能看上去这个动作多此一举，但这却是配置必要做法。主要就是要将配置文件给独立出来。之后的修改都只通过修改配置文件，然后模块通过遍历配置加载配置，渲染对应的功能。

## 前端配置举🌰

  在前端方面，其实很多东西都能抽离成配置，通过写生成器来完成一些抽象，减轻项目工作量。
  比如我们常用的系统的左边栏功能。像后台系统普遍会有用户管理、系统配置、文章列表等等菜单功能。常规操作就是一个模块一个页面，然后写对应的路由配置，然后需要一一对应的效果(比如面包屑)，还需要额外再写点代码。
  但是如果我们将其抽离，写成的用配置代替繁琐重复的代码应该如何处理？
  能够立刻想到的解决方案肯定就是用组件化来实现。在一个 Menu 组件中传入props，然后动态生成侧边。
  但是如果开始分角色，发现不同角色之前要不同的侧边。配置就很管用了。
  比如一开始我们可能要传很多参数到Props里去，但是如果通过一个简单的函数传入一个字符串自动生成详细的json配置，然后让 Menu 组件自动能够生成是不是会更简单的?尤其在权限判断的时候就不需要一行行添加新的菜单，直接去授权列表里拿对应权限的菜单列表就可以了。
  
  
```

   // 管理员
   
if (userInfo.role === '2') {
  // 生成侧边栏
  menus.push(...tools.getSidebars(
    ['TEMPLATE', 
     'LIST', 'ALL_LIST', 
     'CONFIG']
  ));
}


<Sider
  menus={menus}
/>


getSidebars：

 getSidebars: (arr) => {
    if (!Array.isArray(arr) || arr.length === 0) {
      return [];
    }
    const result = [];
    arr.map((item) => {
      const temp = {
        key: commonConstants[`SIDEBAR_${item.toUpperCase()}_KEY`],
        icon: commonConstants[`SIDEBAR_${item.toUpperCase()}_ICON`],
        title: commonConstants[`SIDEBAR_${item.toUpperCase()}_TITLE`]
      };
      result.push(temp);
    });
    return result;
  }
  
```


之后不管增加几个角色还是几个功能菜单，只要在配置里面填好对应信息，就可以按照这种简洁的添加方式给系统增加新的左边栏。

当然这里还可以考虑增加页面组件的配置，最后一个左边栏对应一个子页面。

前端还有一个很重要的部分可以用配置搞定的事情我们其实以前都做过，甚至也在多个项目里运用，那就是将一些第三方组件根据需求再次封装，通过传入参数，完成部分个性化配置。

比如常用的 `Antd design` 的表格组件，当我们想为利用其加一个搜索功能，加一个筛选条件，加一个跳转其他页面的按钮等等操作，根据业务需求我们可能在其上面加入一些特殊的需求，经过Biubiu的操作功能完成了，我们发现未来可能会需要类似的组件，我们就会想到将数据解耦，通过Props传入，然后通过传入的参数进行选择性的渲染特殊的业务功能。

其实这就是配置优先的一些应用。

比如这里我们再改一下需求，我们需要二级子菜单。该如何操作？

我们需要以下形式：

```

一级菜单
  -- 二级菜单
一级菜单
  -- 二级菜单
  -- 二级菜单
  
```

实现这一的功能我们需要做一些修改

```

const getItemTitle = item =>
  item.icon ? (
    <span><Icon type={item.icon} /><span>{item.title}</span></span>
  ) : item.title;

const getMenus = (menus) => {
  let result = [];
  result = menus.map(item => {
    if (item.sub) {
      return (
        <SubMenu key={item.key} title={getItemTitle(item)}>
          {getMenus(item.sub)}
        </SubMenu>
      );
    } else {
      return (
        <Menu.Item
          key={item.key}
          disabled={item.disabled}
        >
          {getItemTitle(item)}
        </Menu.Item>
      );
    }
  });
  return result;
};


```

主要是做了一个 对 `sub` 存在的判断。

这样配置项从原来的纯数组转为json

```

{
key: 'NORMAL_USER',
sub:[
  {key: 'TEMPLATE'},
  {key: 'LIST'},
]
},

```

然后我们需要改动一下之前的生成器。

```

getSidebars: (arr) => {
    if (!Array.isArray(arr) || arr.length === 0) {
      return [];
    }
    const result = [];
    arr.map((item) => {
      const temp = {
        key: commonConstants[`SIDEBAR_${item.toUpperCase()}_KEY`],
        icon: commonConstants[`SIDEBAR_${item.toUpperCase()}_ICON`],
        title: commonConstants[`SIDEBAR_${item.toUpperCase()}_TITLE`]
      };
      result.push(temp);
    });
    return result;
  },
  
  getAdvanceSidebars: (arr) => {
    if (!Array.isArray(arr) || arr.length === 0) {
      return [];
    }
    const result = [];
    arr.map((item) => {
      const arrResult = {
        key: commonConstants[`SIDEBAR_${item.key.toUpperCase()}_KEY`],
        icon: commonConstants[`SIDEBAR_${item.key.toUpperCase()}_ICON`],
        title: commonConstants[`SIDEBAR_${item.key.toUpperCase()}_TITLE`]
      };
      // 如果有子菜单
      if (item.sub) {
        const subResult = [];
        item.sub.map((subItem) => {
          subResult.push({
            key: commonConstants[`SIDEBAR_${subItem.key.toUpperCase()}_KEY`],
            icon: commonConstants[`SIDEBAR_${subItem.key.toUpperCase()}_ICON`],
            title: commonConstants[`SIDEBAR_${subItem.key.toUpperCase()}_TITLE`]
          });
        });
        arrResult.sub = subResult;
      }
      result.push(arrResult);
    });
    return result;
  }

};

```

对比一下增强版，也就实现了一个对子菜单的遍历。
如果都没有二级菜单，还是依旧像以前那样传递个数组就行。
这里就是要说明我们设计的时候需要做为以后功能扩展做个兼容处理。
写到这里，也突然想到了以前学的设计模式在这里其实能应用很多种。比如适配器模式等等。


## 深入

看到以上可能大家会觉得配置优先其实大概就是给自己组件加一个配置项，加一个生成器，然后通过载入约定好的配置文件(json或者其它形式)，就能够达到缩小代码量的目的。

基于此我们还可以做的更加深入，配置的关键在于抽象，抽象的目的是为了将业务中频繁用到的模块通过处理能够和业务进行解耦，将其作为第三方模块，新函数生成目标对象。

这里我们需要考虑的是将非常常用，在多个项目中都会用到的东西做一个技术层面和业务层面的思考，然后再将其抽象，整理成生成器。因为这类生成器需要很强的抽象，它可以应用在不同版本的框架中。当然这需要很强的开发功力，以及对细节的处理。

比如可以将react中 reducer和 action 做配置优先处理。

可以学习koa中中间件的做法，把很多东西的配置项抽离出来，当做中间件使用。

限于精力就不更加细细展开了。





