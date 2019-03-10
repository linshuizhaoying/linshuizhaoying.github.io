---
layout: post
title: 百炼成钢-HTTP请求方式总结
category: 技术
tags: [总结,开发,前端,Http]
keywords: 总结,开发,前端,Http]
description: 
---

# 目标

工作大半年，接触了不少项目，因此给自己定了一个总结和学习的计划，总结现阶段分三个部分。

```
1、http
2、Store
3、Middleware

```

分成这三模块主要是我觉得在开发时候有必要去理顺的部分。

首先是总结项目中遇到的所有数据请求的方（套）法（路）。

## 从axios开始

`Axios 是一个基于promise 的HTTP 库`

接触 `axios`最早的时候还是推送平台时，被封装的函数:

```JavaScript

function callApi(endpoint, schema, data, method, suc, err) {
  axios({
    method,
    url: `/${endpoint}`,
    data: omit(data, isFunction),
    params: method.toLowerCase() === "get" ? omit(data, isFunction) : null
  })
    .then(result => {
      if (schema === schemas.NORMAL_RESP) {
        return suc(Object.assign({}, result.data));
      } else {
        return suc(Object.assign({}, normalize(result.data || {}, schema)));
      }
    })
    .catch(error => {
      err(error.response.data);
    });
}

```

简单的将各种请求封装起来，提高复用。

## 被封装的http

因为`axios` 能同时跑在浏览器与Node端，因此利用它封装一个前端通用的函数就很自然。

```JavaScript

import axios from 'axios';

export default ({
  url,
  method,
  data,
  params,
  paramsSerializer,
  wholeResp = false,
  sucCallback,
  errCallback,
  showErrMsg = true,
  showSucMsg = false,
  defaultValue,
  debug = false,
  responseType = 'json'
}: IParams) =>
  new Promise<{ success: boolean; data: any }>((resolve, reject) => {
    const csrfToken = ((window as any).SERVER_DATA || {}).csrfToken;
    axios({
      url,
      method,
      params,
      paramsSerializer,
      data,
      responseType,
      headers: {
        'csrf-token': csrfToken,
        'x-forword-referer': 'web'
      }
    })
      .then((resp: any) => {
        if (!resp) {
          showErrMsg && showWarn('请求地址不存在 ⊙﹏⊙ !');
          return;
        }
        if (
          resp.status &&
          resp.status === 200 &&
          resp.data &&
          (resp.data.status && +resp.data.status !== 0)
        ) {
          debug && console.info(`%c ${method} ${url} failed!`, failure);
          showErrMsg && showWarn(resp.data.message);
          errCallback && errCallback(resp.data);
          resolve({
            success: false,
            data: defaultValue
          });
        } else {
          debug && console.info(`%c ${method} ${url} successful!`, success);
          showSucMsg && showSuc();
          sucCallback &&
            sucCallback(
              wholeResp ? resp.data : resp.data ? resp.data.data : null
            );
          resolve({
            success: true,
            data: wholeResp ? resp.data : resp.data ? resp.data.data : null
          });
        }
        debug && console.info(`${method} ${url} response`, resp.data);
      })
      .catch(err => {
        debug && console.info(`%c ${method} ${url} failed!`, failure);
        debug && console.info(`${method} ${url} response`, err);
        if (err) {
          if (err.status === 504 || err.status === 404 || err.status === 405) {
            showErrMsg && showWarn('请求地址不存在 ⊙﹏⊙ !');
          } else if (err.status === 403) {
            showErrMsg && showWarn('权限不足,请联系管理员！');
          } else if (err.status === 401) {
            showErrMsg && showWarn('登陆过期啦！');
          } else {
            showErrMsg && showWarn('服务器出现未知错误!');
          }
        } else {
          showErrMsg && showWarn('服务器出现未知错误!');
        }
        errCallback && errCallback(err);
        resolve({
          success: false,
          data: defaultValue
        });
      });
  });
```

因为是封装给前端使用的，因此考虑的错误提示会相对比较多，这样排查错误的时候也能很快定位。

除了常见的参数传递，还可以看见一个特殊的 `paramsSerializer`,这是一个负责 `params` 序列化的函数。正常项目中它很少出现，但是在csb项目中它出现很频繁。而且通常这个参数的形态是这样的:

