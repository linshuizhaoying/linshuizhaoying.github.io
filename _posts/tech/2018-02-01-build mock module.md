---
layout: post
title: 从零开始打造 Mock 平台 - 功能模块篇
category: 技术
tags: [学习,开发,前端,Node,React,React,Mock,功能模块,构造]
keywords: 学习,开发,前端,RxJS,React,React,Mock,功能模块,构造
description: 
---

# 前言

二月初想想这个月还得捣鼓一篇文章，也没啥好的想法那还是记录一下毕设的一些思路吧。

# 重要功能

一些扩展的重要功能将在这里一点点从零开始进行思考。

## 项目导入导出功能

项目导入导出的构想是在设定特色功能时候想到的，主要是用于不同服务器上如果部署了平台，如果想要自己私下部署测试，那么重新建立项目，然后再一个个配置接口路径，配置返回数据是一件很麻烦的事情。为了以后用(自)户(己)用的顺畅，想了一键各种版本的功能，其中就包括项目的导入导出。

因此在设计数据结构的时候，我将最终返回结果数据设计成如下形式

```

{
  project:{
  	 projectId:,
  	 ...
    interfaceList: []
  }
}

```

也就是前端本地缓存的数据就可以直接导出。

当然导出可以把一些信息先过滤处理一遍，比如项目Id ,接口 Id。

于是导出可以为一个纯 Json 文本。

然后倒入时候需要验证 Json 格式，其实就是需要做个遍历，看看该有的属性有没有，没有的话就报错不进行数据导入。

而且导入的时候需要做个分类，是导入到项目示例还是个人/团队项目。

常见的是打包下载 zip。这个大概需要后端处理，因此先找找有没有前端处理的。

搜了一下方案，再结合 github 上一些项目的源码，可以简单的写出一个导出模块

```

export function exportFile(data: string, filename: string, type: string) {
    var typeList = {
      json: 'application/json;charset=utf-8',
      markdown: 'text/markdown;charset=utf-8',
      doc: 'application/mswordcharset=utf-8',
    }
    // 创建隐藏的可下载链接 
    var eleLink = document.createElement('a');
    eleLink.download = filename;
    eleLink.style.display = 'none'; // 字符内容转变成blob地址 
    var blob ;
    blob= new Blob(['\uFEFF' + data],{type: typeList[type]});

    eleLink.href = URL.createObjectURL(blob); 
    document.body.appendChild(eleLink);
    eleLink.click();
    document.body.removeChild(eleLink);
}

```

调用方式很简单，就是传入三个参数:

`exportFile(JSON.stringify(this.state.currentProjectData), 'default.md', 'markdwon')`

