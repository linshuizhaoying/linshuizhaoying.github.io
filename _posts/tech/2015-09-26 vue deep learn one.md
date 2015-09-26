---
layout: post
title: 初步研究Vue.js项目架构
category: 技术
tags: [Vue.js，Vue.js学习,Vue.js实例,前端学习,前端总结，Node.js]
keywords: 前端,资料,学习
description: 
---

## 背景介绍
最近在用vue.js来做项目。遇到一些问题。后来发现自己对于整体项目的掌控与与架构能力有点弱。在做第二个项目的时候发现自己有些东西一开始设定就有些问题。后来有人发了一份github上的项目，看到别人构建的项目非常清晰而且扩展性很强。也将初步讨论。(Ps:我假设你已经有了Node.js基础。)

### 自己的项目
其实应该还是自己没有完全领悟官方文档的精髓。因此多看别人的项目应该是提升自我最快的一种方式。

首先vue.js的组件化是这样的：

```
test.vue:

<style>

</style>

<template>

</template>

<script>
module.exports = {

  ready:function(){

  },
  data: function () {
      return {
     
    }
  }
}
</script>

```

然后通过

```
  components: {
    'test': require('./test')
  }
```

来引用组件。因此我一开始设想的组件化其实是这样的。将所需字段都按需加载。将所需获取的数据都以url形式传入，组件内部自动分析并且在ready时自动加载。

因此我的调用形式是这样的。

```
<adminadditem groupname="activity" desc="活动名称,活动时间,活动地点" class="activityName,DatePicker,activityArea" url="http://xxx.cn/add"></adminadditem>
```
因为组件里面数据获取是动态的，尤其是当你使用分页插件，你需要按需来进行渲染。这就造成了我还是在用原始的append，后来第二个项目中我改回了v-repeat的的形式。

然后数据的获取我直接用的是jquery的ajax获得数据。在某些情况下，你还需用同步请求以及保存this。

但实际上如果你仅仅在项目中只是需要获得后台数据，是不需要额外再加载jquery库。而是用fetch或者自己封装一个xmlHttpRequest。

我的项目结构是这样的：

```
├── Gulpfile.js
├── app
│   ├── admin.html
│   ├── images
│   ├── index.html
│   ├── public
│   │   ├── images
│   │   ├── javascripts
│   │   │   ├── build
│   │   │   │   ├── admin.App.js
│   │   │   │   ├── admin.App.js.map
│   │   │   │   ├── index.App.js
│   │   │   │   ├── index.App.js.map
│   │   │   │   ├── jquery.min.js
│   │   │   │   └── vue.min.js
│   │   │   └── src
│   │   └── stylesheets
│   │       ├── build
│   │       │   ├── css
│   │       │   └── font
│   │       └── src
│   └── vue-src
│       ├── admin.js
│       ├── admin.vue
│       ├── app.js
│       ├── app.vue
│       ├── components
│       ├── css
│       └── images
├── node_modules
├── package.json
├── publish
└── webpack.config.js

```
配合gulp和webpack可以加快开发效率。

但直到后来看了别人的项目，我打算重新设计项目结构。

### 别人家的项目

