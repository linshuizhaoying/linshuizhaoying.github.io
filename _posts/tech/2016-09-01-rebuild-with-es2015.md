---
layout: post
title: 重构UI管理系统实战篇
category: 技术
tags: [学习,私货,前端,实战总结]
keywords: Javascript,代码,前端,学习总结
description: 
---

## 前言
现将现阶段目录结构罗列出来：

```
1
├── README.md
├── app.js
├── bin
│   └── www
├── data
│   ├── link.js
│   ├── userController.js
│   └── userModel.js
├── index.html
├── package.json
├── routes
│   ├── index.js
│   └── uimanage.js
├── src
│   ├── App.vue
│   ├── admin.js
│   ├── assets
│   │   ├── css
│   │   │   ├── animated.css
│   │   │   ├── date-tip.css
│   │   │   ├── font-awesome.min.css
│   │   │   ├── hover.css
│   │   │   ├── reset.css
│   │   │   └── uiManage.css
│   │   ├── fonts
│   │   │   ├── FontAwesome.otf
│   │   │   ├── fontawesome-webfont.eot
│   │   │   ├── fontawesome-webfont.svg
│   │   │   ├── fontawesome-webfont.ttf
│   │   │   ├── fontawesome-webfont.woff
│   │   │   └── fontawesome-webfont.woff2
│   │   ├── img
│   │   │   ├── cd-arrow.svg
│   │   │   ├── cd-avatar.png
│   │   │   ├── cd-logo.svg
│   │   │   ├── cd-nav-icons.svg
│   │   │   ├── cd-search.svg
│   │   │   └── logo.png
│   │   └── js
│   │       └── trianglify.min.js
│   ├── components
│   │   ├── adminmanage.vue
│   │   ├── adminusermanage.vue
│   │   ├── advanced-ui.vue
│   │   ├── advanced-viewcode.vue
│   │   ├── basic-ui.vue
│   │   ├── basic-viewcode.vue
│   │   ├── intro.vue
│   │   ├── login.vue
│   │   ├── reg.vue
│   │   └── userproject.vue
│   ├── index.js
│   ├── store
│   │   └── index.js
│   ├── uimanage.js
│   └── view
│       ├── adminManage.vue
│       ├── intro.vue
│       ├── login.vue
│       ├── reg.vue
│       ├── uiManage.vue
│       └── userProject.vue
├── static
│   ├── 315c43a156661946d0490a53b49b2c32.png
│   ├── admin.App.js
│   ├── assets
│   │   ├── codemirror
│   │   ├── css
│   │   │   ├── animated.css
│   │   │   ├── date-tip.css
│   │   │   ├── font-awesome.min.css
│   │   │   ├── hover.css
│   │   │   ├── reset.css
│   │   │   └── uiManage.css
│   │   ├── fonts
│   │   │   ├── FontAwesome.otf
│   │   │   ├── fontawesome-webfont.eot
│   │   │   ├── fontawesome-webfont.svg
│   │   │   ├── fontawesome-webfont.ttf
│   │   │   ├── fontawesome-webfont.woff
│   │   │   └── fontawesome-webfont.woff2
│   │   ├── img
│   │   │   ├── cd-arrow.svg
│   │   │   ├── cd-avatar.png
│   │   │   ├── cd-logo.svg
│   │   │   ├── cd-nav-icons.svg
│   │   │   ├── cd-search.svg
│   │   │   └── logo.png
│   │   └── js
│   │       └── trianglify.min.js
│   ├── bundle.js
│   ├── codemirror
│   │   ├── bin
│   │   ├── bower.json
│   │   ├── emmet.js
│   │   ├── html-hint.js
│   │   ├── keymap
│   │   ├── lib
│   │   ├── mode
│   │   ├── package.json
│   │   ├── show-hint.js
│   │   ├── theme
│   ├── index.App.js
│   ├── jquery.min.js
│   └── uimanage.App.js
├── views
│   ├── admin.html
│   ├── error.html
│   ├── index.html
│   ├── login.html
│   └── uimanage.html
├── webpack-production.config.js
└── webpack.config.js

```

从目录结构中可以看到重复了assets文件夹和codemirror，这是因为开发过程中为了引用方便导致没考虑直接重复了。将其所有静态文件还是放在express目录下的`static`目录统一管理。

## 正文
为了不遗漏每个细节部分，我从view目录下的每个.vue进行重构，一开始我有考虑用fetch替换，但是整个项目其实有引入完整的jquery，那么引入fetch的意义就不是很大。不过我们需要调整一下以es6语法来写:

```
 getProjectList (){
      var that = this
      console.log(that.current_userid)
      $.get('/getProjects/'+that.current_userid,function(data)                {
        that.projects = data.Projects
      });
    },
```

比如上面这段一般情况下我都会用var that = this 来保存this指向。不过有了箭头函数对于$get我们可以这么改成es6形式:

```
    getProjectList (){
      
      console.log(this.current_userid)
      $.get('/getProjects/'+ this.current_userid, (results) => {
        this.projects = results.Projects;      
      });
    },
```

同理，一段$post也可以改成这样的形式：

```
      $.post("/addProject",{projectname:this.newProjectName,
                           desc:this.newProjectDesc,
                           user_id:this.current_userid
                          },
        (result) => {
          if(result.status === "ok"){
            alert("添加成功!");
            $('#normalModal').modal('hide');
            this.getProjectList()
          }
          else{
            alert("添加失败!"+result.message);
          }
        });
```

依次循环把home.vue修改完成。然后对所有进行调整，发现其实类似函数并不是很多。接下来就是把资源给整合。

一开始我是把一些css文件给独立出来放在`static\assets\css`目录下，然后这么引用:

```
<style>
@import url("../../static/assets/css/date-tip.css");
}
```

但是发现样式会产生变化，尤其一些应该覆盖的效果没有出现。因此我又改成

```
<style scoped src="../../static/assets/css/codemirror.css"></style>

```

`scoped`关键词一定要加~这才能不将样式搞乱。

余下的部分就是权限管理这块，这块我写的比较杂，
比如在express的数据获取这块我写了判断权限:

```
/**

 * [checkAdmin func]
 * 判断用户是否为管理员
 * 接受传来的参数
 * 返回 true 或者 false
 */
exports.checkAdmin = function ( username ){

  /* 判断用户是否存在 */
  User.findOne({username:user.username},function(err,result){
     if(result != null){
        if (result.limit == "admin"){
          return true
        }else{
          return false
        }
     }
  })
};

```

然后在页面获取前获得该用户的ID和name

```

 $.get('/getCurrentUser',  (result) => {
      if(result.status == "ok"){
        this.current_user = result.message
        window.localStorage.current_user = result.message
      }else{
        window.location.href="/login";
      }
    });
    $.get('/getCurrentUserId',  (result) => {
      if(result.status == "ok"){
        this.current_userid = result.message
        this.getProjectList()
      }
    });
    
```

因为路由跳转判断权限不太适合我这个项目因此没有修改。

## 结尾

最后我将项目整理并上传至Github，感兴趣的可以去看看

[UImanage-System](https://github.com/linshuizhaoying/UImanage-System)

并希望能点个star~