![imgn](http://img.haoqiao.me/1241517547766_.pic_hd.jpg)

当然我们现在获得的数据是没有进行处理的，我们需要对数据做个过滤，剔除一些关键信息以及没必要的数据。

假设现在的数据是这样的

```

{
  "_id": "project001",
  "projectName": "演示项目 - REST接口示例超长字符串测试asd123",
  "projectUrl": "/project001",
  "projectDesc": "项目描述",
  "version": "v1.0",
  "transferUrl": "http://haoqiao.me/api/project",
  "status": "transfer",
  "type": "demo",
  "teamMember": [
    {
      "_id": "user001",
      "username": "2333",
      "role": "前端工程师",
      "avatar": "https://zos.alipayobjects.com/rmsportal/ODTLcjxAfvqbxHnVXCYX.png"
    },
    {
      "_id": "user002",
      "username": "宋青树",
      "role": "后端工程师",
      "avatar": "https://zos.alipayobjects.com/rmsportal/ODTLcjxAfvqbxHnVXCYX.png"
    }
  ],
  "interfaceList": [
    {
      "_id": "interface001",
      "interfaceName": "获取",
      "url": "/getAll",
      "method": "get",
      "desc": "接口描述",
      "mode": "{data: 1 || 2}"
    },
    {
      "_id": "interface002",
      "interfaceName": "增加",
      "url": "/add",
      "method": "post",
      "desc": "接口描述",
      "mode": "{data: 1 || 2}"
    },
    {
      "_id": "interface003",
      "interfaceName": "删除",
      "url": "/delete",
      "method": "delete",
      "desc": "接口描述",
      "mode": "{data: 1 || 2}"
    },
    {
      "_id": "interface004",
      "interfaceName": "更新",
      "url": "/update",
      "method": "put",
      "desc": "接口描述",
      "mode": "{data: 1 || 2}"
    }
  ]
}

```

我们需要将里面所有的 `_id(字段)`, `teamMember(数组)` 去除。这里大家想想如果是自己要怎么处理？

其实非常简单，只要你对原生的 `JSON.stringify` 比较熟悉，你就知道它的完整定义如下

> JSON.stringify(value, replacer?, space?)

`replacer` 是一个过滤函数或则一个数组包含要被 `stringify` 的属性名。如果没有定义，默认所有属性都被 `stringify`。

可以做一个遍历器，遍历Json里的属性名。然后内部做个剔除。

比如这样

```

filterData = (json: any) =>{
    console.log(json)
    let expectArr = ['_id', 'teamMember']
    let filterArr = []
    let result = ''
    for( let key in json){
      if (expectArr.indexOf(key) === -1){
      filterArr.push(key)
    }
    result = JSON.stringify(this.state.currentProjectData, filterArr)
    console.log(result)
    
    return result

  }

```

但这样只能拿到第一层的属性名，如何拿到嵌套的数组里的Json的属性呢？

我们只需要再做个判断就可以了


```

filterData = (json: any) =>{
    console.log(json)
    let expectArr = ['_id', 'teamMember']
    let filterArr = []
    let result = ''
    for( let key in json){
      if (expectArr.indexOf(key) === -1){
        filterArr.push(key)
        // 如果是嵌套数组，而且数组内有数据
        if(Object.prototype.toString.call(json[key]) == "[object Array]" && json[key].length > 0){
          for( let item in json[key][0]){
            // 同样对里面的json数据进行属性字段过滤
            if (expectArr.indexOf(item) === -1){
              filterArr.push(item)
            }
          }
        }
      }
    }
    result = JSON.stringify(this.state.currentProjectData, filterArr)
    console.log(result)
    
    return result

  }

```

这样就把该有的属性筛选出来了。然后做个转换就能过滤只剩需要的数据。

数据清理后就变成如下格式

```

{
  "_id": "project001",
  "projectName": "演示项目 - REST接口示例超长字符串测试asd123",
  "projectUrl": "/project001",
  "projectDesc": "项目描述",
  "version": "v1.0",
  "transferUrl": "http://haoqiao.me/api/project",
  "status": "transfer",
  "type": "demo",
  "teamMember": [
    {
      "_id": "user001",
      "username": "2333",
      "role": "前端工程师",
      "avatar": "https://zos.alipayobjects.com/rmsportal/ODTLcjxAfvqbxHnVXCYX.png"
    },
    {
      "_id": "user002",
      "username": "宋青树",
      "role": "后端工程师",
      "avatar": "https://zos.alipayobjects.com/rmsportal/ODTLcjxAfvqbxHnVXCYX.png"
    }
  ],
  "interfaceList": [
    {
      "_id": "interface001",
      "interfaceName": "获取",
      "url": "/getAll",
      "method": "get",
      "desc": "接口描述",
      "mode": "{data: 1 || 2}"
    },
    {
      "_id": "interface002",
      "interfaceName": "增加",
      "url": "/add",
      "method": "post",
      "desc": "接口描述",
      "mode": "{data: 1 || 2}"
    },
    {
      "_id": "interface003",
      "interfaceName": "删除",
      "url": "/delete",
      "method": "delete",
      "desc": "接口描述",
      "mode": "{data: 1 || 2}"
    },
    {
      "_id": "interface004",
      "interfaceName": "更新",
      "url": "/update",
      "method": "put",
      "desc": "接口描述",
      "mode": "{data: 1 || 2}"
    }
  ]
}

```

之后是要考虑导入项目，我首先想到点击上传 JSON 格式文件然后读取里面的内容，然后验证数据再让后台将其导入到指定表中。

这里面发生了一些事情，比如我肯定不希望用户真的把json文件上传到服务器上，我觉得这玩意前端肯定能解析和解决，但是我们肯定还是需要一个上传的按钮和UI交互。

这里我直接用了 `antd` 的 `upload` 组件，它里面有个方法就是 `beforeUpload` , 只要我们在这个函数里直接返回 `false` 那么是不会真正触发上传动作的，但是我们又可以拿到本地的 `File`，可以用 `HTML5` 的新方法 `FileReader` 来帮助读取内容。

我们的组件可以这么修改

```

const uploadProps = {
      name: 'file',
      action: '',
      showUploadList: false,
      beforeUpload: (file: any) => {
        const isJSON = file.type === 'application/json';
        if (!isJSON) {
          Message.error('只允许上传JSON格式文件!');
        }
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isLt2M) {
          Message.error('JSON文件大小必须小于 2MB!');
        }
        var reader = new FileReader(); // 读取操作都是由FileReader完成的
        var that = this
        reader.readAsText(file);
        reader.onload = function(){//读取完毕从中取值
          const json = this.result
          if(isJson(json) && that.state.uploadJsonData.length === 0){
            that.setState({
              uploadProject:true,
              uploadJsonData: json
            })
            Message.success('Json文件上传识别成功!');
          }
        }
        return false;
      }},
      
      onChange: (info: any) => {
      },
    };
    
   <Dragger {...uploadProps}>
      <p className="ant-upload-drag-icon">
      <Icon type="inbox" />
      </p>
      <p className="ant-upload-text">点击上传JSON文件或者拖拽上传JSON文件</p>
              
  </Dragger>

```

来看下实际的交互效果

![imgn](http://img.haoqiao.me/2018-uploadJson.gif)

这样我们就打通了项目的导入导出功能的交互。之后就是接口对接一下就行了。



## 项目克隆 / 接口克隆

这两个功能其实很类似，主要是用于帮助用户能够复制已经存在的接口或者项目。
比如我已经之前建立了一套系统的接口，包括了增删减改。我下一个系统和这套系统很类似，可能只需改几个字段就可以用了。
我们当然可以利用导入和导入功能，但是在系统内部我们最好有一键迁移的方式，那就是克隆。

克隆我们需要注意，首先是接口克隆，假设我们接口定义的格式如下:

```

_id(pin): "interface005"
interfaceName(pin): "注册"
url(pin): "/reg"
method(pin): "post"
desc(pin): "接口描述"
mode(pin): "{data: 1 || 2}"

```

然后我为了方便定义的项目 Model 里面包含了接口 Model。

也就是我只需把 接口 Id,和 项目 Id 传给后台，让后台做一个查询接口内容，然后新建接口把查询到的内容插入到指定 Id 就可以了。

这很简单。主要部分是 UI 这块，不过通过对数据流的管理也是花时间就能解决的事情。如下图:

![imgn](http://img.haoqiao.me/build%20mock%20module%201.gif)

之后是克隆项目这块,
我们首先已经知道项目 Model 里面包含了 接口 Model,因此我们克隆整个项目其实是需要将整个接口提取出来，团队成员是需要剔除的，因为新克隆项目应该是只有创建者，因此我们需要把 用户 Id 也从前端传过去，当然也可不传，通过 Jwt 对 token 解析也能识别用户信息。

主要是后端拿到信息之后它的思路应该是先查询这个项目的信息，然后提取部分信息创建新项目, 然后遍历原有项目的接口列表，批量创建接口。

## 个人信息的更改

基本上中后台的应用都会有个人信息管理这项，有的用表单，有的拆分。

其余数据都好搞定，无非是传参的问题，前后端约定的问题。
当然比较麻烦的其实是头像的更改。
假设你注册的时候默认分配给一个用户头像，然后再个人信息里用户想要更改。
这时候问题来了。更改头像其实是一个交互问题，你肯定不能让用户一步步操作。而是一步到位，符合要求的图片上传之后，拿到上传后的图片地址。然后更改本地的数据。还需要在后台自动更新数据。

![imgn](http://img.haoqiao.me/build%20mock%20module%202.gif)

这里是用 redux 维护了一个本地的前端数据层，所有显示的变更显示操作需要对其进行更新。

然后是 reducer 里监听了动作, 定义为 `UPDATE_LOCALXXX` 的动作是用于提交数据后等待后台处理完毕后返回成功，然后将本地的数据进行更新。

显而易见这个动作是异步操作。如果很多个类似操作需要管理我们代码写起来肯定会很乱。因此我在技术评估阶段引入了 `rxjs`。

通过其特点时间线的管理，就很容易了。以下是简单的示例代码

```

export const userUpdate = (action$: any) =>
  action$.ofType(UPDATE_USER)
    .mergeMap((action: any) => {
      return fetch.post(updateUser, action.data)
        // 登录验证情况
        .map((response: any) => {
          console.log(response);
          if (response.state.code === 1) {
            updateUserSuccess(response.state.msg);
            return updateLocalUser(action.data);
          } else {
            console.log('token error')
            updateUserError(response.state.msg);
            return nothing();
          }
        })
        // 只有服务器崩溃才捕捉错误
        .catch((e: any): any => {
          // console.log(e)
          return Observable.of(({ type: USER_LOGINERROR })).startWith(loadingError())
        })

    });


```



# 结尾

还有一些功能需要等后端开发的时候再记录思路，因此这部分先到这里。

