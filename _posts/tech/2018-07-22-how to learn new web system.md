---
layout: post
title: 如何接手一个项目并进行新功能开发
category: 技术
tags: [总结,开发,前端,体系,工程化]
keywords: 总结,开发,前端,体系,工程化]
description: 
---

# 前言

入职的第一个任务是接手一个成熟架构的项目，然后进行新功能的开发。

系统是大部分已经完成，需要做的是先验证之前的代码是否能正常运行。
然后了解整个项目设计，熟悉代码。先初步阅读前端部分代码，再NODE中间层代码。
联通之后再去研究细节流程。
再尝试运行并写新功能。
再写新功能，联调测试。
最后系统测试上线。

# 前端代码

首先了解目录

├── .csscomb.json 
├── .git
├── .gitignore
├── .tslintignore //忽略一些内容
├── README.md
├── _client_src //前端代码
├── _server_src //node端代码
├── common //公共的函数
├── config //一些配置
├── node_modules
├── package-lock.json
├── package.json // 依赖包列表
├── tsconfig.json //typescript配置
└── tslint.json // typescirpt格式验证

这里唯一不熟悉的是 csscomb
查了一下

```

它可以帮助你重新排序CSS中定义的属性，帮助你按照你预定义的排序格式生成新的CSS。会按照你想要的格式定义 css 空格，换行，缩进，代码编写方式。

```

`.csscomb.json `就是一个css的写法规范工具。

整体开发运行需要跑两个命令，分别用于跑本地Node server端，和前端项目的热更新。
之所以不合在一起是因为前端热更新将比较耗时，导致node服务重新启动较慢。

# 从前端代码入手

前端代码处于 _client_src 目录
首先前端项目的技术栈是React15.6 + rxjs5.4 + typescript


├── _WULIComponents // 公共组件
├── base.less // 设定字号，body初始化
├── entities // 实体目录，里面包含了一些配置生成器与配置列表
├── index.d.ts // 标记项目里window对象的类型
├── index.tsx // 文件主入口，加载了路由文件, 判断了热更新，建立了csrf防御。做了加密措施。
├── node_modules // 模块依赖包
├── pages // 前端页面
├── routes // 前端路由
├── routes.tsx // 路由读取并处理
├── tsconfig.json // typescript配置
├── utils // 工具函数库
└── vendor.tsx // 将公共的库在webpack分开打包更快


目录这么多，从核心流程读起。

启动 -> 载入路由配置 -> 载入功能模块

从 `index.tsx` 文件中最后是引入了routes

在routes目录中

```
├── admin_xxx.routes.tsx
...
├── index.tsx
├── login.routes.tsx
├── product.routes.tsx
├── routesSet.tsx
└── welcome.routes.tsx

```


`routesSet.tsx`是最先引入的，它里面只暴露了一个数组

```

export default [
  'login',
  'welcome',
  'admin_account',
  ...
  'product'
];

```

引入之后，做一些处理

```
import routesSet from './routesSet';

let childRoutes = [];
routesSet.forEach(element => {
  const route = require(`./${element}.routes`).default; // 加载同目录下的对应路由配置
  childRoutes = childRoutes.concat(route); // 将其拼接成一个数组
});

export const routes = childRoutes; // 赋值给routes并暴露出去

const routeConfig = [{
  path: '/pages',
  component: App,
  exact: true, // 如果为true，则仅在路径与location.pathname完全匹配时才匹配。
  childRoutes: [
    ...childRoutes,
    {name: 'Page not found', component: PageNotFound, path: '*'}
  ].filter(r => r['component'] || (r['childRoutes'] && r['childRoutes'].length > 0)) // 过滤掉不匹配格式的项
}]; // 生产配置json，其中每个路由配置还只是string，需要处理

const handleIndexRoute = route => { 
  // 处理子路由
  if (!route.childRoutes || !route.childRoutes.length) {
    return;
  }
  
  // 找到index为true的子路由
  const indexRoute = route.childRoutes.find(child => child.isIndex); 
  if (indexRoute) {
    const first = { ...indexRoute };
    first.path = route.path;
    first.exact = true;
    first.autoIndexRoute = true;
    route.childRoutes.unshift(first); // 在对应路由下的子路由前端添加
  }
  route.childRoutes.forEach(handleIndexRoute); // 继续向内检索
};

routeConfig.forEach(handleIndexRoute); // 遍历每一项路由看看有没有子路由
export default routeConfig; // 将完整的路由表暴露出去


```

