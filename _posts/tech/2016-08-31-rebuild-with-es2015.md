---
layout: post
title: 重构UI管理系统基础篇1
tags: [学习,私货,前端,基础整理]
keywords: Javascript,代码,前端,学习总结
description: 
---

## 前言

重构项目之前一般都先找几个成熟的开源项目作为参考，看看别人的思路。
首先确定一下自己需要重构哪些，带着自己的目标去翻看源码总会事半功倍。

```

1.项目结构
2.代码风格
3.javascript源码需要转为ES6
4.ajax需要替换为其它方式
5.去除编程过程中快速开发遗留的冗余代码以及优化

```

## 准备工作

第一个项目我找的是`vue-ghpages-blog` 这里不贴地址，自行去github搜一下就行了。

先看这份项目的组织方式:

```
├── dist
│   ├── app.js
│   └── vendors.js
├── favicon.ico
├── index.html
├── package.json
├── src
│   ├── components
│   │   ├── App.vue
│   │   ├── ListView.vue
│   │   └── PostView.vue
│   ├── filters
│   │   └── index.js
│   ├── main.js
│   ├── setting
│   │   └── index.js
│   ├── store
│   │   └── index.js
│   └── templates
│       └── index.html
└── webpack.config.js

```

粗略看了下代码，发现比较有学习价值的只有store目录下的Index.js 因为就这部分包含了ES6的一些特性。来看一下这个文件内容,我将直接分解内容。

```
// vendor
//ES6模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。

import { EventEmitter } from 'events';
import { Promise } from 'es6-promise';
// ES6模块
上面代码的实质是从events模块加载EventEmitter方法，其他方法不加载。这种加载称为“编译时加载”，即ES6可以在编译时就完成模块加载，效率要比CommonJS模块的加载方式高。当然，这也导致了没法引用ES6模块本身，因为它不是对象。

// setting
import setting from '../setting';

// https://developer.github.com/v3/repos/#get
const LIST_API_URL = `https://api.github.com/repos/${setting.config.repo}/contents/${setting.config.path}?ref=${setting.config.branch}`;


let store = new EventEmitter();
//EventEmitter可以说是观察者模式的体现,所以实现概念可以理解为基于事件的监听和发布


export default store;//让ES6模块有一个默认的导出

/**
 * fetch post content from github
 *
 * @param title
 * @returns {Promise}
 */
 
 
 /*
 基础补充
 ES6允许使用“箭头”（=>）定义函数。

	var f = v => v;
	上面的箭头函数等同于：
	
	var f = function(v) {
	  return v;
	};
 
 
 
 
 所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise是一个对象，从它可以获取异步操作的消息。Promise提供统一的API，各种异步操作都可以用同样的方法进行处理。
 
 Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。它们是两个函数，由JavaScript引擎提供，不用自己部署。

resolve函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从Pending变为Resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；reject函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从Pending变为Rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

Promise新建后就会立即执行。
 
 
 */
store.getPost = (title) => {

    const POST_API_URL = `https://api.github.com/repos/${setting.config.repo}/contents/${setting.config.path}/${title}?ref=${setting.config.branch}`;

    return new Promise((resolve, reject) => {

        const xhr = new XMLHttpRequest();

        xhr.open('GET', `${POST_API_URL}`);
        // https://developer.github.com/v3/media/#html
        xhr.setRequestHeader("Accept", "application/vnd.github.v3.html");
        xhr.onload = () => {
            const resText = xhr.responseText;
            resolve(resText);
        };
        xhr.onerror = () => reject;
        xhr.send();

    });
}

/**
 * fetch the list from cache or from github api
 *
 * @todo paginate not yet
 *
 * @param page
 * @returns {Promise}
 */
store.getListByPage = (page = 1) => {
    return new Promise((resolve, reject) => {

        if (sessionStorage && sessionStorage.getItem('posts')) {

            // read data from cache
            resolve(JSON.parse(sessionStorage.posts));

        } else {

            const xhr = new XMLHttpRequest();

            xhr.open('GET', LIST_API_URL);
            xhr.onload = () => {
                const resJson = xhr.responseText;
                // caching
                sessionStorage.setItem('posts', resJson);

                resolve(JSON.parse(resJson));
            };
            xhr.onerror = () => reject;
            xhr.send();

        }

    });

}

