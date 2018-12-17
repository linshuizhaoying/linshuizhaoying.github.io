---
layout: post
title: PC端项目移动端化总过程
category: 技术
tags: [总结,开发,前端,移动端]
keywords: 总结,开发,前端,移动端]
description: 
---

## 前言
在之前的开发经验中，完整负责开发一个项目的PC端和移动端还未有过。
真正开发的时候，踩了一些坑，但是总体来说，在开发之前设计思路明确，技术难点枚举后无风险，那么将一个产品的移动端化其实比想象的简单。

### 设计之初

一开始设计的时候是先考虑现有的PC开发环境，将其转为移动端，因为团队小伙伴有专门移动端的开发环境，经过调研，因为项目PC端采用自研的React开发环境，而小伙伴移动端是采用Vue，而且代码风格非常不一致，在有限的开发时间中，原本想用通过移动端脚手架来帮助降低移动端组件编写工作量，但业务逻辑无法大量直接复用，因此，考虑再三，最终决定将原有PC端开发环境更改配置，去掉一些依赖，简化到最最简，引入`ant-mobile react mobx` 这三个依赖,其它功能尽量少引用模块而是用原生写法代替，最终项目打包在`700kb`以内。

### 开发之初

每个项目开发之前是最好理清思路，确定核心流程。然后开发核心页面，验证技术难点，再扩展的开发。

但是将PC端转为移动端思路变了一下，因为核心流程已经跑通，后台接口变化不大。而且很多逻辑代码可以复用，那么唯一需要确定的是移动端的布局，以及移动端新增的交互，比如下拉刷新、滚动加载、移动端图片上传、移动端特有的交互。

并且开发之前有个目标就是需要将大部分交互抽成组件进行复用，比如上传组件，高级查询组件，标签组件，条件筛选组件。

事实上花在组件上时间不会太多，因为原本就要写交互，而且开发组件思路就是确定参数，外部引用，同步调试。功能跑通，组件也对应有了雏形就可以着手优化。这样到后面复用的时候，如果有新的需求，加参数即可。

### 典型问题

1、TS环境问题

```

ts环境引入 antd mobile

需要在  tsconfig.json 加入 "allowSyntheticDefaultImports": true 

不然会报错

```

2、移动端ICON问题

```

由于 antd mobile不提供icon了

https://github.com/ant-design/ant-design-icons/tree/master/packages/icons/svg

需要自行手动下载svg

使用如下



<div style={{
width: '22px',
height: '22px',
background: `url(${require('../assets/icons/home.svg')}) center center / 21px 21px no-repeat` }
}

事实上 svg 不够多,可以去开源的站点下载

http://www.iconfont.cn/search/index?q=computer

```

3、移动端图片上传问题

```

写移动端图片上传组件时，发现formData 一直出错，一开始将base64直接作为file传过去，但是上传接口返回错误

后来读了vue版的移动端源码。看到

Object.keys( this.appendData ).map( dataKey => {
formData.append(`${dataKey}`, this.appendData[ dataKey ]);
});

不能够将formData独立出来，以json形式传，而是要将所有参数都加进去

formData.append('file', files[files.length - 1].url);
/** 图片body插入 */
formData.append('file', files[files.length - 1].file, file.name );
formData.append('categoryId', 'todo_web_id' );
formData.append('isPublic', '1');
formData.append('isInnerPublic', '0' );
// Object.keys( this.appendData ).map( dataKey => {
// formData.append(`${dataKey}`, this.appendData[ dataKey ]);
// });
// // this.config['file'] = formData;
console.log(file, formData, this.props.iac);
await axios({
method: 'post',
data: formData,
url: this.props.url,
headers: {
'x-iac-token': this.props.iac,
'Content-Type': 'multipart/form-data'
}
}).then(res=> {
console.log(res);
});

```

4、移动端样式问题

```

css中用background 引入svg

会有一个问题。如果svg内部是透明的。那么就会被其它底色覆盖。

因此需要做一定处理

width: 50px;
height: 50px;
background: url(../assets/icons/add.svg) center center / 50px 50px no-repeat;
position: absolute;
left: 80%;
background-color: white;
border-radius: 50%;
border: #00BD9D;

主要通过border来覆盖

```

5、TS与antd-mobile的冲突

```
利用antd mobile 添加拉动刷新功能时，因为ts配置，需要将所有的
pullToRefresh参数写全不然会报错

pullToRefresh={<PullToRefresh
refreshing={this.state.refreshing}
onRefresh={this.onRefresh}
getScrollContainer={() => {
return <div></div>;
}}
direction="down"
distanceToRefresh={25}
damping={100}
indicator={{
// activate: <div>activate</div>,
// deactivate: <div>deactivate</div>,
release: <div>加载中...</div>
// finish: <div>finish</div>,
}}
/>}

```