在看功能模块之前需要对基础工具库做一些了解。

`utils`目录下有三个文件

```

├── debug.tsx
├── http.service.tsx
└── tools.tsx

```

重点关注`http.service.tsx` 因为它是封装了ajax整个流程。

通过引入rxjs，将整个异步请求以流的形式进行控制。

因为流的强大，因此在过程中还将一些提示的ui也加入进来。

比如

```

const success = [
 'background: green',
 'color: white',
 'display: block',
 'text-align: center'
].join(';');

const failure = [
 'background: red',
 'color: white',
 'display: block',
 'text-align: center'
].join(';');

```

关键的是请求操作数据

```

const fetchData = (method, url, data?, options?, errNotification = true) => new Promise((resolve, reject) => {
  let finalURL = url; 
  if (needQuery(method) && data) { // 如果需要query里的数据
    const params = Object.keys(data)
      .filter(key => data[key] || data[key] === false)
      .map(key => encodeURIComponent(key) + '=' + encodeURIComponent(data[key]))
      .join('&')
      .replace(/%20/g, '+');
    finalURL = `${url}?${params}`; // 拼接成最终Url
  }

  // 最终body拼接
  const finalOptions = Object.assign({}, {
    headers: {
      'Content-Type': 'application/json',
      'csrf-token': window['csrfToken'],
      'crypt-key': (window['crypt'] || {}).cryptkey
    },
    credentials: 'same-origin',
    method
  }, options);
  if (needBody(method)) {
    finalOptions.body = JSON.stringify(data);
  }
  // 发送请求
  fetch(finalURL, finalOptions).then(response => {
    if (response.ok) {
      response.json().then(res => {
        if (res.errCode) {
          if (debuggerOn) {
            console.info(`%c ${finalOptions.method} ${finalURL} failed with res ${JSON.stringify(res)}!`, failure);
          }
          if (res.errCode === 401) {
            /** 授权失败 */
            if (!has401Notification) {
              showErrNotification(`登陆过期，正在为您刷新界面`, `操作失败`);
              has401Notification = true;
            }
            setTimeout(() => {
              window.location.href = `/pages/login`;
            }, 2000);
          } else {
            /** 接口报错 */
            errNotification && showErrNotification(`错误代码：${res.errCode}，错误信息：${res.errMsg}`, '操作失败');
          }
          reject(res);
        } else {
          if (debuggerOn) {
            console.info(`%c ${finalOptions.method} ${finalURL} successful!`, success);
          }
          resolve(res.data);
        }
      });
    } else {
      if (debuggerOn) {
        console.info(`%c ${finalOptions.method} ${finalURL} failed!`, failure);
      }
      /** 网络连接错误 */
      errNotification && showErrNotification(`网络连接失败，请检查网络后重新操作`);
      response.text().then(res => reject(res));
    }
  }).catch(err => {
    if (debuggerOn) {
      console.info(`%c ${finalOptions.method} ${finalURL} failed!`, failure);
    }
    /** 前端错误 */
    errNotification && showErrNotification(`错误代码：00000，请联系管理员`, '操作失败');
    reject(err);
  });
});


```

之后就是通过

```

export const http = (method, url, data?, options?, errNotification?) => {
  return Observable.fromPromise<any>(fetchData(method, url, data, options, errNotification));
};

```

通过暴露http方法，将fetch请求的整个异步流程封装。

http 函数就是一个流，只要被订阅，就会执行异步操作，并且将请求结果或者失败原因用流的形式返回。

```

http('GET', '/user', {}, {}).subscribe(data => {
  console.log(data);
}, err => {
  console.log(err);
});

```


当多次发起请求（多次订阅）后会发现，我们之前的订阅没有被释放，造成了资源的浪费。所以我们定义自己的 request 函数如下：

