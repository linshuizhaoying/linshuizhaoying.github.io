---
layout: post
title: 如何从一个成熟项目中抽象开发环境（上）
category: 技术
tags: [总结,开发,前端,体系,工程化]
keywords: 总结,开发,前端,体系,工程化]
description: 
---

## 前言

本篇文章是延续之前接手项目的时候一个想法，那就是接手旧项目并完成开发后，还能继续做点什么。在内部的分享中，我写了一条就是抽离项目开发环境，将精华部分抽出，打造一个自己的开发环境。

目的是为了更好地巩固所学，以及能够替换以前的个人开发环境。

因此本文就是整个流程的记录。

## 目标

在做一件事情前，一定要明确自己目标，并且将其分成多个小部分，制定计划一步步完成。

这点是在我大脑有了一些想法，就直接去抽离，发现一些误区之后重新考虑的。

在之前我是直接对项目进行下手，没有考虑到一些细节的影响，导致，改东西都会报错，然后原以为自己挺熟悉项目了，却发现自己不是真正的熟悉。

因此目标管理是非常重要的。

项目分为前端部分和NODE后端部分。

而且NODE后端部分基本只做鉴权与接口转发。并不自己去操控数据库。而我是期望能够有一套能连接MySQL的小系统开发环境。因此如果从原项目进行一步步修改，明显是很费力的。而且还要将很多东西一点点从项目里移除，还要考虑是否会对其它部分产生影响。

因此这块需要自己先搭建一个基本环境，然后根据需求一步步迭代是比较好的选择。

Node端我希望能从项目里学习到的有:

```

1、webpack 配置，能够自动热更新，能够区分移动端与PC入口。
2、路由与页面的自动生成器
3、封装好的一些库
4、日志功能

```

因此我打算先构建一个 `Typescript + koa + mysql` 的项目，然后再一步步递进。

然后发现似乎不对，感觉没法更深刻体会原有的精华。

因此还是决定从头删起，然后先设定一下整个流程。

```
1. 配置两个webpack, 一个用于pc端，一个用于移动端。开发阶段分为三个，一个起node服务，一个热更新pc端，一个热更新移动端。
2. node端需要api生成器。需要csrf鉴权。需要数据库模块。需要图片上传模块。
   api生成器不能完全借鉴已有的，得做新的设计。
   总的来说是配置
   {
     url: '/product/:productId/feedback'
     method: get/post/put/delte
     name: '产品反馈',
     
   }
   

```


### 抽离环境

学习这套写法最好的方式还是自己动手重新做一个环境。

首先在之前开发以及了解了这套系统的几个核心模块。

它是一套支持三端并行开发的环境。

通过webpack与node处理，能够完美的兼容开发与部署模式。

因此这个开发环境需要有以下模块

```
1、后端
   1.1 api generate
   1.2 mysql数据操作模块
   1.3 热更新与路由配置，三端转发。
   1.4 请求校验
   1.5 日志记录
  

```

### Api Generate

简单的构想代码应该是如下的：