```

从上段代码和以前看过的几个项目来看，filters目录好像很常见，把一些通用的处理数据的函数放进去多个地方调用。这是优化的一部分。

接下来是三个版本不同的cnodejs的写法。其中一个是大概去年拿到的学习版本。

```
版本1的目录结构：
├── LICENSE
├── README.md
├── dist
│   ├── app.css
│   ├── app.css.map
│   ├── app.js
│   ├── app.js.map
│   ├── dcddd5036c95dcdc001313b59b500fdf.gif
│   └── main.js
├── favicon.ico
├── index.html
├── npm-debug.log
├── package.json
├── server.js
├── src
│   ├── api.js
│   ├── app.js
│   ├── app.vue
│   ├── components
│   │   ├── loading.vue
│   │   ├── message-default.vue
│   │   ├── modal.vue
│   │   ├── nickname.vue
│   │   ├── no-data.vue
│   │   ├── profile.vue
│   │   ├── read.vue
│   │   ├── slide.vue
│   │   ├── tips.vue
│   │   └── unread.vue
│   ├── css
│   │   ├── about.css
│   │   ├── app.css
│   │   ├── common.css
│   │   ├── home.css
│   │   ├── layout-box.css
│   │   ├── layout.css
│   │   ├── message.css
│   │   ├── normalize.css
│   │   ├── post.css
│   │   ├── profile.css
│   │   └── topic.css
│   ├── directives.js
│   ├── filters.js
│   ├── fonts
│   │   ├── iconfont.svg
│   │   └── iconfont.ttf
│   ├── images
│   │   ├── bg.jpg
│   │   ├── bg.png
│   │   ├── github.ico
│   │   ├── loading.gif
│   │   └── loading.png
│   ├── router.js
│   └── views
│       ├── about.vue
│       ├── home.vue
│       ├── login.vue
│       ├── message.vue
│       ├── post.vue
│       ├── tail.vue
│       └── topic.vue
├── static
│   └── vue.js
├── webpack.config.babel.js
├── webpack.config.prod.babel.js

```

这个版本有个api.js里面，里面内容如下：

```
import promise from "es6-promise"
import "whatwg-fetch"

