---
layout: post
title: 从零开始打造 Mock 平台 - 核心篇
category: 技术
tags: [学习,开发,前端,Node,React,React,Mock,解析器,构造]
keywords: 学习,开发,前端,RxJS,React,React,Mock,解析器,构造
description: 
---

# 前言

最近一直在捣鼓毕设，准备做的是一个基于前后端开发的Mock平台，前期花了很多时间完成了功能模块的交互。现在进度推到如何设计核心功能，也就是Mock数据的解析。

根据之前的需求设定加上一些思考，用户可以像写json一般轻松完成数据的mock，也可以通过在mock数据模型之上进行构建出复杂的数据模型并在项目中引用。

这看似简单的需求其实需要处理几个不同的模块功能以及交互设计。该如何处理解析不同mock数据并进行构造？前端交互中模拟数据该如何处理？数据构造时如何加载用户设定的数据模型？错误捕捉与处理？

这些都暂时没有一个好的处理结果。因此想要完成核心功能我们需要明确需求，并且通过同类产品是如何处理的，通过阅读它们的源码来学习思想并加入。

## 明确需求

在明确该功能模块之前我们可以通过模拟流程来明确。


> 用户 -> 添加数据模型 - > 实时看到构造结构

> 用户 -> 添加接口 -> 构造json格式返回参数  -> 预览

构造json格式返回参数 不仅包含返回的正文，同时也设定了 header 和 method。





# 阅读源码

符合大部分需求的开源项目有

 1. `mock.js` 
 2. `easy-mock`  
 3. `eolinker` 	
 4. `YAPI`
 5. `DOCCLEVER`

## MOCK.JS篇

首先我们需要明确现阶段大部门的 Mock 平台或多或少都是受到 `Mock.js` 的思想或者是其增强版。

我们可以用下面简单的 json 通过 `Mock.js`来构造数据:

```

example:

{
    "status|0-1": 0, //接口状态
    "message": "成功", //消息提示
    "data": {
        "counts":"@integer", //统计数量
        "totalSubjectType|1-4": [ //4-10意味着可以随机生成4-10组数据
            { 
              "subjectName|regexp": "大数据|机器学习|工具", //主题名
              "subjectType|+1": 1 //类型
            }
        ],
        "data":[
            {
                "name": "@name", //用户名
                "cname":"@cname",
                "email": "@email", //email
                "time": "@datetime" //时间
            }
        ]}
}

```

返回结果

```

{
    "status": 0,
    "message": "成功",
    "data": {
        "counts": 2216619884890228,
        "totalSubjectType": [
            {
                "subjectNameregexp": "大数据|机器学习|工具",
                "subjectType": 1
            },
            {
                "subjectNameregexp": "大数据|机器学习|工具",
                "subjectType": 2
            },
            {
                "subjectNameregexp": "大数据|机器学习|工具",
                "subjectType": 3
            },
            {
                "subjectNameregexp": "大数据|机器学习|工具",
                "subjectType": 4
            }
        ],
        "data": [
            {
                "name": "Ruth Thompson",
                "cname": "鲁克",
                "email": "z.white@young.gov",
                "time": "1985-02-06 05:45:21"
            }
        ]
    }
}

```

而且可以通过其 Mock.Random.extend() 来扩展自定义占位符.

```

example:

Random.extend({
    weekday: function(date) {
        var weekdays = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
        return this.pick(weekdays);
    },
    sex: function(date) {
        var sexes = ['男', '女', '中性', '未知'];
        return this.pick(sexes);
    }
});

console.log(Random.weekday());  // 结果: Saturday
console.log(Mock.mock('@weekday'));  // 结果: Tuesday
console.log(Random.sex());  // 结果: 男
console.log(Mock.mock('@sex'));  // 结果: 未知

```

来延伸所需进的拓展。

这个可以将自定义数据模型先进行解析，然后通过extend将其加入。

## easy-mock

`easy-mock` 是我参考的主要项目之一，它的UI交互非常符合我的设定，而且作为开源项目可以从它的源码中学到很多。

直接来看它提供接口编辑的页面