```

/*
  * 日期: 2018-08-14 09:07:28
  * 函数名: api_generator
  * 描述: 根据配置生成API路由，并执行对应数据操作方法。
  * 接口配置如下:
  *
  *    method: get | post | put | delete, // 调用方式
  *    name: 'xx列表', // 接口名称
  *    apiPath: '/product/:productId/auto_reply/:id', // 接口地址 等同于匹配路由
  *    apiSourceType: 'database' || 'externalUrl', // 接口源分为内部数据库与外部地址,默认为内部数据库
  *    apiSourcePath: func || 'Url' // 根据接口源类型去分别找对应的接口源地址拿数据，执行操作
  *    [可选] getHeader: function (ctx) { return ({
  *      authorization: md5(('/user/detail' + Date.now() + ldapConfig.authKey).toUpperCase())
  *    }); }
  * 生成器参数如下:
  *
  *    method: get | post | put | delete, // 调用方式, 默认 get
  *    name: xx, // 接口名称 === 日志名称
  *    apiPath：// 需要解析的接口地址
  *    requestAuthCheck：false || true 校验中间件, 默认true,校验accessToken. 如登录接口是不需要身份校验，因此false跳过
  *    apiSourceType: 'database' || 'externalUrl', // 接口源分为内部数据库与外部地址,默认为内部数据库
  *    apiSourcePath: func || 'Url' // 根据接口源类型去分别找对应的接口源地址拿数据，执行操作
  *    businessBefore: server 前的 (ctx, params, query, body, file) => {}
  *    business： server后的 (ctx, params, query, body, file) => {}
  *    header: 设置请求头
*/

import * as request from './request_helper';
import * as messageHelper from './message_helper';
import {
  accessTokenCheck
} from './middlewares/authCheck';

const defaultRequestAuthCheck = accessTokenCheck();

// 用于加载校验中间件, 可以自定义或者使用默认校验。
const wrapAuthCheck: (params: asyncFunction | boolean) => asyncFunction | void = params => {
  if (params instanceof Function) return params;
  if (params === false) return async (ctx, next) => await next();
};

/** 找出 path 的参数 */
const getParams = path => (path.match(/:\w*/g) || []).map(param => param.substring(1));

/** 使用 path 参数替换服务器路径中形如 :id 的字串 */
const getRealServerPath = (serverPath, serverParams, params) => {
  let result = serverPath;
  serverParams.forEach(param => {
    result = result.replace(`:${param}`, params[param]);
  });
  return result;
};

export default (apis: IApis) => router => {
  apis.forEach(api => {
    /** api 本身的调用方式 */
    const method = api.method;
    /** api 的显示名称 */
    const name = api.name;
    /** api 本身的 path */
    const apiPath = api.apiPath;
    /** 校验中间件 */
    const requestAuthCheck = wrapAuthCheck(api.requestAuthCheck) || defaultRequestAuthCheck;
    /** 后台的调用方式 */
    const apiSourceType = api.apiSourceType;
    /** 后台的调用数据源 */
    const apiSourcePath = api.apiSourcePath;
    /** 设置请求头 */
    const getHeader = api.getHeader || (async ctx => {});

    /** 调用后台前执行的业务 */
    const businessBefore = api.businessBefore;
    /** 调用后台接口后，或者不调用接口时执行的业务 */
    const business = api.business;

    const Database  = (apiSourcePath, serverParams) => {
      return {
        data: {
          api: apiSourcePath,
          params: serverParams
        }
      }
    }

    router[method](apiPath, requestAuthCheck, async ctx => {
      const params = ctx.params || {};
      const query = ctx.request.query || null;
      const body = ctx.request.fields || null;
      const file = ctx.request.files || null;
      const header = await getHeader(ctx);
      /** 执行调用后台接口前的逻辑 */
      if (businessBefore) {
        await businessBefore(ctx, params, query, body, file);
      }

      /** 获取真实URL里的参数 */
      const serverParams = getParams(apiPath);
      /** 根据 params，将参数填充，得到真正的 path */
      const realServerPath = getRealServerPath(apiPath, serverParams, params);
      const startTime = new Date().getTime();
      /** 如果接口源是内部数据库，那么请求要发到对应 controller */
      if (apiSourceType === 'database') {
        ctx.logger.log(`/******************** ${name} connect to database ***********************/`);
        ctx.logger.log(JSON.stringify({
          path: realServerPath,
          query: query,
          body: body,
          header
        }));
        ctx.logger.log(`/*******************************************************************/`);
        try {
          const result: any = await Database(apiSourcePath, serverParams);
          ctx.logger.log(`/****************** ${name} response from server ********************/`);
          ctx.logger.log(JSON.stringify(result));
          ctx.logger.log(`/********************************************************************/`);
          ctx.logger.log(`time spent: ${new Date().getTime() - startTime} ms`);

          // /** 执行调用后端接口后的逻辑 */
          if (business) {
          /* 将apiResult挂载到ctx上，让busniess函数去处理数据，做聚合、清洗等判断操作  */
            ctx.apiResult =  result;
            await business(ctx, params, query, body, file);
          } else {
            return messageHelper.success(ctx, result);
          }
        } catch (e) {
          ctx.logger.log(`/****************** ${name} error from Database ***********************/`);
          ctx.logger.log(JSON.stringify(e));
          ctx.logger.log(`/********************************************************************/`);
          ctx.logger.log(`time spent: ${new Date().getTime() - startTime} ms`);

          if (business) {
            ctx.apiResult = e;
            await business(ctx, params, query, body, file);
          } else {
            return messageHelper.response(ctx, e);
          }
        }
      }
      /** 如果接口源是外部URL，那么请求要发到对应外部接口 */
      if (apiSourceType === 'externalUrl') {
        try {
          const result: any = await request.send({
            url: apiSourcePath,
            method,
            path: realServerPath,
            query,
            data: body,
            ctx,
            header
          });
          ctx.logger.log(`/****************** ${name} response from server ********************/`);
          ctx.logger.log(JSON.stringify(result));
          ctx.logger.log(`/********************************************************************/`);
          ctx.logger.log(`time spent: ${new Date().getTime() - startTime} ms`);

          // /** 执行调用后端接口后的逻辑 */
          if (business) {
          /* 将apiResult挂载到ctx上，让busniess函数去处理数据，做聚合、清洗等判断操作  */
            ctx.apiResult = result;
            await business(ctx, params, query, body, file);
          } else {
            return messageHelper.success(ctx, result);
          }
        } catch (e) {
          ctx.logger.log(`/****************** ${name} error from server ***********************/`);
          ctx.logger.log(JSON.stringify(e));
          ctx.logger.log(`/********************************************************************/`);
          ctx.logger.log(`time spent: ${new Date().getTime() - startTime} ms`);

          if (business) {
            ctx.apiResult = e;
            await business(ctx, params, query, body, file);
          } else {
            return messageHelper.response(ctx, e);
          }
        }
      }
    });
  });
};

type asyncFunction = (ctx: any, next: any) => Promise<void>;
export interface IApiConfig {
  method: 'get' | 'post' | 'delete' |'put' | 'patch';
  name: string;
  apiPath: string;
  requestAuthCheck?: any;
  apiSourceType?: 'database' | 'externalUrl';
  apiSourcePath?: string;
  businessBefore?: any;
  business?: any;
  getHeader?: (ctx) => object;
}
export type IApis = IApiConfig[];


```