```

import qs from 'query-string';

function paramsSerializer(params) {
  return qs.stringify(params, { arrayFormat: 'brackets' });
}

```

`query-string` 在npm上的效果是这样的

```

queryString.stringify({foo: [1,2,3]}, {arrayFormat: 'bracket'});
// => foo[]=1&foo[]=2&foo[]=3

queryString.stringify({foo: [1,2,3]}, {arrayFormat: 'index'});
// => foo[0]=1&foo[1]=2&foo[3]=3

```

来个实际例子更加形象:

我们删除多个数据，会传递一个包含id的数组给接口

```
ids: ["436a70f535f14904ba748b4fe3bd2ef2", "d4aac569ed2b47f4987eb918a6defc77"]

```

假设接口长这样:

```

axios.delete(`/xxx`
      params: { ids },
      paramsSerializer
    });

```

通过转换后数据变成

```
xxx?ids=436a70f535f14904ba748b4fe3bd2ef2&ids=d4aac569ed2b47f4987eb918a6defc77

```

而且这个封装函数还让用户能够自定义成功后或失败后需要额外执行的函数。这大大增加了可扩展性。

**这段封装有个地方，就是`promise` 的返回值总是被`resolve` 而不是`reject`,这是为什么 ？**

答案很简单，是为了统一返回的格式，这样处理起来只需要判断状态是否成功即可。


## superagent

`Superagent 是一个基于 Promise 的轻量级渐进式 AJAX API，非常适合发送 HTTP 请求以及接收服务器响应`

在项目中:

```JavaScript

import * as request from 'superagent';

export async function getAllTodo(ctx, data: any) {
  return new Promise((resolve, reject) => {
    request
      .get(`xxxx/page`)
      .end((err, result) => {
        if (err) {
          ctx.logger.error(`getAllTodo error: ${JSON.stringify(err)}`);
          resolve(false);
        } else {
          ctx.logger.log(
            `getAllTodo result: ${JSON.stringify(result)}, ${result.status}`
          );
          // @ts-ignore
          resolve(JSON.parse(result.text).data);
        }
      });
  });
}

const res: any = await getAllTodo(ctx, {
  accessToken
});

```
这种写法基本就是利用`superagent`自带的写法，外面再封装一层promise把里面的内容返回出去。
适用于node去请求特殊的api接口地址，而且还需要带上鉴权的参数。

### 二进制转base64

这里需要提及一个业务场景，也就是制作用户信息卡片的时候，需要显示用户的头像。
这种根据接口的不同可以有不同的实现。
`node端去请求接口，拿到二进制数据后转为base64然后前端展示的时候处理一下`

```JavaScript

import * as request from 'superagent';

/** 图片二进制转base64 */
const binaryParser = (res, callback) => {
  res.setEncoding('binary');
  res.data = '';
  res.on('data', chunk => (res.data += chunk));
  res.on('end', () =>
    callback(null, Buffer.from(res.data, 'binary').toString('base64'))
  );
};

/** 获取用户头像 */
export async function getUserAvatar(ctx, data: any) {
  return new Promise((resolve, reject) => {
    request
      .get(
        `xxx/api/v1/avatar?username=${
          data.username
        }`
      )
      .buffer()
      .parse(binaryParser)
      .end((err, result) => {
        if (err) {
          resolve(false);
        } else {
          ctx.logger.log(
            `getUserAvatar result: ${JSON.stringify(result)} ${JSON.stringify(
              result.body
            )}`
          );
          resolve(result.body);
        }
      });
  });
}

```

前端引用的时候用`antd`的`Avatar组件`

```

 <Avatar src={`data:image/png;base64,${avatar}`} />

```

这种实现挺别致的，也很有意思。因为一般来讲实现思路是拿到头像的地址，拼接起来，然后让`img`的url指向它。
这边是node把数据给直接转了并提供。

### request_helper

基本是只要是为了能提高复用的基本是都会被抽成函数，因此，即时是 `Superagent` 也被抽成 `request_helper`