6、 下拉刷新不生效

```

引入 antd-mobile的tabs与下拉刷新组件。

官网例子以及github例子都是测试通过生效的，

先排查是否是样式 overscroll 会影响

后来测试发现更改后问题依旧存在

这时在真机测试发现滚动刷新很容易跳到另一个tab

猜测是tab的滚动机制导致下拉刷新无法生效。

给tabs 组件添加

swipeable={false}

后问题解决

```

### 鉴权

移动端的鉴权操作和pc端通过token或者单点登录直接校验是不一样的。因为涉及到微信企业号，鉴权流程需要变更。而且需要考虑到移动端有个场景就是传入对应详情ID，自动跳转到详情页。

```

const accessToken: string = getUrlParams(ctx.url, 'token');
  // 微信跳转而来的
  const code: string =  getUrlParams(ctx.url, 'code');
  const tokenCookie: string = ctx.cookies.get(COOKIE_TOKEN_NAME);

  // ctx.cookies.set(COOKIE_TOKEN_NAME, '');
  // 入口分两种，一种是跳转到详情，一种是访问列表
  // 详情Url: /todo?token=xxxx&todoId=xxxvvvv
  // 主入口: /todo
  // 如果有accessToken,去和后端兑换token，保存到cookie，然后渲染页面
  console.log('当前页面链接', ctx.url);
  console.log('accessToken', accessToken);
  if(accessToken) {
    console.log('有accessToken');
    const jwtToken: any = await request.sendRequest({
      url:  serverHost,
      method: 'get',
      ctx
    });
    console.log('此时 jwtToken', jwtToken);
    if(jwtToken.data) {
      const expires = new Date().getTime() + ACCESS_TOKEN_TIME * 1000;
      await redisHelper.set(jwtToken.data, expires, ACCESS_TOKEN_TIME);
      ctx.cookies.set(COOKIE_TOKEN_NAME, jwtToken.data, {
        maxAge: ACCESS_TOKEN_TIME * 1000
      });
      this.renderHtml(ctx);
    } else {
        // 如果连接过期,拿到的token是null,那么重新通过微信鉴权跳转
      const todoId: string = getUrlParams(ctx.url, 'todoId');
      const result = await ctx.redirect(`https://xxxx`);
      console.log('token过期后回跳 result', result);
    }
  } else {
    console.log('tokenCookie', tokenCookie);
    // 如果是访问列表入口,那么去查看是否有cookie，有的话直接渲染页面
    if(tokenCookie) {
      console.log('有cookie 直接渲染');
      this.renderHtml(ctx);
    } else {
      console.log('转到微信校验', 'code', code);
      // 如果已经鉴权了
      if(code) {
        const iacToken = await ciac.getToken();
        console.log('去拿用户');
        // 拿code换用户
        const username: any = await request.sendRequest({
          url:  `https://xxx`,
          path: '',
          method: 'get',
          ctx
        });
        // 拿用户换jwt
        const currentJwtToken: any = await request.sendRequest({
          url: 'https://xxx/jwt',
          method: 'post',
          ctx,
          data: {xxx}
        });
        console.log('username', username, 'currentJwtToken', currentJwtToken);
        // cookie设置
        const expires = new Date().getTime() + ACCESS_TOKEN_TIME * 1000;
        await redisHelper.set(currentJwtToken.data, expires, ACCESS_TOKEN_TIME);
        ctx.cookies.set(COOKIE_TOKEN_NAME, currentJwtToken.data, {
          maxAge: ACCESS_TOKEN_TIME * 1000
        });
        // 渲染页面
        this.renderHtml(ctx);
      } else {
        // 跳到企业号中间件,设置完cookie后渲染页面
        console.log('进入微信鉴权阶段');
        // 跳转回跳拿到code
                const result = await ctx.redirect(xxx);
        console.log('result', result);
        // this.renderHtml(ctx);
      }

    }
  }

```

拿到的手绘流程如下

![imgn](http://img.haoqiao.me/jianquanliuchengtuec467d8322d538edadb25a86b7adec3f.jpg)



### 兼容

一直出现一个问题就是iphone5的机子，列表总是超过宽度，导致模拟器下iphone5列表模块是可以左右移动的。一开始以为是设定的固定宽度过大，于是父元素加flex,子元素的max-width一点点调小。子元素加上`flex:1`,但作用不大。
后来思考是不是要百分比比较适合，但是给父元素加了`100%`，以及父元素之前的元素都加`100%`也不起作用。再次思考是不是父元素应该定个宽度，但是必须是自适应的宽度，于是改成`width:100vw`,之后再细调就可以了。