[Github地址](https://github.com/zerqu/qingcheng)

```
├── api.js
├── app.vue
├── components
│   ├── cafe-card.vue
│   ├── comment-form.vue
│   ├── comment-item.vue
│   ├── dropdown.vue
│   ├── hentry.vue
│   ├── login-form.vue
│   ├── logo.vue
│   ├── markdown-area.vue
│   ├── overlay.vue
│   ├── pagination.vue
│   ├── topic-form.vue
│   ├── topic-item.vue
│   ├── user-avatar.vue
│   ├── user-notifications.vue
│   └── webpage.vue
├── css
│   ├── animate.css
│   ├── base.css
│   ├── pygments.css
│   ├── responsive.css
│   ├── ui.css
│   └── yue.css
├── filters.js
├── ga.js
├── index.js
├── routers.js
├── utils.js
├── views
│   ├── cafe-list.vue
│   ├── cafe-members.vue
│   ├── cafe-topics.vue
│   ├── cafe.vue
│   ├── create-topic.vue
│   ├── edit-topic.vue
│   ├── home.vue
│   ├── topic.vue
│   ├── user-topics.vue
│   └── user.vue
└── xhr.js

```
从目录来看我们可以看到几个关键的文件,api.js index.js app.vue.

首先从api.js分析:

```

var http = require('./xhr');
'加载封装的ajax

http.defaults.afterRequest = function(req, duration) {
  ga('send', 'timing', 'AJAX', req.url, duration, req.status);
};
...
exports.timeline = function(params, cb) {
  var url = '/api/topics/timeline';
  ga('send', 'pageview', {title: 'Home'});
  http.get(url, params, cb);
};

exports.preview = function(text, cb) {
  http.post('/api/preview', {text: text}, cb);
};

...
```
显示加载了封装的http请求，下面又暴露一些调用http请求的模块。

然后来看下xhr.js写了什么：

```

var defaults = {};

function request(url, options) {
  var req = new XMLHttpRequest();
  req.error = function(cb) {
    options.error = cb;
  };

  var method = options.method || 'GET';
  var data = options.data;
  if (method === 'GET' && data) {
    url += '?' + Object.keys(data).filter(function(k){
      return k && data[k];
    }).map(function(k) {
      return k + '=' + data[k];
    }).join('&');
    data = null;
  }

...

exports.defaults = defaults;

exports.http = request;

exports.get = function(url, data, success, options) {
  return request(url, parseParams('GET', data, success, options));
};

exports.post = function(url, data, success, options) {
  return request(url, parseParams('POST', data, success, options));
};

exports.del = function(url, data, success, options) {
  return request(url, parseParams('DELETE', data, success, options));
};

exports.put = function(url, data, success, options) {
  return request(url, parseParams('PUT', data, success, options));
};

...
```
手动封装了一个XMLHttpRequest,并且最后暴露了get,post,del,put方法。
回到api.js,可以看到还有

```
ga('send', 'timing', 'AJAX', req.url, duration, req.status);

```
但是这里api.js并没有载入ga，那么可以猜测肯定某个地方预先加载了ga.js。

查看ga.js源代码发现这是段加载Google Analytics（分析）的代码。嗯，对现阶段的我并没什么用。

重新纵览全局，可以发现他组件的构建是分components和views。而我是直接将页面各个模块抽成组件。事实上如果真的为了以后扩展方便还是应该学习他的思想将组件细分开。

之后翻看components目录下的vue文件，从第一个开始cafe-card.vue:

```
module.exports = {
    replace: true,
    props: ['cafe', 'subpath'],
    computed: {
      link: function() {
        var url = '/c/' + this.cafe.slug;
        if (this.subpath) url += this.subpath;
        return url;
      },
      color: function() {
        var style = this.cafe.style;
        var rv = {};
        if (style.color) {
          rv['background-color'] = style.color;
        }
        if (style.cover) {
          rv['background-image'] = 'url(' + style.cover + ')';
        }
        return rv;
      },
      description: function() {
        return this.cafe.description || 'No description'
      }
    },
    components: {
      'user-avatar': require('./user-avatar.vue')
    }
  }
  
```

可以发现跟官网的大型项目组件一样,但他把数据的内容全写在了coputed里。这让我除了ready就是created的感到羞愧-0-还是自己没有细看官方文档，导致一些东西没放对位置。

然后下一个comment-form.vue:

```
<template>
  <form class="comment-form" v-on="submit: formSubmit" v-el="form">
    <div class="comment-form-mask" v-on="click: showLogin" v-if="!user.id"></div>
    <user-avatar user="{{ user }}" v-if="user.id" class="small circle"></user-avatar>
    <markdown-area class="comment-item" placeholder="Write your response" content="{{@ comment }}" v-ref="textarea"></markdown-area>
    <button v-if="user.id">Reply</button>
  </form>
  //这段真的把vue状态驱动的理念用上了。
  //v-if这些指令让判断用户登录状态非常容易。
</template>

```
接下来是comment.item.vue:

```
<template>
  <li id="c-{{ comment.id }}" class="comment-item item-container" v-show="comment.id" v-transition="fade" v-class="comment-hide: isHide">
    <user-avatar user="{{user}}"></user-avatar>
    <div class="comment-main item-content">
      <div class="comment-info">
        <a href="/u/{{user.username}}">{{user.username}}</a>
        <time datetime="{{ comment.created_at }}">{{ comment.created_at | timeago }}</time>
        #{{ comment.id }}
        <div class="comment-actions">
          <span v-show="comment.like_count">{{ comment.like_count }} likes</span>
          <a class="tip tip-west like-comment" role="button" href="javascript:;" aria-label="like this comment"
          v-if="!isOwner" v-on="click: toggleLike" v-class="liked: comment.liked_by_me"> //这里的isOwner是在直接在computed里返回的判断结果
            <i class="qc-icon-heart"></i>
          </a>
          <a role="button" href="javascript:;" v-if="!isOwner" v-on="click: flagComment" aria-label="report spam"><i class="qc-icon-flag"></i></a>
          <a role="button" href="javascript:;" v-if="isOwner" v-on="click: deleteComment" aria-label="delete comment"><i class="qc-icon-bin"></i></a>
        </div>
      </div>
      <div class="comment-content" v-html="content"></div>
    </div>
  </li>
</template>

<script>
  var api = require('../api');
  module.exports = {
    replace: true,
    props: ['comment'],
    computed: {
      user: function() {
        return this.comment.user;
      },
      isOwner: function() {
        return this.$root.user.id === this.user.id;
      },
      isHide: function() {
        return this.comment.flag_count > 5;
      },
      content: function() {
        var content = this.comment.content;
        // > is for end of the tag: `<p>`
        content = content.replace(/(>|\s)@([0-9a-z]+)/g, '$1<a href="/u/$2">@$2<\/a>');
        return content;
      }
    },
    methods: {
      deleteComment: function() {
        if (confirm('Are you sure to delete this comment?')) {
          api.comment.delete(this.comment, function() {
            this.comment.id = null;
            this.$parent.topic.comment_count -= 1;
          }.bind(this));
        }
      },
      flagComment: function() {
        if (confirm('Are you sure to report this comment?')) {
          api.comment.flag(this.comment)
        }
      },
      toggleLike: function() {
        var comment = this.comment;
        if (comment.liked_by_me) {
          api.comment.unlike(comment, function(resp) {
            comment.liked_by_me = false;
            comment.like_count -= 1;
          });
        } else {
          api.comment.like(comment, function(resp) {
            comment.liked_by_me = true;
            comment.like_count += 1;
          });
        }
      },
    },
    components: {
      'user-avatar': require('./user-avatar.vue')
    }
  }
</script>
```
对着api的comment模块我们可以更加直观的理解

```
exports.comment = {
  url: function(comment) {
    return '/api/topics/' + comment.topic_id + '/comments/' + comment.id;
  },
  create: function(tid, payload, cb) {
    var url = '/api/topics/' + tid + '/comments';
    http.post(url, payload, cb);
  },
  delete: function(comment, cb) {
    http.del(this.url(comment), cb);
  },
  like: function(comment, cb) {
    http.post(this.url(comment) + '/likes', cb);
  },
  unlike: function(comment, cb) {
    http.del(this.url(comment) + '/likes', cb);
  },
  flag: function(comment, cb) {
    http.post(this.url(comment) + '/flag', cb);
  }
};
```
他对每一个模块的细化把握非常到位，尤其是对vue.js的组件概念理解很透彻。这才是真正看懂官方文档，善用指令以及对每一个细节把握都很到位。给人一种醍醐灌顶的感觉。而且他对于闭包的理解已经达到随心所欲了。这些代码每部分都很直观，完全抛开其它都能看懂
对于components目录大概到这里,有兴趣可以自己去翻翻。肯定会有不少收获。

接下来是view层,第一个cafe-list.vue:

```
<template>
  <div class="body cafe-list">
    <logo class="loading center" v-if="$loadingRouteData"></logo>

    <div class="section container" v-if="following.length">
      <h2 class="section-title">Following</h2>
      <div class="cafe-cards clearfix">
        <cafe-card v-repeat="cafe: following" subpath="{{ subpath }}" track-by="id"></cafe-card>
      </div>
    </div>

    <div class="section container">
      <h2 class="section-title">Cafes</h2>
      <div class="cafe-cards clearfix">
        <cafe-card v-repeat="cafe: cafes" subpath="{{ subpath }}" track-by="id"></cafe-card>
      </div>
    </div>
  </div>
</template>

<script>
var api = require('../api');

module.exports ={
  data: function() {
    return {
      cursor: 0,
      following: [],
      cafes: []
    }
  },
  computed: {
    subpath: function() {
      if (this.$route.query.show === 'create') {
        return '/create';
      }
      return '';
    }
  },
  route: {
    data: function(transition) {
      api.cafe.list(function(resp) {
        transition.next({
          following: resp.following || [],
          cafes: resp.data,
          cursor: resp.cursor,
        })
      });
    }
  },
  components: {
    'cafe-card': require('../components/cafe-card.vue'),
    'logo': require('../components/logo.vue')
  }
};
</script>
```
这种view层的vue写法应该算是非常棒的。它用上了vue.js自带的route,然而我之前写的都没考虑到route，丢人=-=。

[vue.js Route官方文档](http://vuejs.github.io/vue-router/zh-cn/index.html)

这种写法给我一种rails的感觉。以及当初用node.js写后台route。考虑的很周全。

## 总结

```
1.设计好api.js
2.封装常规http请求(这个可以抄现成的-0-)
3.组件的细分
  1.通用组件
  2.view层组件
4.灵活使用route定位。

```
接下来就是根据总结来重新实现一个项目开发基础架构。