```

export const request = ({
  method, url, data, options, sucCallback, errCallback, errNotification
}: IRequestParams) => {
  const sub = http(method, url, data, options, errNotification).subscribe(data => {
    sucCallback(data);
    sub.unsubscribe();
  }, err => {
    errCallback(err);
    sub.unsubscribe();
  });
};

```


> Observable.fromPromise 将 Promise 转化为 Observable。如果 Promise resolves 一个值, 输出 Observable 发出这个值然后完成。 如果 Promise 被 rejected, 输出 Observable 会发出相应的 错误。

了解了请求是如果发出的，继续了解如何处理缓存这些数据。

在以往都是利用redux这类的库来帮助管理数据。

Rxjs自带了流特性，让组件订阅 Rxjs 的流，并且在流更新的时候刷新界面,可以实现单项数据流。
用 BehaviorSubject 这个类来作为存储的数据流，因为它能实现数据共享。

因为是仿redux的一系列思想写的生成器，因此很多地方都参照对应格式。

`entities`目录下:

```

├── _common
├── account
├── auth
├── dictionary
├── entity_generator.tsx // 关键的生成器 
├── feedback
├── permission
├── product
├── role
└── view

```

生成器的源码：

```

// 定义动作类型。更主要是异步动作，比如http请求

export const actionTypes = {
  CUSTOM: 'custom',
  DATA: 'data',
  VALUE_DATA: 'valueData',
  BOOL_DATA: 'boolData',
  ACTION: 'action',
  ASYNC_ACTION: 'asyncAction'
};

export default (entityName: string, configs: IConfigs) => {
  const result: any = {};

  /** every entity has a common actionLoading */
  // 共同的 loading 初始化为false
  const actionLoading$ = new BehaviorSubject(false).debug(`[${entityName}]:[actionLoading]`); // 
  result.actionLoading$ = actionLoading$;
  // 遍历每个配置
  configs.forEach(config => {
    const {
      /** common */
      type, name, custom,

      /** data, valueData, boolData */
      init,

      /** boolData */
      privateName,

      /** action */
      handler,

      /** asyncAction */
      privateLoading, actionLoading, confirmLoading, bindLoading,
      requestOpts, filter, data, pagination
    } = config;

    // 对每个动作的类型进行判断操作，创建对应的BehaviorSubject
    switch (config.type) {
      case actionTypes.CUSTOM:
        // 如果是custom,说明是让用户自定义流
        result[`${name}$`] = custom(result).debug(`[${entityName}]:[${name}]`);
        break;
      case actionTypes.DATA:
        /** create data Observable */
        
        result[`${name}$`] = new BehaviorSubject(init || false).debug(`[${entityName}]:[${name}]`);
        break;

      case actionTypes.VALUE_DATA:
        /** create data Observable */
        result[`${name}$`] = new BehaviorSubject(init || 0).debug(`[${entityName}]:[${name}]`);

        /** set data operation */
        // 更新对应BehaviorSubject的当前值
        result[`set${toCamel(name)}`] = params => {
          result[`${name}$`].next(params);
        };
        break;

      case actionTypes.BOOL_DATA:
        /** create data Observable. If privateName is true, it'll use ${name} to be the name */
        if (privateName) {
          result[`${name}$`] = new BehaviorSubject(init || false).debug(`[${entityName}]:[${name}]`);
        } else {
          result[`${name}Show$`] = new BehaviorSubject(init || false).debug(`[${entityName}]:[${name}Show]`);
        }

        /** private loading，you can emit data wherever you need */
        if (privateLoading) {
          result[`${name}Loading$`] = new BehaviorSubject(false).debug(`[${entityName}]:[${name}Loading]`);
        }

        /** show operation */
        result[`show${toCamel(name)}`] = () => {
          if (privateName) {
            result[`${name}$`].next(true);
          } else {
            result[`${name}Show$`].next(true);
          }
        };

        /** hide operation */
        result[`hide${toCamel(name)}`] = () => {
          if (privateName) {
            result[`${name}$`].next(false);
          } else {
            result[`${name}Show$`].next(false);
          }
        };
        break;

      case actionTypes.ACTION:
        /** create operation */
        result[`${name}`] = params => {
          handler(params, result);
        };
        break;

      /** async action */
      case actionTypes.ASYNC_ACTION:

        /** private loading，auto emit data in async operation */
        if (privateLoading) {
          result[`${name}Loading$`] = new BehaviorSubject(false).debug(`[${entityName}]:[${name}Loading]`);
        }

        /** create data Observable，auto emit data when request success */
        // 创建数据流，每当它请求成功都自动发送数据
        if (data) {
          result[`${name}Data$`] = new BehaviorSubject(data.init).debug(`[${entityName}]:[${name}Data]`);
          result[`set${toCamel(name)}Data`] = data => {
            result[`${name}Data$`].next(data);
          };
          result[`merge${toCamel(name)}Data`] = (key, data) => {
            result[`${name}Data$`].next(
              Object.assign({}, result[`${name}Data$`].value, { [key]: data })
            );
          };
        }

       。。。
       
       
   return result;
};

```