export let getList = async (page, tag) => {
	let response = await fetch(`https://cnodejs.org/api/v1/topics?page=${page}&limit=20&tab=${tag}`, {
		mode: "cors"
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}

export let getTopic = async (topicId) => {
	let response = await fetch(`https://cnodejs.org/api/v1/topic/${topicId}`, {
		mode: "cors"
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}

export let login = async (token) => {
	let response = await fetch(`https://cnodejs.org/api/v1/accesstoken `, {
		method: "POST",
		mode: "cors",
		headers: {
			"Content-Type": "application/x-www-form-urlencoded"
		},
		body: `accesstoken=${token}`
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}

export let like = async (id, token) => {
	let response = await fetch(`https://cnodejs.org/api/v1/reply/${id}/ups`, {
		method: "POST",
		mode: "cors",
		headers: {
			"Content-Type": "application/x-www-form-urlencoded"
		},
		body: `accesstoken=${token}`
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}

export let reply = async (token, topicId, content, replyId) => {
	let body = replyId ? `accesstoken=${token}&content=${content}&reply_id=${replyId}` : `accesstoken=${token}&content=${content}`

	let response = await fetch(`https://cnodejs.org/api/v1/topic/${topicId}/replies`, {
		method: "POST",
		mode: "cors",
		headers: {
			"Content-Type": "application/x-www-form-urlencoded"
		},
		body: body
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}

export let getProfile = async (nickname) => {
	let response = await fetch(`https://cnodejs.org/api/v1/user/${nickname}`, {
		mode: "cors"
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}

export let getMessages = async (token) => {
	let response = await fetch(`https://cnodejs.org/api/v1/messages?accesstoken=${token}`, {
		mode: "cors"
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}

export let getMessageCount = async (token) => {
	let response = await fetch(`https://cnodejs.org/api/v1/message/count?accesstoken=${token}`, {
		mode: "cors"
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}

export let post = async ({token, title, tab, content}) => {
	let response = await fetch("https://cnodejs.org/api/v1/topics", {
		method: "POST",
		headers: {
			"Content-Type": "application/x-www-form-urlencoded"
		},
		mode: "cors",
		body: `accesstoken=${token}&title=${title}&tab=${tab}&content=${content}`
	}).catch((error) => {
		console.log(error)
	})

	return await response.json().catch((error) => {
		console.log(error)
	})
}
```

`await` 和 `async`是es7的异步解决方案。然后用了fetch来获取数据。
其它部分优化部分不明显也就不看了~

另一个版本`cnodejs-vue`它的主文件Main.js含有这么一段：

```
router.beforeEach((transition) => {
  document.body.scrollTop = 0;
  const token = getToken(store.state);
  if (token) {
    fetchMsgCount(store, token)
      .catch((e) => console.log(e));
  }
  if (transition.to.auth) {
    if (token) {
      transition.next();
    } else {
      const redirect = encodeURIComponent(transition.to.path);
      transition.redirect({ name: 'login', query: { redirect } });
    }
  } else {
    transition.next();
  }
});

```
它把用户验证这块放在了路由这边处理。router.beforeEach(hook).这个函数会在路由切换开始时调用。调用发生在整个切换流水线之前。这个适合单页面的，但是我项目用了好几个路由，包括前端和后台的，因此即使是分拆好像也不是很适合。

而且这个项目引用了vuex，关于vuex我具体还不是很了解。因此先跳过。不过值得一提的是它数据获取也是通过fetch来获取的。

第三个版本的`vue-cnodejs`它的main.js中也写了路由中间验证

```
//登录中间验证，页面需要登录而没有登录的情况直接跳转登录
router.beforeEach((transition) => {
    //处理左侧滚动不影响右边
    $("html, body, #page").removeClass("scroll-hide");
    FastClick.attach(document.body);

    if (transition.to.auth) {
        if (localStorage.userId) {
            transition.next();
        } else {
            var redirect = encodeURIComponent(transition.to.path);
            transition.redirect('/login?redirect=' + redirect);
        }
    } else {
        transition.next();
    }
});

```

可以对比一下，一个把id放到vuex，一个放到了localStorage。我写的是存储在redis和localStorage里。

第三个版本的数据获取是用ajax。

而且这三个版本有个共同点就是都使用了scss来写css.

在某个vue版知乎日报里我发现这么一段代码:

```
    data: function() {
        return {
            msg: "article",
            api: {},
            title: "今日要闻",
            tin:Tin
        }
    },
    methods: {
        err: function(event) {
            var _this = event.target;
            _this.style.visibility = "hidden";
        }
    },
    route: {
        data: function() {
            var query = this.$route.query;
            //$route.query对象，包含路由中查询参数的键值对。例如，对于 /foo?user=1 ，会得到 $route.query.user == 1 。
            
            this.$http.get(Tin+'/tiny-theme', {
                params: {
                    id: query.id
                }
            }).then(function(response) {
                this.$data.api = response.json();
                this.$data.title = query.name;
                this.$root.$data.headerTitle = query.name;
            });
        }
    },
```
这是在artcile.vue里面的，它获取数据是通过路由加载的时候用vue resource来获取.这是让组件去主动获取数据。可以借鉴一下。

最后一个项目是vue版豆瓣。
它的路由直接写在router.js里面

```
	app.route('/movie/:id')
			 .get(Movie.detail)
			 .delete(MovieComment.del);

	// User.signinRequired 用户登录控制   User.adminRequired 用户权限控制

	// 用户评论路由
	app.post('/admin/movie/movieComment',User.signinRequired,MovieComment.save);

```
从这么一小段我们可以看到它代码写的也蛮严谨。

然后翻出app.js我们来看它的用户信息存储在哪

```
    session = require('express-session'),  				      // session依赖cookie模块
    mongoStore = require('connect-mongo')(session),		  // 对session进行持久化
    
    ...
    
app.use(session({
	secret:'douban',                          // 设置的secret字符串，来计算hash值并放在cookie中
	resave: false,                                    // session变化才进行存储
	saveUninitialized: true,
	// 使用mongo对session进行持久化，将session存储进数据库中
	store: new mongoStore({
		url: dbUrl,                                     // 本地数据库地址
		collection: 'sessions'                          // 存储到mongodb中的字段名
	})
}));

```
用Mongodb的扩展来做session持久化。

这块有个地方值得学习的是它的schemas部分：

这边获取数据又有个小变化，直接看代码


```
export default {
		name: 'ChooseMovies',
		components: {
			ChooseMoviesTitle,
			ChooseMovieItem
		},
		data() {
			return {
				title: ['热门','最新','经典','豆瓣高分','冷门佳片','华语','欧美','韩国'],	// 标题列表
				selected: '热门',													// 当前选择标题
				dataList: [],															// 存储Ajax返回的数据
				currentData: {}														// 当前标题显示数据
			}
		},
		created() {																		// 实例创建之后调用getData方法请求该标题对应的数据
			this.getData(this.selected);
		},
		watch: {																			// 监听selected属性，属性发生变化执行getData方法
			selected: 'getData'													// 该属性是双向绑定，当子组件中改变该属性会触发getData方法
		},
		methods: {
			getData(value) {
				// 在dataList中查找该value值是否存在，若存在则说明该数据已请求过，无需再次发送get请求
				for(let item of this.dataList) {
          if(item.name.includes(value)) {
            this.currentData = item;
            return;
          }				
				}
				// dataList中没有该标题数据 发送新请求到服务器
				let url = '/?fliterName=' + encodeURIComponent(value + '电影');
				$.get(url, (results) => {
					this.currentData = results.data;				 // 该属性值变化会让榜单该属性的子组件重新渲染
					this.dataList.push(results.data);				 // 添加到dataList中以便再次请求时进行判断
				});
			}
		}
	};
</script>

```

它是对面板所选择的内容进行颗粒化请求数据。

## 结尾

看了这些项目，大概对自己的项目重构有了一定想法。接下来就是一点点去重构了~


