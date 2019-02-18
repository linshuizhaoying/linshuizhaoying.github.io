---
layout: post
title: 如何从一个成熟项目中抽象开发环境（下）
category: 技术
tags: [总结,开发,前端,体系,工程化]
keywords: 总结,开发,前端,体系,工程化]
description: 
---

## 前言

继续上篇文章的内容，这篇主要是前端相关的代码。

在进行抽离的时候发现，需要删除的内容非常多，然后需要学习的东西也不少。
因此先分成几个模块，并一步步建立核心的开发流程。

```

1、前端路由
2、身份鉴权
3、数据管理
4、动态权限管理

```

## 前端路由

常规路由基本就是写一个组件，然后去总的目录下增加一个。
这里是通过配制式开发，将所有路由维护在一个目录下，暴露出指定格式的内容，比如:

```

import Welcome from '../pages/Welcome';

export default [
  {component: Welcome, path: 'home'}
];

```

然后在路由设置文件`routesSet.tsx`中，导入该类文件

```
export default [
  'login',
  'welcome'
];
```

入口文件`index.tsx`如下写:

```

import routesSet from './routesSet';

let childRoutes = [];
routesSet.forEach(element => {
  const route = require(`./${element}.routes`).default; // 加载同目录下的对应路由配置
  childRoutes = childRoutes.concat(route); // 将其拼接成一个数组
});

export const routes = childRoutes; // 赋值给routes并暴露出去

const routeConfig = [{
  path: '/',
  component: App,
  exact: true, // 如果为true，则仅在路径与location.pathname完全匹配时才匹配。
  childRoutes: [
    ...childRoutes,
    {name: 'Page not found', component: PageNotFound, path: '*'}
  ].filter(r => r['component'] || (r['childRoutes'] && r['childRoutes'].length > 0))
}]; // 过滤掉不匹配格式的项

// 生产配置json，其中每个路由配置还只是string，需要处理
const handleIndexRoute = route => {
   // 处理子路由
  if (!route.childRoutes || !route.childRoutes.length) {
    return;
  }
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

## 身份鉴权

这部分主要是node端做完鉴权的操作后，前端做个存储的动作，并在发送请求中加入鉴权的token。

主要是渲染的时候node端新加一个节点，然后前端从这个节点取数据后删除这个节点。

```

/** save csrfToken and remove the element */
const csrfToken = document.querySelector('#csrfToken');
csrfToken && (
  window['csrfToken'] = csrfToken['value'],
  csrfToken.remove()
);


```

封装自己的http请求，在最终请求中都带上token 

```

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

```

## 数据管理

数据管理的主要内容和之前node端的`api_generate`很相似。通过配置生成思想，利用rxjs的一些特性来实现。

```


// 定义动作类型。主要是异步动作，比如http请求
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
  // 共同的 loading 初始化为false, 并且每个动作都有其debug输出的动作。
  const actionLoading$ = new BehaviorSubject(false).debug(`[${entityName}]:[actionLoading]`);
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
      // 如果是custom,说明是自定义流
      case actionTypes.CUSTOM:
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
          result[`merge${toCamel(name)}Data`] = (key, data, isArray) => {
            result[`${name}Data$`].next(
              Object.assign(isArray ? [] : {}, result[`${name}Data$`].value, { [key]: data })
            );
          };
        }

        /** create filter Observable */
        if (filter) {
          result[`${name}FilterData$`] = new BehaviorSubject(filter.init).debug(`[${entityName}]:[${name}FilterData]`);
          result[`set${toCamel(name)}FilterData`] = data => {
            result[`${name}FilterData$`].next(data);
          };
        }

        if (pagination) {
          // 默认第一项
          result[`${name}CurPage$`] = new BehaviorSubject(1).debug(`[${entityName}]:[${name}CurPage]`);
          result[`set${toCamel(name)}CurPage`] = data => {
            result[`${name}CurPage$`].next(data);
          };

          result[`${name}PageSize$`] =
            new BehaviorSubject(pagination.pageSize || 20).debug(`[${entityName}]:[${name}PageSize]`);
          result[`set${toCamel(name)}PageSize`] = data => {
            result[`${name}PageSize$`].next(data);
          };

          result[`${name}Total$`] = new BehaviorSubject(0).debug(`[${entityName}]:[${name}Total]`);
        }

        /** create the main operation */
        result[name] = (params?) => {
          /** loading => true */
          actionLoading && result[`actionLoading$`].next(true);
          privateLoading && result[`${name}Loading$`].next(true);
          confirmLoading && common.showConfirmLoading();

          /** bind custom loading */
          // 更细颗粒度的绑定，与公共组件的loading区分开
          bindLoading && result[`${bindLoading}`].next(true);

          requestOpts.before && requestOpts.before(params, result);

          request({
            method: requestOpts.method,
            url: requestOpts.url(params, result),
            errNotification: requestOpts.errNotification,

            /** requestOpts.data is a function that return data according to params and the entity */
            // omit 删除对象中给定的 keys 对应的属性，后台不接受suc,err等字段
            data: requestOpts.data && requestOpts.data(omit(['sucCallback', 'errCallback'], params), result),

            sucCallback: response => {
              /** loading => false */
              actionLoading && result[`actionLoading$`].next(false);
              privateLoading && result[`${name}Loading$`].next(false);
              confirmLoading && common.hideConfirm();

              /** bind custom loading */
              bindLoading && result[`${bindLoading}`].next(false);

              /** set data according to response */
              data && result[`${name}Data$`].next(data.handler(params, response, result));

              /** set total according to response if pagination */
              pagination && result[`${name}Total$`].next(pagination.total(params, response, result));

              /** do callback if needed */
              requestOpts.sucCallback && requestOpts.sucCallback(params, response, result);
              params && params.sucCallback && params.sucCallback(params, response, result);
            },
            errCallback: err => {
              /** loading => false */
              actionLoading && result[`actionLoading$`].next(false);
              privateLoading && result[`${name}Loading$`].next(false);
              confirmLoading && common.hideConfirmLoading();

              /** bind custom loading */
              bindLoading && result[`${bindLoading}`].next(false);

              /** do callback if needed */
              requestOpts.errCallback && requestOpts.errCallback(params, err, result);
              params && params.errCallback && params.errCallback(params, err, result);
            }
          });
        };

        break;
    }
  });

  return result;
};

```

之后只要暴露类似配置，让其自动生成即可

```

  {
    type: actionTypes.ASYNC_ACTION,
    name: 'login',
    privateLoading: true,
    requestOpts: {
      method: 'POST',
      url: (params, entity) => `/apis/user/login`,
      data: (params, entity) => cipher(params),
      sucCallback: (params, data, entity) => {
        /** 登陆后设置用户信息 */
        setValue('userInfo', JSON.stringify(data));
        console.log(data);
      }
    }
  },

```

## 动态权限管理

这个其实主要思路是前后端约定权限列表的内容。包括视图和功能模块。
每次登陆后返回一份列表，前端存到本地。读取需要显示的内容。
这样就做到了动态的权限分配。