生成器主要初始化了一些动作BehaviorSubject，根据对应出发条件让其发送数据或者更新当前数据。
等同于redux的action与reducer结合体。

重新回顾
`entities`目录下:

```

├── _common //公共模块，包括loading、toast显示隐藏，页面jump
├── account 
├── auth
├── dictionary
├── entity_generator.tsx // 关键的生成器 
├── feedback
├── permission
├── product
├── role
└── view

```

`account`目录下三个文件

```
├── account.constants.tsx
├── account.entity.tsx
└── index.tsx
```

`index.tsx`代码如下：

```

import generator from '../entity_generator'; // 载入生成器

const ENTITY_NAME = 'account'; // 实体名称
const entities = ['account']; // 实体组合

const result: any = {};
entities.forEach(item => {
  const entity = require(`./${item}.entity`).default; // 引入每个对应实体配置
  Object.assign(result, generator(ENTITY_NAME, entity)); // 通过实体生成器将对应流初始化
});

export default result;

```

查看对应`account.entity.tsx`的内容：

```

export default [
  {
    type: actionTypes.ASYNC_ACTION,
    name: 'accountList',
    // 私有的loading，在每次异步请求时自动发送数据
    privateLoading: true,
    // 请求参数
    requestOpts: {
      method: 'GET',
      url: (params, entity) => `/apis/account`,
      data: (params, entity) => Object.assign({},
        wrapFilterData(entity[`accountListFilterData$`].getValue()), {
          limit: entity[`accountListPageSize$`].getValue(),
          offset: (entity[`accountListCurPage$`].getValue() - 1) * entity[`accountListPageSize$`].getValue() || '0'
        }, wrapFilterData(params))
    },
    filter: {
      init: {}
    },
    pagination: {
      pageSize: 10,
      total: (params, response, entity) => response.count
    },
    data: {
      init: [],
      handler: (params, response, entity) => response.userList
    }
  }
  。。。
];

```

这个文件专门维护动作的配置表，每当需要新的功能/动作，就根据约定写好对应的配置即可。

`account.constants.tsx` 维护了一堆常量。

`entities`的其它目录里的内容都不是独立的，需要结合功能模块来理解，

比如子目录下的`view`就比较复杂？？？

`pages`目录主要包含了所有页面。

```

├── AdminAccount
├── AdminDictionary
├── AdminPermission
├── AdminRole
├── AdminView
├── App
├── FavoriteFeedBack
├── FeedBackInfo
├── Login
├── PageNotFound
├── Product
├── ProductInfo
├── ProductMy
├── ProductSetting
├── Welcome
└── _common

```

首先需要了解公共模块`_common`里都有什么内容:

```

├── components
|  └── ProductList  //封装了一个商品详情的公共组件。
├── decorators
|  ├── injectSubs.decorator.ts // 读取配置中需要载入的订阅源并手工订阅，在组件什么周期结束后取消订阅。
这里是一个高阶函数，因此需要在内外两个组件的生命周期里都进行订阅操作，因此加入了mixinLifecycleEvents,先订阅数据源，再执行各个订阅操作。
|  ├── observer.decorator.tsx // 管理的数据的批量订阅与取消订阅
|  └── saveRouterProps.decorator.tsx // 实现了一个高阶组件，用于将路由里的参数进行保存
├── selectors
|  └── common.selector.tsx // 引入了之前的common实体，将一些弹框类型的动作进行了封装。
└── utils.tsx // 实现了一些查询函数与权限判定函数。

```

