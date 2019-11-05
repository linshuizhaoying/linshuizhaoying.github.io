---
layout: post
title: 协助开发公众号功能模块的小结
category: 技术
tags: [总结,开发,前端,开发, 微信公众号,html]
keywords: 总结,开发,前端,开发, 提升效率]
description: 
---

# 前言

最近接了一个其它部门的临时插入的项目。整个开发流程一言难尽，所幸也积累了一些经验，以及在开发中深刻感受到的问题。

# 一些问题

首先这是一个公众号的功能模块，分四个页面，属于某个产品的一个功能模块，按道理这玩意应该是该产品团队开发，但是没有前端，因此找人开发。

因为该项目是之前有个前端开发了一年，因此我接手前觉得只是在其中开发模块，应该还是可以的，但是没想到，没有磨合过的团队总会出现奇奇怪怪的坑。

于是对微信公众号开发0经验，凭借一些h5的经验，加上google，磕磕碰碰去熟悉开发。

## 开发评审

这个对我来说基本没有，这属于该项目最坑的地方，需求有了，设计有了，后端接口开发了，想到没前端了，需求评审也在找到我之前过了，导致我最终拿到的是需求稿和设计稿，沟通基本靠微信。

对于一个正常的项目来说，一个前端人员却不知道交互细节，那么开发中就会返工。实际上也遇到了需求稿上没有注明的一些细节，导致浪费了不少时间。

当然这还和个人有一点关系，没有去深入了解所有交互细节，按道理来说不应该，问题是出在一开始就不想给他们开发，内心一开始就排斥，怎么可能会去主动，拿到项目源码和交互稿都是直接按流程去了解，把精力用在实现功能上。这就导致后面返工。

总的来说，遇到插入项还是要调整心态。

## 前端项目的文档

在接手一个前端项目时，固定流程就是

```

1、了解代码结构的组织
2、了解api的组织
3、了解接口调用和store管理
4、写demo

```

一般情况下，已经开发了一年的项目应该这些都已经固化，比较好上手，但是问题出现在了没有文档，所有沟通靠微信，这就很难受，因为你很难确定东西是不是该放你想要的位置，以及代码明明复制过来，只改了参数为什么报错？等等问题的出现。

问题出现在接口调用，因为封装了一个函数来作为请求，但是微信的校验机制加上业务的某些校验导致它的方法有点奇怪。但是没有说明让人排错的时候很容易遗漏，最终导致开发的时候在接口联调上双方都不知道问题出现在哪。问之前的前端也没法定位问题所在，这就导致耗时很久，提出延期。后来发现是因为接口调用的方法里有个参数不常见，需要加上。


## 接口问题

接口一般就接口质量和自测。这点后端如果不保证，那么前端开发的时候就很被动。

当然这属于联调的问题。

问题更多出现在一开始，有线上的接口地址，但是文档不全，或者有信息没补上，然后后端给了一份markdown的文档，后来又补充了一份。

导致最终对接接口的时候不清不楚。

后来是让其在需求稿把对应页面需要的接口标注起来，前端再进行修改。

## 小结

可以看出，明明挺简单的功能模块开发，却可以把一些不该踩的坑都覆盖了，说明从源头就出现了问题。所以我一开始就挺讨厌去接手其它项目的模块开发，因为它实在坑太多了。

# 一些功能的实现

## 长按生成长图片分享保存

这个需求很常见，原理通过第三方库`html2canvas`来实现，需要注意的是在微信企业号会有图片跨域的情况，解决方案一个是node端生成一个接口来转发，一个是接收base64。

```
service:

async image(url) {
  const ctx = this.ctx;
  const { app } = ctx;
  const result = await app.fetch(ctx.query.url, null, {
    contentType: null,
    dataType: null,
  });
  return result;
}

controller:

// 获取图片，为图片添加 CORS 头部
async image() {
const { ctx } = this;
const { service } = ctx;
const result = await service.data.image();
if (result.status === 200) {
  ctx.set('Cache-Control', 'max-age=2592000');
}
ctx.set('Access-Control-Allow-Origin', '*');
ctx.body = result.data;
}


page:

const IMAGE_URL = "/data/api/image";
imgSrc(url) {
  return `${IMAGE_URL}?url=${encodeURIComponent(url)}`;
},

```

生成长图片逻辑就是页面塞张空图，然后隐藏，点击生成后，通过转换拿到base64的图片数据塞到空图里，然后展示即可:

