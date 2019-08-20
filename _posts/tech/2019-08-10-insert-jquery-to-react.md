---
layout: post
title: 如何在react项目引用jQuery
category: 技术
tags: [总结,开发,前端,开发,React,Jquery,Jquery插件,jquery与react,模块]
keywords: 总结,开发,前端,开发,React,Jquery,Jquery插件,jquery与react,模块]
description: 
---

## 前言

做兼容是比较难受的事情，更主要的是有些兼容一开始都是无从下手。

而在现代前端技术栈中去引入一个jQuery的商业插件，来兼容之前的项目，是更加需要精力的事情。

背景就是现在有一个比较复杂的基于antd二次封装的的table组件，它可以接收一堆参数，并自己生产数据，绑定事件，管理端只需要配配配即可。

但它在越来越复杂的使用场景中，有些比较极端的情况比较难处理，就是大数据的展示与操作，几百条数据，几十个列需要同时展示时，会卡顿严重影响体验。这种极端场景在测了不少现有的开源组件后发现无法满足，而jQuery版的一款商业插件可以在这个场景中对其优化，因此就考虑了在项目中引入这jQuery插件来优化交互体验。

## 正文

在一开始时，走了不少弯路，最重要的就是项目要如何引入，jQuery的插件肯定一开始默认就带了jQuery，这款版本还比较低，除此之外它并没封装打包的类似`.min.css`的文件，取而代之的是一整个样式文件夹，提供了换肤功能。

之前一开始打包的时候是直接想着 `npm i jquery` 然后 `import`进来，然后再把对应的jQuery插件当做模块也同样`import` 进来，但是这会有个问题，造成`webpack`打包后的文件超大。

因此，在现在这种已经普遍利用`webpack`等打包工具的情况下，想要引入`jquery`文件最好的方式是通过自建`cdn`或者网上现成的`cdn`的服务。

同理`jQuery插件`也通过cdn的方式引入。这样可以避免很多问题，尤其是原本项目就比较大的时候，打包大型js文件进去会出现溢出的情况。