`utils.tsx`的代码如下：

```

const {
  currentTab$
} = view;

export const getLatestDataByProductId = (target: any) => {
  return Observable.combineLatest(target, currentTab$,
    (targets, currentTab: IcurrentTab) => targets[(currentTab.params || {}).productId]
  );
};

export const getLatestDataByFeedbackId = (target: any) => {
  return Observable.combineLatest(target, currentTab$,
    (targets, currentTab: IcurrentTab) => targets[(currentTab.params || {}).feedbackId]
  );
};

export const displayActionByAuthority = (wrap: any, code: string) => {
  const currentValue = view.currentTab$.getValue();
  if (!currentValue.hasOwnProperty('params')) {
    return '';
  }
  const { productId } = currentValue.params;
  const productRole = product.productActionData$.getValue();
  const IsDisplay = (productRole[productId] || []).includes(code);
  return IsDisplay ? wrap : '';
};

export const IsAuthority = (code: string) => {
  const currentValue = view.currentTab$.getValue();
  if (!currentValue.hasOwnProperty('params')) {
    return false;
  }
  const { productId } = currentValue.params;
  const productRole = product.productActionData$.getValue();
  const IsDisplay = (productRole[productId] || []).includes(code);
  return IsDisplay;
};

```


了解之后需要定位到`App`，程序入口，看看都做了哪些动作

```

├── containers // 组件化
|  ├── Confirm
|  ├── Content
|  ├── Footer
|  ├── Header
|  ├── Notification
|  ├── ResetPsdModal
|  └── Sider
├── index.tsx // 入口文件
└── style.less

```

`index.tsx`源码:

```

import * as React from 'react';
import './style';

import { getValue, jsonParse, getQuery } from 'tools';
import auth from 'e_auth'; // 引入权限校验实体
import view from 'e_view';
import common from 'e_common';

import Header from './containers/Header';
import Footer from './containers/Footer';
import Sider from './containers/Sider';
import Content from './containers/Content';
import Notification from './containers/Notification';
import Confirm from './containers/Confirm';
import ResetPsdModal from './containers/ResetPsdModal';

/** 是否需要还原用户 tab 数据 */
const NEED_RESTORE_TAB_INFO = true;

class App extends React.PureComponent<IProps, IStates> {

  componentDidMount() {
    common.hideNotification();
    common.hideToast();

    this.init();
  }

  init = () => {
    if (getValue('userInfo')) {
      const userInfo = jsonParse(getValue('userInfo'));
      auth.setUserInfo(jsonParse(getValue('userInfo')));

      /** 获取存储的用户 tab 数据 */
      const id = userInfo.id;
      const restoreTabs = (jsonParse(getValue('restoreTabs')) || {})[id];
      const restoreCurrentTab = (jsonParse(getValue('restoreCurrentTab')) || {})[id] || {};

      if (restoreTabs && NEED_RESTORE_TAB_INFO) {
        view.setTabs(restoreTabs);
        view.switchTab(restoreCurrentTab.mapping ? restoreCurrentTab : restoreTabs[0]);
      }

      this.setSystemListAndLoadViewList(restoreTabs);
    }
  }

  /** 设置系统列表，获取系统的可见视图列表，并加载已存储 tab 的模块和按钮权限 */
  setSystemListAndLoadViewList = restoreTabs => {
    if (getValue('systemList') && !(window.location.pathname === '/pages/login')) {
      auth.setSystemList(jsonParse(getValue('systemList')));
      /** 只有一个系统的情况 */
      view.viewList({
        code: jsonParse(getValue('systemList'))[0].code,
        sucCallback: (params, res) => {
          if (restoreTabs && NEED_RESTORE_TAB_INFO) {
            restoreTabs.forEach(tab => {
              if (this.isInViewList(res.viewList, tab.id)) {
                view.viewPermissionList({ id: tab.id });
              }
            });
          }
        }
      });
    }
  }

  /** 判断一个 tab 是否属于可见视图列表 */
  isInViewList = (viewList, id) => {
    const finalViewList = [];
    const pushToViewList = view => {
      finalViewList.push(view.id);
      view.viewViewTreeSet.forEach(item => pushToViewList(item));
    };
    pushToViewList(viewList[0]);
    return finalViewList.includes(id);
  }

  isLoginPage = () => {
    return location.pathname === '/pages/login';
  }

  render() {
    return (
      <div className="app-layout">
        <div className="header-layout">
          <Header />
        </div>
        <div className="container">
          <div className={`container-layout w100 ${this.isLoginPage() ? '' : 'mw'}`}>
            <div className="sider-layout">
              <Sider />
            </div>
            <div className="content-layout">
              {this.isLoginPage() ? this.props.children : <Content />}
            </div>
          </div>
        </div>
        <div className="footer-layout">
          <Footer />
        </div>
        <Notification />
        <Confirm />
        <ResetPsdModal />
      </div>
    );
  }
}

interface IProps {
  location: {
    search: string;
  }
}
interface IStates { }

export default App;

```