这个第一版本可以优化的点还是蛮多的。

不过最重要的是让它先跑起来。

正式跑起来之前还需要将之前的`requestAuthCheck` 方法给注释掉，因为并不存在校验中间件。

之后将looger输出优化一下，代码如下:

```
export const loggerInfo = (ctx, requestData, name, apiSourceType, result, startTime) => {
  ctx.logger.log(`/****************** ${name} 目标响应来自:  ${apiSourceType === 'database' ? '数据库' : '外部服务'} ********************/`);
  ctx.logger.log(`/****************** 正在请求 ********************/`);
  ctx.logger.info(`/****************** ${JSON.stringify(requestData)}  ********************/`);
  ctx.logger.log(`/****************** 返回结果如下： ******************************************/`);
  ctx.logger.info(`/****************** ${JSON.stringify(result)} ********************/`);
  ctx.logger.log(`/********************************************************************/`);
  ctx.logger.log(`花费时间: ${new Date().getTime() - startTime} ms`);
};

export const loggerError = (ctx, requestData, name, apiSourceType, result, startTime) => {
  ctx.logger.log(`/****************** ${name} 目标响应来自:  ${apiSourceType === 'database' ? '数据库' : '外部服务'} ********************/`);
  ctx.logger.warn(`/****************** 请求出错！ ********************/`);
  ctx.logger.log(``);
  ctx.logger.log(`/****************** 错误信息如下： ******************************************/`);
  ctx.logger.error(`/****************** ${JSON.stringify(result)} ********************/`);
  ctx.logger.log(`/********************************************************************/`);
  ctx.logger.log(`花费时间: ${new Date().getTime() - startTime} ms`);
};
```

