---
layout: post
title: 打造基于websocket的iframe插件
category: 技术
tags: [总结,开发,前端,移动端]
keywords: 总结,开发,前端,移动端]
description: 
---

## 前言
这是根据一个需求做的开发总结，适合场景就是类似放在门户网站右边的小按钮，点击弹框载入一个窗口，以及业务需求上接入某个产品页面，然后需要和这个产品做一些数据通信。因此就有了这个插件。

开发雏形难度不大，更多的是细节上一些考虑。

## 正文

设计上先按照雏形开发，功能完善这个想法进行。

首先肯定是搭建基础骨架，而且现在都ES6开发了，不能用以往ES5那种封装方式来开发。
更主要是配合typescript让开发体验更好一些。

因此一开始

```

demo/
    index.html
src/
    index.ts
tsconfig.json

```

这样的结构就可行，`index.html` 加载的是`tsc`命令执行后生产的`build目录下dist目录里的文件`。

具体代码如下:

```index.html

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <script src="../build/dist/index.js"></script>
    <title>Document</title>
  </head>
  <body>
  </body>
  <script>
    const test = new TodoPlugin();
    test.init()
  </script>
</html>

```

之后就是骨架代码index.ts的实现



```

class TodoPlugin {
  todoPluginId = "";
  todoUrl = "";
  todoBackEndUrl = "";
  constructor() {
  }
  isExist(id: string) {
    return document.getElementById(id);
  }
  remove(id: string) {
    const child = document.getElementById(id);
    child && child.parentNode && child.parentNode.removeChild(child);
  }
  async create(id: string) {
    // 创建容器
    const container = await document.createElement("div");
    container.setAttribute("id", id);
    // 容器设置样式
    container.style.height = "100%";
    container.style.position = "fixed";
    container.style.top = "0";
    container.style.right = "0";
    container.style.left = "0";
    container.style.bottom = "0";
    container.style.zIndex = "1000";
    container.style.backgroundColor = "#595959";
    container.style.opacity = "0.9";
    container.style.display = "none";
    // 将容器设置到tab最前面,让esc事件能生效
    container.tabIndex = 0;
    //容器绑定点击事件
    container.addEventListener(
      "click",
      () => {
        console.log("click");
        this.hide();
      },
      false
    );
    //绑定键盘esc事件
    container.onkeydown = e => {
      console.log("e", e);
      if (e.keyCode == 27) {
        this.hide();
      }
    };
    // 创建iframe
    const iframe = await document.createElement("iframe");
    iframe.src = this.todoUrl;
    iframe.setAttribute("id", id + "Iframe");
    // iframe 设置样式
    iframe.style.height = "80%";
    iframe.style.width = "80%";
    iframe.style.margin = "10%";
    // 去掉边框
    iframe.scrolling = "no";
    iframe.frameBorder = "no";
    // 载入到页面
    await container.appendChild(iframe);
    await document.body.appendChild(container);
  }
  init(appId: string, username?: string) {
    // 判断是否已有iframe
    if (this.isExist(this.todoPluginId)) {
      console.log("节点存在");
      // 删除节点
      this.remove(this.todoPluginId);
      // 重新初始化
      this.init(appId, username);
    } else {
      console.log("节点不存在");
      // 创建节点
      this.create(this.todoPluginId);
    }
  }
  // 展示iframe
  show() {
    const container = document.getElementById(this.todoPluginId);
    container ? (container.style.display = "block") : "";
    // 防止通过按钮执行show函数，导致焦点在button上使esc事件失效，因此手动聚焦。
    container ? container.focus() : "";
  }
  // 隐藏iframe
  hide() {
    const container = document.getElementById(this.todoPluginId);
    container ? (container.style.display = "none") : "";
  }
  // 销毁iframe
  destroy() {
    const iframe = document.getElementById(this.todoPluginId);
    iframe ? document.body.removeChild(iframe) : null;
  }
  // 重新初始化iframe
  refresh() {
    // @ts-ignore
    const iframe = document.getElementById(this.todoPluginId + "Iframe");
    // @ts-ignore
    iframe.src = iframe.src;
  }
}

```

可能一开始看到代码会有点迷，怎么骨架有怎么多方法，原因是我们是基于iframe来加载目标应用，那么对iframe涉及到的操作我们都需要考虑。用原生写法是为了兼容和可扩展性考量。

这里也有一些设计上的考量，那就是对已有的重复节点需要做处理。具体流程图如下：

![wechatimg163.png](http://img.haoqiao.me/blogwechatimg163.png)

这个比较简单。重点讲一下按键绑定的过程。

这是基于很常见的交互需求，点击iframe外部或者右上角叉叉可以隐藏iframe,直接按esc,也可以隐藏。点击关闭很好解决，隐藏需要注意的是要将`插件父节点的tabIndex`设置为0，这样浏览器才会帮你把默认的焦点放在父节点上。

创建完骨架后，已经可以载入目标应用了，此时就提到了上述的需求，我们需要和目标应用做通讯操作，而iframe中最方便也就是websocket了。

我们需要再目标应用中加入一个监听器来监听来自插件的询问，以及将各个功能模块抽象，以便于插件提出的功能实现。

而插件也需要监听来自目标程序的回应，以及一些秘钥的交换。

首先我们在目标程序初始化的生命周期中中加入如下代码:

```

      // 启动 PostMessage 监听信息
      window.addEventListener(
        'message',
        e => {
          if (window.location.origin !== e.origin) {
            console.log('监听到消息:', e, e.data);
            // @ts-ignore
            this.postObject = e;
            console.log('this.postObject', e);
            if(e.data.type === 'sendTokenRequest') {
              // @ts-ignore
              e.source.postMessage({type: 'token', token: window.SERVER_DATA && window.SERVER_DATA.accessToken} , e.origin);
            }
            if(e.data.type === 'selectTodo') {
              this.selectTodo(e.data.id);
            }
          }
        },
        false
      );

```

这里简单加了两个指令，用户插件通知目标程序做些什么。

首先是秘钥的交换。`插件一开始是什么都没有的，因此初始化的时候它会发个请求让目标应用去将秘钥发回。`。

而插件的监听代码如下：


```

window.addEventListener(
  "message",
  e => {
    if (window.location.origin !== e.origin) {
      console.log("插件获取返回数据:", e, e.data);
      switch(e.data.type){
        case 'token': 
          window["todoPluginToken"] = e.data.token;
          break;
      }
    }
  },
  false
);

```

到这里，简单的通讯就建立了。

因为插件多个地方需要用到和目标应用通讯，因此需要把这个函数抽象出来

```

  sendMessage(id, data) {
    // 发送消息
    // @ts-ignore
    const iframeWindow = document.getElementById(id + "Iframe").contentWindow;
    iframeWindow.postMessage(data, "*");
  }

```

至此，简单的小插件就完成了。