首页是根据登录后从服务端拿到的系统列表，根据权限等信息，从缓存中读取存储的tabs数据，再通过`view`的实体流来发起UI更新。


## 需要了解更深入的其他流程

### 整个登录退出的流程，包括路由的状态

整个登录退出的核心流程包括：

1. 用户访问页面，未登录情况跳转到登录页面。
2. 输入账号密码，验证。
3. 登录成功跳转。
4. 点击退出，跳转回登录页面。

第一步的跳转是需要做服务端验证的，因此路由的控制在Node层的`page/controller`
它先进行身份的校验，如果不符合直接重定向到登录页面，附带上登录前的来源。

```

if (needToken && !isExist) {
    return ctx.redirect(`/pages/login?next=${encodeURIComponent(ctx.originalUrl)}`);
  }

```

第二步分两块，当用户输入之后，页面会做简单的校验，这部分代码是写在`pages/Login`

该目录的结构如下:

```

├── containers
|  └── LoginForm
|     ├── LoginForm.component.tsx // 配置提示项，登录UI界面
|     ├── LoginForm.decorator.tsx // 高阶组件，在componentDidMount中初始化订阅登录流。在componentWillUnmount中取消订阅。
|     ├── LoginForm.selector.tsx //  登录加载流的配置
|     ├── index.tsx // 将decorator与component组合暴露
|     └── style.less
├── index.tsx // 主要是加载LoginForm组件
└── style.less

```

在 `LoginForm.decorator.tsx` 中，调用了auth entity的login接口。
在 auth 验证登录成功后，将后台传来的关于菜单的权限列表写到本地的 `userinfo` 和 `systemlist`,接着返回的数据传给对应流。然后再去校验视图ID

```
     sucCallback: (params, res, entity) => {
        /** 登陆后设置用户信息和系统列表 */
        setValue('userInfo', JSON.stringify(res.account));
        setValue('systemList', JSON.stringify(res.systemList));
        entity['userInfo$'].next(res.account);
        entity['systemList$'].next(res.systemList);
        view.viewList({
          code: res.systemList[0].code
        });
      }
      
```

当成功跳转到 `App` 对应的路由，而且已经有登录标志的情况下，`App`之前缓存的`userinfo` 和 `systemlist` 分别读出来，并且根据配置渲染对应的UI。

当用户点击退出的时候。触发`auth`的logout，这里前端就直接跳转到了`login`页面，接着将页面的`tabs`都清空，删除本地缓存的`restoreTabs`与`restoreCurrentTab`。本地保存着两个是为了让用户刷新的时候还能够定位到原先之前的操作位置。
最后是还原`currentTab`为配置初始化的值，让下次登录的时候是最初的欢迎页面。

路由的控制可以从整个流程看到，只有在未授权的情况下是根据了Node服务器返回的重定向，其他授权的跳转操作都是前端来控制的。
也就是在每个有异步交互动作执行的成功，都经历了权限的校验。


### 功能模块的tabs选择，删除，点击已存在更新状态，点击新的创建。

这个交互是做了很细的颗粒度处理。
包括每次点击都将用户的操作存到`localstorage`
然后让各个流都更新数据，让UI也随之更新。

其中涉及实体中`view`里的配置:

```

 { // 当前选择的，展示中的view，非编辑中的view
    type: actionTypes.VALUE_DATA,
    name: 'editView',
    init: {}
  },
  {
    type: actionTypes.VALUE_DATA,
    name: 'editingView',
    init: {}
  },
  { // 当编辑中的view，编辑状态下：与选中的editView是同一个，增加子视图状态：parentId 为editView.id
    type: actionTypes.BOOL_DATA,
    name: 'viewEditModal',
    privateLoading: true
  },
  
```

以及在`App`的UI更新

```

if (getValue('userInfo')) {
      const userInfo = jsonParse(getValue('userInfo'));
      auth.setUserInfo(jsonParse(getValue('userInfo')));

      /** 获取存储的用户 tab 数据 */
      const id = userInfo.id;
      const restoreTabs = (jsonParse(getValue('restoreTabs')) || {})[id];
      const restoreCurrentTab = (jsonParse(getValue('restoreCurrentTab')) || {})[id] || {};

      if (restoreTabs && NEED_RESTORE_TAB_INFO) {
        view.setTabs(restoreTabs);
        view.switchTab(restoreCurrentTab.mapping ? restoreCurrentTab : restoreTabs[0]);
      }

      this.setSystemListAndLoadViewList(restoreTabs);
    }

```


### 写一个新功能页面需要更改几处内容

1. 路由匹配
   需要在前端路由中将路由配置写明。命名要和约定一致。

2. 页面内容
   在`pages`建立对应组件目录，结构如下：
   
   ```
   
    ├── Xxx.component.tsx
    ├── Xxx.decorator.tsx
    ├── Xxx.selector.tsx
    ├── index.tsx
    └── style.less
       
   ```

3. 数据管理

   根据需要的流数据，写好配置，在`entities`中建立对应目录，结构如下:
   
   ```

    ├── xxx.constants.tsx
    ├── xxx.entity.tsx
    └── index.tsx
   
   ```

4. 后台交互

   在server端写入对应的`apis`


以上顺序是不一定的，都覆盖即可。

举个🌰

在个人中心加入一个通知配置的功能。因为是在同个菜单中的功能，因此照抄之前的修改信息的写法。

```

     items: [
          {
            key: 'resetpsd',
            title: '修改密码'
          },
          {
            key: 'notifyconfig',
            title: '通知配置'
          }
        ]

```

找到对应`modal`触发的地方。是`auth`里的数据流控制的，因此在对应的配置里加入。

```

 case 'resetpsd':
          auth.showResetPsdModal();
          break;
        case 'notifyconfig':
          auth.showNotifyConfigModal();
          break;

```

之后找到对应`resetPsd`组件,结构如下:

```

├── Notification.component.tsx
├── Notification.decorator.tsx
├── Notification.selector.tsx
├── index.tsx
└── style.less

```

建立对应的`notifyConfig`组件,修改逻辑与对接接口，并将其引入到`App/index.tsx`



## Node代码

node中间层的结构比较熟悉。
还是koa2 + typescript的组合。
目录如下:

```

├── index.js
├── index.ts
├── node_modules
├── portal
├── public
├── routes
├── views
├── webpack.dev.server.js
├── webpack.dev.server.ts
├── webpack.mobile.server.js
└── webpack.mobile.server.ts

```

在ts还无法直接运行的情况下，基本还是通过webpack进行转码成js格式，然后再运行。

`index.ts`中还是koa的常见写法，通过外部定义各种中间件，然后逐个引用，最后监听。

这里做了比较多的操作的是路由这部分。

`routes`也分为 `apis` 和 `pages`. 

在`apis`中，主要是根据模块区分了不同的配置列表，再通过根目录下的`common`中的`api_generator`来完成配置项到真实API映射。

`api_generator` 主要做了这么几件事情:

1. 获取了传递来的apis列表，并遍历。
2. 将其封装成函数到路由序列中。其中也约定了接口调用的顺序中需要执行哪些判断。

`pages` 主要逻辑在`controller.ts`, 它将几个常见入口（PC端，mobile端）做了逻辑处理，并进行对应的重定向跳转。




## 结尾

其实在熟悉核心流程以及如何配置后就可以着手开发了，不过需要先试验接通一个小功能从前端到node层的功能，然后再一步步扩展即可。