然后将原来Logger的地方替换掉。

### mysql数据操作模块

接着应该是mysql数据模块。这里为了测试先定一个一个 `news` 数据库。
然后常见的 `User` 表
字段设定为:

```

username
password
email
role
regDate


```

之后是要学习[Sequelize](https://demopark.github.io/sequelize-docs-Zh-CN/getting-started.html)。

>> Sequelize 是一个基于 promise 的 Node.js ORM, 目前支持 Postgres, MySQL, SQLite 和 Microsoft SQL Server. 它具有强大的事务支持, 关联关系, 读取和复制等功能.

需要先学习一下原生的写法。

了解个大概开始学习 [sequelize-typescript](https://github.com/RobinBuschmann/sequelize-typescript)，它封装了一堆装饰器能进行快速开发。

然后是需要参考以前的数据库结构，再学习一下网上的代码的数据结构。结合现有环境进行设计。


先将该装的依赖都装上:

```

npm install --save sequelize
npm install --save mysql2
npm install --save pg pg-hstore

```

利用简单的测试语句测试远程数据库是否连接成功

```

const sequelize = new Sequelize('mysql://user:pass@host:3306/dbname');
sequelize
  .authenticate()
  .then(() => {
    console.log('successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });

```

然后写四个简单的接口，包含增删改查。

这里测试的时候踩了很多坑，而且中文文档基本没有记录，这里记录一下方便后来人。

首先 tsconfig.json需要写明
```
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
   
    "target": "es6",
    ...
```

target 不能用低于es6的版本，原因是引用了Sequelize 4，它的Model方法使用 es6 classes。所以你想用就必须不能用低版本的转换。

如果你没有将`emitDecoratorMetadata` 设置为`true`
那么你将得到各种错误信息。。。

还有就是被官方的文档坑了一小把。
主要原因是解析路径的问题。

官方是`modelPaths: [__dirname + '/models']`在`sequelize`初始化的时候要先去加载Models，但是这里有个问题，设定的Model在配置文件的同父级的另一个目录，因此加上了`../`。这里就会各种报错。无法解析`Model`, 错误信息有`Model not initialized`。

这里解决方案就是

```

import { resolve } from 'path';

modelPaths: [resolve(__dirname, '../_server_src/db/models/')]

```

这样写就行了。

这里演示一个简单的例子，首先去`routes\apis\user` 下创建`index.ts`

里面的接口内容如下：

```

  method: 'get',
  name: '所有用户列表',
  apiPath: '/user',
  apiSourceType: 'database',
  apiSourcePath: 'UserList'
  
```

然后去 `db` 目录下建立如下目录:


```

├── controller
|  ├── index.js
|  ├── index.ts
|  ├── user.js
|  └── user.ts
├── models
|  ├── user.js
|  └── user.ts
└── services
   ├── index.js
   ├── index.ts
   ├── user.js
   └── user.ts

```

可以看到将整个结构分成了三部分。

`models` 作为基础的模型

```

import {AutoIncrement, Column, CreatedAt, Model, PrimaryKey, Table} from 'sequelize-typescript';

@Table
export default class User extends Model<User> {
  @Column({
    comment: '自增ID',
    autoIncrement: true,
    primaryKey: true
  })
  id: number;

  @Column({
    comment: '姓名'
  })
  username: string;

  @Column({
    comment: '密码'
  })
  password: string;

  @Column({
    comment: '邮箱'
  })
  email: string;

  @Column({
    comment: '注册时间'
  })
  regDate: string;
}


```

`controller` 作为操作数据层。

```

const UserList = () => {
  return User.findAll();
};

```


`service` 作为服务层，提供最终返回数据的内容。它可以处理传递前的参数与最终返回数据的格式。


```

const UserListService = (params: any) => {
  console.log('params', params);
  return controller['UserList']();
};

```

结构分层后各司其职。


###  热更新

开发阶段最重要的就是开发效率，热更新就是一个很重要的提升因素。

常见的热更新都是基于`webpack`，因此本例也不例外。只不过在node层热更新的形式上加了不少东西。

首先肯定是来看基本的配置

```
  /** 热更新插件 */
  plugins.push(new webpack.HotModuleReplacementPlugin());

  /** 热更新插件配套插件 此插件将导致在启用 HMR 时显示模块的相对路径。建议用于开发。 */
  plugins.push(new webpack.NamedModulesPlugin());

```

特别注意的是要`const DEBUG_MODE = process.env.NODE_ENV !== 'production';
`
定义开发模式和生产模式的区别。

因为热更新仅限制于开发环境。

在开发模式中还需要定义环境变量。

```

  /** 定义环境变量 */
  plugins.push(
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: JSON.stringify('development')
      }
    })
  );
  
```

>>DefinePlugin 允许创建一个在编译时可以配置的全局常量。这可能会对开发模式和发布模式的构建允许不同的行为非常有用。如果在开发构建中，而不在发布构建中执行日志记录，则可以使用全局常量来决定是否记录日志。这就是 DefinePlugin 的用处，设置它，就可以忘记开发和发布构建的规则。

因此整个热更新的逻辑是这样的

```


if (DEBUG_MODE) {
  /** 热更新插件 */
  plugins.push(new webpack.HotModuleReplacementPlugin());

  /** 热更新插件配套插件 */
  plugins.push(new webpack.NamedModulesPlugin());

  /** 定义环境变量 */
  plugins.push(
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: JSON.stringify('development')
      }
    })
  );
} else {
  /** 编译前清空文件夹 */
  plugins.push(new CleanWebpackPlugin(['public/js'], {
    root: path.join(__dirname, '../_server_src'),
    verbose: true,
    dry: false
  }));

  /** 压缩 js */
  plugins.push(
    new ParallelUglifyPlugin({
      uglifyJS: {
        output: {
          comments: false
        },
        compress: {
          warnings: false,
          drop_console: true,
          pure_funcs: ['console.log']
        }
      }
    })
  );

  /** 定义环境变量 */
  plugins.push(
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: JSON.stringify('production')
      }
    })
  );
  plugins.push(new MD5HashPlugin());
  /*
    Use the optimization.noEmitOnErrors to skip the emitting phase whenever there are errors while compiling.
    This ensures that no erroring assets are emitted.
    The emitted flag in the stats is false for all assets.
  */
  plugins.push(new webpack.NoEmitOnErrorsPlugin());
  /*
    一个更具侵略性的块合并策略的插件,即使是相似的块也会被合并，如果总尺寸减小到足够。产生更少的 bundle
  */
  plugins.push(new webpack.optimize.AggressiveMergingPlugin());
  /*
    提升(hoist)或者预编译所有模块到一个闭包中，提升你的代码在浏览器中的执行速度。
  */
  plugins.push(new webpack.optimize.ModuleConcatenationPlugin());
}


```

我们还有需要node端去渲染页面，自动注入js。这种形式可以帮助区分开发和部署模式，而且可以根据不同路由去渲染不同的页面，因此可以完成移动端和pc端页面的入口检测。

具体思路就是 `package.json` 里写的开发模式启动命令是起一个`koa`服务器利用`koa-webpack`去读取之前写的配置。然后在`koa`路由里，区分`apis`与`pages`路由。在`pages`路由的`controller`里去注入`webpack`开发配置生成的js文件。

```

const injectScript = (options?: string[]) => {
  const files = options || ['js/vendor.js', 'js/app.js'];
  const result = [];
  if (env === 'development') return files;
  fs.readdirSync(path.join(constants.ROOT_PATH, '/_server_src/public/js')).forEach(item => {
    result.push(`js/${item}`);
  });
  result.sort((a, b) => {
    if (a > b) return -1;
    if (a < b) return 1;
    return 0;
  });
  return result;
};


```

入口文件`entry`这么写:

```

export const entry = async (ctx, next) => {
  try {
    if ( isMobile(ctx.request.header['user-agent'])) {
      return ctx.redirect(env === 'development' ? jump.mobileUrl + `mobile/feedbackDetail` : '/' + `mobile/feedbackDetail` );
    } else {
      return ctx.redirect(env === 'development' ? jump.pcUrl  + `pages/home` : '/' + `pages/home`);
    }
  }catch {
    ctx.body = pug.renderFile('_server_src/views/error.pug');
    return ctx;
  }
};

```

pc端入口`index`路由里获取注入的文件名称。然后渲染。

```

export const index = async (ctx, next) => {
  console.log('index')
  const scripts = injectScript();
  const options = {
    env,
    scripts
  };

  ctx.body = pug.renderFile('_server_src/views/index.pug', options);
  return ctx;
};

```
    
移动端入口同上加一个`injectMobileScript`即可。

这样`node`端不管你前端页面用什么开发。最终打包输出的目录确定了，只有加一个读取路径的函数，就能做入口检测，自动渲染。

### 请求校验
请求校验在公司项目里会用到非常多，但是在个人项目中，实际上没有那么多权限需要校验，而且没法做到那么复杂的后端校验逻辑。
因此校验只提取简单的accessToken。

通过对每个用户的请求之前加载中间件，然后鉴权即可。
`accessToken` 中间件也比较简单，只需要获取前端设置的cookies。这个cookie就是我们登录后存进去的token，可以利用`uuid`模块生产唯一标识符。也可以开一个`redis`服务来储存对应的`token`。

简单的写法如下:

```

/** 默认的请求校验 */
export const accessTokenCheck: requestAuthCheckType = (options?) => {
   return async (ctx, next) => {
    const accessToken: string = ctx.cookies.get(params.tokenName);
    const needToken: boolean = !(new RegExp(params.noForbidden.join('|')).test(ctx.originalUrl));
    const isExist: boolean = await tokenExist(accessToken);

    if (needToken && !isExist) {
      return messageHelper.response(ctx, {
        errCode: 401,
        errMsg: '登录超时'
      });
    }
    if (isExist) {
      const value: any = await redisHelper.get(accessToken);
      redisHelper.set(accessToken, value, params.extendTime);
      if (value && value !== 'undefined') {
        ctx.uid = JSON.parse(value).id;
        console.log(ctx.uid);
      }
    }
    await next();
  };
};
```


### 日志记录
日记记录一般是基于现有的工具封装，比如`koa-logger`。
需要做的处理一个是全局的错误捕捉。
一个是静态资源`css`等的跳过。
然后是写到配置的文件目录。
最后是暴露一个函数到node全局。这样直接调用`ctx.logger`即可使用。

简单的一个例子

```

// 记录基础的请求时间,跳过静态资源
  // 参考koa-logger
  const start = new Date().getTime();
  const logsMemory = []; // logs缓存，打log不会真的输出，而是记录

  ctx.logger = libLoggerInstance.generate(logsMemory);

  if (!isStatic(ctx.url) || (isStatic(ctx.url) && !skipStatic)) {
    ctx.logger.log(`Started ${ctx.method}   ${ctx.url} for ${ctx.ip} at ${new Date()}`);
    ctx.query && ctx.logger.log(`  query: ${JSON.stringify(ctx.query)}`);
    ctx.request.fields && ctx.logger.log(`  body: ${JSON.stringify(ctx.request.fields)}`);
  }

  try {
    await next();
  } catch (err) {
    ctx.logger.error(error2string(err));
    ctx.logger.flush();

    // 告诉全局的error监控，此错误已经处理过了
    err.hasHandled = true;
    // 抛出去 方便其他程序监控
    ctx.throw(err);
  }


```


## 前端代码

前端代码分为两部分，一个是pc端的代码，一个是移动端的代码。

这两个基本上都可以自由搭配使用流行的框架。

只要保持入口文件`index.tsx`和`webpack`打包一致就可以了。

这篇文章具体就不展开这部分了。

## 总结

最后发现其实要自己写的内容主要是数据库这块，思想都是先通读源码，走流程，在项目结束后再去回顾总结。

总的来说，在公司接手旧项目能看到这么优秀的代码还是很庆幸的，学到了很多东西~