```

<div
  v-if="isPageShow"
  ref="page"
  :class="{
    'sharing': this.isShare,
    'medal-wall': true,
  }"
>
 </div>

<img
  @touchstart="touchIn"
  @touchend="cleartime"
  @touchmove="touchMove"
  v-if="isShare"
  crossorigin="anonymous"
  :src="pageImageSrc"
  class="page-image"
  alt
/>


generateImg() {
  html2canvas(this.$refs.page, {
    useCORS: true,
    dpi: window.devicePixelRatio,
    height: this.$refs.page.clientHeight,
    allowTaint: true,
    logging: true
  })
    .then(canvas => {
      canvas.toDataURL("image/png", 1.0);
      this.pageImageSrc = canvas.toDataURL("image/png", 1);
      this.isPageShow = false;
      this.tipShow = true;
    })
    .catch(e => {
      console.error("error", e);
    });
},

```



## 微信公众号去掉分享按钮组

要实现的效果是：

![imgn](http://img.haoqiao.me/blog871572944598_.pic.jpg)

主要就是在微信配置加载完成后去做对应因此操作。这个在微信开发者工具是看不到效果，必须发到测试环境，在测试公众号打开才有效。


```
axios({
        url: `/xxx/js_config`,
        method: "get",
        params: {
          jsApiList:"onMenuShareAppMessage,onMenuShareTimeline,hideMenuItems,hideOptionMenu",
          url: encodeURIComponent(location)
        },
        xsrfCookieName: "csrfToken",
        xsrfHeaderName: "x-csrf-token"
      }).then(res => {
        const configs = res.data;
        wx.config(configs.data);
        wx.ready(() => {
          wx.hideOptionMenu();
          wx.hideMenuItems({
            menuList: ["menuItem:share:appMessage", "menuItem:share:timeline"] // 要隐藏的菜单项，只能隐藏“传播类”和“保护类”按钮，所有menu项见附录3
        });
          });
wx.error(err => {
  console.error(err);
});
          
```



# 一些问题的解决

## 服务端渲染与引入前端模块的冲突

```

server render bundle error, try client render, the server render error ReferenceError: window is not defined
node_modules/html2canvas/dist/html2canvas.js:6907:29

```

在服务端渲染的时候，node报错，导致部分渲染没法进行。

问题就出现在页面里面 `import` 了 `html2canvas` 这个只在前端环境正常跑的模块，解决方案很简单，服务端渲染不支持的前端模块全部不直接引用，动态载入保存到window即可。

```


if (typeof window !== 'undefined') {
  window.html2canvas = require('html2canvas');
}


在vue项目中

  created() {
    if (typeof window !== 'undefined') {
      window.html2canvas = require('html2canvas');
    }
  },


```

## 长按图片埋点与图片滚动的冲突

因为需要埋点用户长按图片保存的行为，一开始的方案是给`img`加上touch事件的监控

```

<img
  @touchstart.prevent="touchin"
  @touchend.prevent="cleartime"
  v-if="isShare"
  crossorigin="anonymous"
  :src="pageImageSrc"
  class="page-image"
  alt
/>

touchin() {
  clearInterval(this.Loop); //再次清空定时器，防止重复注册定时器
  this.Loop = setTimeout(() => {
    // 这里长按后执行的操作
    console.log("长按");
    // 长按分享埋点
    this.buriedDataShare('medalWallPageImgShare');
  }, 1000);
},
// 这个方法主要是用来将每次手指移出之后将计时器清零
cleartime() {
  clearInterval(this.Loop);
}


```

虽然实现了监控长按的行为，但是也造成了默认的一些行为被禁止了，比如跨屏长图的滚动行为。

搜了一下网上没有类似的情况，后来搜资料看了一些vue下滚动行为的库，看到有人封装了touchMove的方法。

于是借鉴这个思路自己来实现兼容.

```

<img
  @touchstart="touchIn"
  @touchend="cleartime"
  @touchmove="touchMove"
  v-if="isShare"
  crossorigin="anonymous"
  :src="pageImageSrc"
  class="page-image"
  alt
/>
    
touchIn(event) {
  this.Loop = setTimeout(() => {
    if (this.Loop) {
      this.buriedDataShare("medalWallPageImgShare");
    }
  }, 1000);
  clearInterval(this.Loop); //再次清空定时器，防止重复注册定时器
},
cleartime() {
  clearInterval(this.Loop);
  this.Loop = null;
},
touchMove(event) {
  this.cleartime();
}

```

比较简单，就是有滚动行为就不触发，如果是长按就触发埋点。