![imgn](http://img.haoqiao.me/build%20mock%201.png)

```

{
  data: {
    img: function({
      _req,
      Mock
    }) {
      return _req.body.fileName + '_' + Mock.mock('@image')
    }
  }
}

```

可以从上得之它既可以处理Mock数据模拟也可以处理函数，而且它内部有一套能处理req的内容。

先是在源码中找了一下，找到几个疑似点，但是不确定，还是在本地装好环境,主要是需要按照redis.然后启动服务去打几个断点输出。

根据经验先确定 `controllers\mock.js` 应该是处理数据模拟的地方。通过浏览源码并分析，最终定位于 297行处的代码

```

    await redis.lpush('mock.count', api._id)
    if (jsonpCallback) {
      ctx.type = 'text/javascript'
      ctx.body = `${jsonpCallback}(${JSON.stringify(apiData, null, 2)})`
        .replace(/\u2028/g, '\\u2028')
        .replace(/\u2029/g, '\\u2029') // JSON parse vs eval fix. https://github.com/rack/rack-contrib/pull/37
    } else {
      ctx.body = apiData
    }
```

首先是看到最终返回的 apiData 。用过 koa 或者 express 都应该清楚 ctx.body 的含义。然后我在上面写了句 `console.log(apiData)`。

然后在浏览器端发送请求。看下 node 端输出和浏览器端拿到的数据，基本可以肯定最终输出就是这个。

![imgn](http://img.haoqiao.me/build%20mock%202.png)

然后我们往上翻，可以看到这么一段代码:

```

 const vm = new VM({
        timeout: 1000,
        sandbox: {
          Mock: Mock,
          mode: api.mode,
          template: new Function(`return ${api.mode}`) // eslint-disable-line
        }
      })
      console.log('数据验证')
      console.log(mode)
      vm.run('Mock.mock(new Function("return " + mode)())') // 数据验证，检测 setTimeout 等方法
      apiData = vm.run('Mock.mock(template())') // 解决正则表达式失效的问题

```

通过查询了解到 VM 是一个沙盒，可以运行不受信任的代码。

大概就能了解 `easy-mock` 通过 vm 沙盒模式运行 mode 代码解析后返回结果。

核心代码就是 `Mock.mock( template )` 这么一句。根据数据模板生成模拟数据。

通过查文档了解 template 是可以直接内部写函数然后执行的。

这样解析的难度大大下降，发现原来并没有特别复杂的，依旧是依赖了 `Mock.js` 的原生方法。

然后我们可以看到 `easy-mock` 另一的操作就是可以获取 `请求参数_req`。也就是可以通过以下代码来根据请求参数返回指定数据。

```

{
  success: true,
  data: {
    default: "hah",
    _req: function({
      _req
    }) {
      return _req
    },
    name: function({
      _req
    }) {
      return _req.query.name || this.default
    }
  }
}

```

_req 一看就是从请求参数中获得的对象。

`Mock.js`是没有这个对象的，我们来找找源码中是哪里注入了这个对象。

还是在 `mock.js` 这个文件中第234行处找到

```

   Mock.Handler.function = function (options) {
      const mockUrl = api.url.replace(/{/g, ':').replace(/}/g, '') // /api/{user}/{id} => /api/:user/:id
      options.Mock = Mock
      options._req = ctx.request
      options._req.params = util.params(mockUrl, mockURL)
      options._req.cookies = ctx.cookies.get.bind(ctx)
      return options.template.call(options.context.currentContext, options)
    }

```

通过阅读 `MockJS` 的源码，了解到 `Handler`是处理数据模板的地方，打个断点再输出一次可以发现其实是在 `Mock.mock(new Function("return " + mode)())'` 之后传入的参数。

`options._req = ctx.request` 这句代码告诉了我们所谓的 `_req`是从哪里来的。

因此这个技术点我们也了解了是怎么做的，那么剩下一个灵活的支持 `restful` 通过阅读源码发现其实也没怎么处理，只是用 `pathToRegexp` 进行了一次验证。它先是在 `middlewares/index.js` 中 的 `mockFilter` 进行了路径正则。

```

static mockFilter (ctx, next) {
    console.log(ctx.path)
    const pathNode = pathToRegexp('/mock/:projectId(.{24})/:mockURL*').exec(ctx.path)
    console.log(pathNode)
    if (!pathNode) ctx.throw(404)
    if (blackProjects.indexOf(pathNode[1]) !== -1) {
      ctx.body = ctx.util.refail('接口请求频率太快，已被限制访问')
      return
    }
    console.log('通过筛选')

    ctx.pathNode = {
      projectId: pathNode[1],
      mockURL: '/' + (pathNode[2] || '')
    }

    return next()
  }

```

然后通过存在 `redis` 里的接口内容再进行了验证匹配。

```

const { query, body } = ctx.request
    const method = ctx.method.toLowerCase()
    const jsonpCallback = query.jsonp_param_name && (query[query.jsonp_param_name] || 'callback')
    let { projectId, mockURL } = ctx.pathNode
    console.log('ctx.pathNode', ctx.pathNode)
    const redisKey = 'project:' + projectId
    let apiData, apis, api
    console.log('通过URL匹配检验')
    apis = await redis.get(redisKey)
    console.log(apis)
    if (apis) {
      apis = JSON.parse(apis)
      console.log('pure apis', apis)
    } else {
      apis = await MockProxy.find({ project: projectId })
      console.log('find projectId', apis)
      if (apis[0]) await redis.set(redisKey, JSON.stringify(apis), 'EX', 60 * 30)
    }

    if (apis[0] && apis[0].project.url !== '/') {
      mockURL = mockURL.replace(apis[0].project.url, '') || '/'
    }

    api = apis.filter((item) => {
      const url = item.url.replace(/{/g, ':').replace(/}/g, '') // /api/{user}/{id} => /api/:user/:id
      return item.method === method && pathToRegexp(url).test(mockURL)
    })[0]
    console.log('api',api)

    if (!api) ctx.throw(404)

```

基本不匹配的路径请求都是在 `item.method === method && pathToRegexp(url).test(mockURL)` 这句代码里被拦截的。

非常优秀的代码。通读下来，加上断点对其思路逻辑学到了很多。

## eolinker

它的后端代码是 PHP 的，这就略过不看了。

## YAPI

它的核心后端处理代码是在 `mockServer.js` 里

有了之前的阅读经验很快找到处理 Mock 数据的地方

```

  let res;

        res = interfaceData.res_body;
        try {
            if (interfaceData.res_body_type === 'json') {
                res = mockExtra(
                    yapi.commons.json_parse(interfaceData.res_body),
                    {
                        query: ctx.request.query,
                        body: ctx.request.body,
                        params: Object.assign({}, ctx.request.query, ctx.request.body)                     
                    }
                );
                try {
                    res = Mock.mock(res);
                } catch (e) {
                    yapi.commons.log(e, 'error')
                }
            }
            
```

非常简单粗暴的处理方法。。。

对增强功能比较好奇在, 于是在 `common\mock-extra.js` 里找到了 `mock(mockJSON, context)` 方法。根据参数其实就能了解绑定上下文然后做了一些动作。这里就不展开详细。等之后开发的时候用到再去细读。因为这是做了其自己的增强的Mock功能,而暂时不需要这方面的考虑。

## DOClecer

这个项目是国内一个创业团队做的，我也加入了其官方群。虽然还没有用过。不过不妨碍阅读其源码了解思路。不过讲道理这个代码组织风格是挺糟糕的。。。

而且源码中不止一次出现了eval... 于是放弃参考。

# 写个小模块开心一下

通过阅读以上项目的源码，其实主要是前三个，感觉可以完成自己想要的需求了。那么先写一个小的来作为基础模块。

```

export const mock = async(ctx: any) => {
  console.log('mock')
  console.log(ctx)
  console.log(ctx.params)
  const method = ctx.request.method.toLowerCase()
  // let { projectId, mockURL } = ctx.pathNode
  // 获取接口路径内容
  console.log('ctx.pathNode', ctx.pathNode)
  // 匹配内容是否一致
  console.log('验证内容中...')
  // 模拟数据
  Mock.Handler.function = function (options: any) {
    console.log('start Handle')
    options.Mock = Mock
    // 传入 request cookies，方便使用
    options._req = ctx.request
    return options.template.call(options.context.currentContext, options)
  }
  console.log('Mock.Handler', Mock.Handler.function)
//   const testMode = `{
//     'title': 'Syntax Demo',
//     'string1|1-10': '★',
//     'string2|3': 'value',
//     'number1|+1': 100,
//     'number2|1-100': 100,
//     'number3|1-100.1-10': 1,
//     'number4|123.1-10': 1,
//     'number5|123.3': 1,
//     'number6|123.10': 1.123,
//     'boolean1|1': true,
//     'boolean2|1-2': true,
//     'object1|2-4': {
//         '110000': '北京市',
//         '120000': '天津市',
//         '130000': '河北省',
//         '140000': '山西省'
//     },
//     'object2|2': {
//         '310000': '上海市',
//         '320000': '江苏省',
//         '330000': '浙江省',
//         '340000': '安徽省'
//     },
//     'array1|1': ['AMD', 'CMD', 'KMD', 'UMD'],
//     'array2|1-10': ['Mock.js'],
//     'array3|3': ['Mock.js'],
//     'function': function() {
//         return this.title
//     }
// }`
const testMode = `{success :true, data: { default: "hah", _req: function({ _req }) { return _req }, name: function({ _req }) { return _req.query.name || this.default }}}`
  const vm = new VM({
    timeout: 1000,
    sandbox: {
      Mock: Mock,
      mode: testMode,
      template: new Function(`return ${testMode}`)
    }
  })
  vm.run('Mock.mock(new Function("return " + mode)())') // 数据验证，检测 setTimeout 等方法, 顺便将内部的函数执行了
  // console.log(Mock.Handler.function(new Function('return ' + testMode)()))
  const apiData = vm.run('Mock.mock(template())')
  console.log('apiData2333' , apiData)
  let result
  switch (method) {
    case 'get':
      result = success({'msg': '你调用了get方法'})
      break;
    case 'post':
      result = success({'msg': '你调用了post方法'})
      break;
    case 'put' :
      result = success({'msg': '你调用了put方法'})
      break;
    case 'patch' :
      result = success({'msg': '你调用了patch方法'})
      break;
    case 'delete' :
      result = success({'msg': '你调用了delete方法'})
      break;
    default:
      result = error()
  }
  // console.log(result)
  return ctx.body = result
}

```

这里调试的遇到一些问题，主要是一开始测试的时候发现 Mock 只将规则的数据模拟出，发现 function 类型的函数都没执行，一开始定位以为是`Mock.Handler.function` 在 ts 中未执行。于是在里面写了一个输出，发现的确没有。经过各种猜想和测试，发现是模拟mode有问题。

一开始我是这么写的

```

const testcode = {
    'array1|1': ['AMD', 'CMD', 'KMD', 'UMD'],
    'array2|1-10': ['Mock.js'],
    'array3|3': ['Mock.js'],
    'function': function() {
        return this.title
    }
}


```

事实上应该这么写

```

const testcode = `{
    'array1|1': ['AMD', 'CMD', 'KMD', 'UMD'],
    'array2|1-10': ['Mock.js'],
    'array3|3': ['Mock.js'],
    'function': function() {
        return this.title
    }
}`


```

参照 `easy-mock` 的思路可以实现一个基础的 Mock数据解析器，而且可以根据 koa 的特性同时支持 _req 的一些参数，这里先不加进去。

如何支持自定义的数据模型也有了基本的思路，在之前没有考虑 redis 情况下还是用传统的数据库查询。具体实现等后期再捣鼓出来再写出来。

## 结尾

通过这两天的学习，总算把一个Mock的核心模块该如何实现的思路给理顺了。

其实无论你是用户自定义数据,比如

```

{
  'user': User, // User是用户自定义的数据类型
   'string2|3': 'value',
   'number1|+1': 100,
    _req: function({
      _req
    }) {
      return _req
    },
    name: function({
      _req
    }) {
      return _req.query.name || this.default
    }
}

```

还是 Mock.js 原生的语法，你最终转换过来需要执行的是一样的内容，无非是在其转换前需要做一定的处理。只有搞懂了基本的数据模拟实现，基本上你可以将各个参数都做定制化。比如有的平台会将用户自己编写的函数一起和 json 拼接。其实用的最终核心思路还是一样的。




## 参考资料

[Mock.js使用](https://segmentfault.com/a/1190000008839142#articleHeader6)

[mockjs官方文档](http://mockjs.com/0.1/#Address)