![f54577e2611fc70ac069c6d6aae70d4d.jpg](http://img.haoqiao.me/blogf54577e2611fc70ac069c6d6aae70d4d.jpg)

### demo

在正式想要完全接入某个插件，然后对旧组件的参数进行全部适配前，首要必须得先建立一个`demo`，验证核心的功能能正常稳定运行。

而在`demo`开发期间是没有直接接触真实或者现成的api接口数据。需要自己伪造。尤其是大数据的`columns`和填充数据，这个单独写个方法即可.

```javascript

export const randomFullColumns = num => {
  const result = [];
  for (let i = 0; i < num; i++) {
    //  { field: 'age', width: 100, headerAlign: 'center', header: '年龄'},
    result.push({
      field: Math.random().toString(36).substring(2),
      width: 50,
      headerAlign: 'center',
      header: '随机' + Math.random()
    });
  }
  return result;
};
// 随机数据生成
export const randomDataGenerate = (columns, num) => {
  console.log('now colums', columns);
  const result = [];
  for (let i = 0; i < num; i++) {
    const temp: any = {};
    columns.forEach(item => {
      if (item === '110') {
        temp[item] = '123';
      } else {
        temp[item] = Math.random().toString(36).substring(2);
      }
    });
    result.push(temp);
  }
  return result;
};


```

然后适配一些基础的参数后就可以先把demo跑起来。

## 着手改造

在此之后整个改造的思路设计如下：

```

1、在demo基础上先搭建组件的结构，规划好api,until,store
2、因为要兼容的内容属于比较复杂的而且难维护的表格代码，因此先考虑把colmuns和data数据通过接口拿到并通过一系列旧函数处理后最终的样子。
3、将旧函数处理后的colmuns通过新方法转为插件能够接收的参数格式。
4、将旧函数处理后的data通过新方法转为jQuery插件能够接收的参数格式。
5、新data通过数据字典转换对应字段参数

```

核心目标是需要把一个`jQuery插件`给转变成适配的`react组件`,也就是要将生命周期控制清楚，数据源，以及要考虑未来多实例化的场景。

第三部兼容的操作就是把原来的colmuns的格式列出来，然后想要兼容的格式列出来，写个函数将其转换。但涉及到某些单元格合并的操作，我们需要考虑递归实现。

```

// 将旧的colmuns适配为支持的格式
/*
将 oldType = {
    align: "right"
    dataIndex: "account"
    fixed: false
    sorter: true
    title: "账号"
    width: 120
}
转为
newType: {
  headerAlign: "right",
  field: "account",
  header: "账号",
  width: 120,
  allowSort: true
}

复杂的嵌套
{
children: [oldType, oldType]
title: "test"
width: 404
}

转为
{ header: "test", headerAlign: "center", width: 404,  columns: [newType, newType]

因此需要做个递归操作

export const recursiveColmun = column => {
  let result: any = {};
  if(column.children && column.children.length) {
    result = {
      columns: [],
      header: column.title,
      headerAlign: 'center',
      width: column.width
    };
    result.columns = [];
    column.children.forEach(item => result.columns.push(recursiveColmun(item)));
  } else {
    result = {
      headerAlign: column.align,
      field: column.dataIndex,
      header: column.title,
      width: column.width,
      allowSort: column.sorter
    };
  }
  return result;
};
export const transformColmuns = columns => {
  const result = [];
  columns.forEach(column => {
    result.push(recursiveColmun(column));
  });
  return result;
};


```

## jQuery与react之间的数据通讯

一些比较正经的数据，比如通过接口获取的表格数据，我们完全可以通过`react`配合`mobx`获取后再通过`jquery插件`的方法传递参数。

但是有些场景这种数据传递方式就不可行了。

>原来的组件有个功能就是可以通过管理端配置一些参数生成按钮，然后前端实现对应的onclick事件，这样可以实现对某行数据的操作。

但是要兼容到jQuery插件有一些的阻碍，最大的就是jQuery是没法直接插入antd的组件到dom里。

而且jQuery插件的某行动态渲染只能通过renderer参数来返回Html字符串实现。但这里又会有个问题就是如果是html字符串生成的按钮，想要绑定从前端初始化传进来的事件会有问题，`onclick=“function(){}”`，虽然可以通过`eval`来执行，但是还是缺失上下文，而且`eval`都是尽量避免不采取的技术手段。因此这里的做法就是在生成colmuns参数的时候，把对应传入的绑定事件约定名称 `xxx_btnHandle_name` 加入到`window`下，然后生成html的时候执行的是对应的函数即可。

但是这里又会有个问题，就是每行的`record`想要传递出去，直接通过call传会发现在html生成后是`[object,object]`。因此这里的处理手段也是同样的通过渲染时的下标作为索引，然后约定名称 `xxx_row_index`
传递出去。这样前端就可以以 `handle(record)`的形式拿到对应数据做处理。

这一步流程属于用户的自定义按钮的适配。


```

// 用户定义的渲染
const columnRender = e => {
  const index = e.rowIndex;
  const record = e.record ;
  // 因为不能直接把对象当做参数写入到html执行，因此把对象存到window,然后作为中转转发出去
  window['xxx_row_'+index] = record;
  // console.log('columnRender', e['column']['renderParams'], 'index', index , 'record', record);
  try {
    return e['column']['renderParams'].map(item => `<button onclick="window['xxx_btnHandle_${item.handler}'].call(null, ${index}, window['xxx_row_${index}'])">${item.label}</button>`).join('');
  } catch(error) {
    return null;
  }
};

```

而且要注意

```

Tip：

jQuery的参数需要额外注意和react的写法不一致
不能像react那样 
<CC
   xx={true}
/>

而是要

<CC
   xx={‘true’}
/>

或者

<CC
   xx=“true”
/>


```


## 多实例的解决方案

我们渲染某个`react组件`往往都会引入`mobx`的返回的`entity`，但是直接引用暴露出来的`store`实例会导致想要同时在一个页面引用两个相同组件，导致第二个组件的数据会覆盖第一个。

当然可以考虑直接只暴露非实例化的类，但是问题会出现在模块中不同文件中需要引用同一个例子的问题。

如果在入口index.tsx 

```
  /** 生成entity */
  private tableEntity: any = new entity();
  
```

这么写，多实例化时可以保持每个文件的数据独立性，但是如果切分功能模块的话，就无法让其他工具函数引用相同的entity。

因此可以采用的方法有两种
1、通过把class实例化后暴露到window中，让其他文件能够通过某个标识来引用。
2、在store里再做一层，接收入口传入的id，并且在入口文件引用其它工具函数时提前注入store。这样在同一个组件实例中，所有模块引用同一份store.

第一种方法比较粗暴，而且把那么大的数据量直接丢在window里不可取。
第二种方法抛开技术难度，是比较合适的。

最后按照第二种方法实现了多实例化的功能，需要注意的就是工具类引用时需要注意一些细节即可。

## 结尾

在整个兼容项目中，阻碍最大其实就是现有的jQuery转为react组件的资料几乎没有，全靠经验一点点摸索然后试错。其实在`demo`能正常跑起来后，基本上最大的问题就是两种框架数据交互的问题。这个也是后来通过黑科技window来解决。说明一些场景还是需要基础打的扎实，才能有比较多的黑科技方案来实现一些特殊交互。