```JavaScript

import * as request from 'superagent';

export async function sendRequest(configs: IConfigs) {
  const { url, method, path, query, data, ctx, isForm, header } = configs;

  return new Promise((resolve, reject) => {
    const finalHeader = header;

    const q = request[method](`${url}${path}`).query(
      Object.assign({}, query || {})
    );

    if (finalHeader) {
      for (const key in finalHeader) {
        if (key && finalHeader[key]) {
          q.set(key, finalHeader[key]);
        }
      }
    }

    if (data) {
      q.send(data);
    }

    if (isForm) {
      q.type('form');
    }

    q.end((err, result) => {
      if (err) {
        console.error(JSON.stringify(err));
        if (err.code === 'ECONNREFUSED') {
          reject({
            status: 50000,
            message: '服务器连接失败，请重试'
          });
        }
        if (err.response) {
          reject({
            status: err.response.body.status || 50000,
            message:
              err.response.body.message || '服务器出错啦，请重试' + err.status
          });
        } else {
          reject({
            status: 50000,
            message: '服务器出错啦，请重试' + JSON.stringify(err)
          });
        }
      } else {
        if (result.body) {
          resolve(result.body);
        } else {
          resolve(result.text);
        }
      }
    });
  });
}

export const send = async (configs: IConfigs) => {
  const result = await sendRequest(configs);
  return result;
};

interface IConfigs {
  url?: string;
  method: 'get' | 'post' | 'del' | 'put' | 'patch';
  path: string;
  query?: object;
  data?: object;
  ctx: any;
  isForm?: boolean;
  header?: object;
}

```

这里的封装是为了符合业务，比如 `type`, 提交的数据有表单，因此需要符合和后台约定的格式。其余的也是在请求层做了一层
错误兼容。

## 进击的封装

基础的封装只适用于简单的请求，因为业务的不同也会进行进一步的封装，或者侧重点不同的封装。

仅仅是请求接口可以有很多形态。

### fetchList

fetchList 是结合了mobx一些特性的封装。

作用是获取常见的列表，需要考虑的是分页情况。

```JavaScript

/**
const tableFields = {
  des: { descend: "DESC", ascend: "ASC" },
  orderField: "orderBy",
  orderType: "order",
  page: "page",
  pageSize: "pageSize",
  total: "total"
}
*/

export const fetchList = ({
  list,
  url,
  pagination = { [tableFields.page]: 1, [tableFields.pageSize]: 1 },
  filter = {},
  sort = {},
  query = {},
  noPagination = false
}) => {
  noPagination && (pagination = {});
  const params = { ...pagination, ...filter, ...sort, ...query };
  list.loading = true;
  return http({
    method: 'get',
    url,
    params,
    sucCallback: data => {
      list.loading = false;
      list.data = data;
    },
    errCallback: () => {
      list.loading = false;
    }
  });
};

```

这里会看到分页的各种参数，包括排序都需要考虑进去。
这里的`http`就是`axios`封装的基础版本。

而且可以看到这里其实有个耦合项就是loading。这是为了方便前端请求的时候连loading状态都可以直接省略。

要使用`fetchList`,就要引入Mobx。并约定`list`参数是什么样的。这和后台也是一并约定遵守的格式。

```JavaScript

  @observable
  list = {
    loading: false,
    filter: {},
    sort: {},
    data: {
      list: [],
      pagination: {
        pageNum: 1,
        pageSize: 10,
        total: 0
      }
    }
  };
  
  /** 加载数据 */
  @asyncAction
  *loadList(
    pagination = {
      pageNum: this.list.data.pagination.pageNum,
      pageSize: this.list.data.pagination.pageSize
    },
    filter = this.list.filter,
    sort = this.list.sort
  ) {
    yield fetchList({
      list: this.list,
      url: '/xxx',
      pagination,
      filter,
      sort
    });
  }
```

## 不得不提的Generator

  算了太多了，之后提。
  
# 小结

从最后来看，现有的封装手段已经可以支持大部分业务的，特殊处理的是分页的封装，需要和后端接口统一参数。而且要根据开发框架和状态管理工具的不同而利用不同的特性来实现对开发优化的封装。封装这类公共的组件，尤其需要考虑可扩展性，即时有些地方已经帮用户解决了，比如错误的判断，但是还是需要支持用户自定义的错误处理函数。

# 讨论

对于 这类请求 我们还能封装到什么程度？ 全能类？针对业务类？

在项目中，你们是怎么封装的？

感兴趣可以邮箱联系~

