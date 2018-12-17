---
layout: post
title: React从入门到上手三部曲
category: 技术
tags: [react，react学习,react实例,react入门,react总结]
keywords: 学习资料，程序语言
description: 
---

## 正文
最近学习react，总结一些东西，仅仅只是将一些个人认为比较重要的概念配合例子来讲解。有些例子来自阮一峰的学习笔记。其余的来自官网教程和<<React-引领未来的用户开发框架>>这本书。

## 概念
  React是UI组件开发思路。配合jsx将组件不在写在html页面而是script中。它可配合Jquery，Backbone等等第三方库来使用,仅仅是将我们平常写的组件开发思路变了一下。曾经我想写一个组件，第一个思路就是写在jq的插件中。但是这种第三方插件的写法效率低下，虽然功能可能都可以实现，但是复杂，尤其是嵌套组件时，那将是个噩梦。React的出现给组件开发带来了革命性的创新。

### Jsx
这种东西的基础学习请参考[官方教程](http://facebook.github.io/react/docs/i).
补充点实用的：

    jsx中class的写法不再是
    <div class="xxx"> </div>
    而是
    <div className="xxx"> </div>
    
    Style的写法:
    
    var divStyle = { 
    color: 'white',   
    backgroundImage: 'url(' + imgUrl + ')',  
    WebkitTransition: 'all', // note the capital 'W' here   msTransition: 'all' // 'ms' is the only lowercase vendor prefix };
    React.render(<div style={divStyle}>Hello World!</div>, mountNode);

#### Jsx的条件判断
  
1. 三目  
2. 设置一个变量在属性中引用它
3. 将逻辑转化到函数中
4. 使用&&运算符


三目运算符例子：

    this.state.icComplete?"is-complete":'none'
     
使用变量例子

    var isComplete = this.getIsComplete();
    return <div className={isComplete}>...</div>
    
使用函数
    
    getIsComplete:function(){
    	if(...) 
    	return true
    }

    render:function(){
      return <div className={this.getIsComplete}>...</div>;
    }
    
使用&&运算(如果前面为真就使用后面的字符串)

	return <div className={this.state.isComplete && "is-complete"}



#### 引用(ref)
    return <div ref="myInput">
在组件的任何地方使用 this.refs.myInput获取这个引用，并且通过这个引用，使用 this.refs.myInput.getDOMNode()获得真实的DOM节点

#### 事件
  在jsx中，事件名被规范化并统一采用驼峰形式表示。change -> onChang , click -> onClick 
  捕获事件和设置属性一样
     
     return <div onClick={this.handleClick}>...</div>
     
   React自动绑定了组件所有方法的作用域，因此不需要手动绑定。
   
#### 特殊属性
  label的标签添加for属性需要用htmlfor
  
### 小结
  其实react是有不用jsx的写法，但是我个人认为，既然jsx伴随react，而且官方文档里面也采用jsx的写法，如果真的自己想去手写纯react，那就是舍弃轮子的做法，除非自己有更好的轮子，否则还是学习jsx的写法。
  
### 组件的生命周期
   组件的生命周期分成三个状态：
   
    Mounting：已插入真实 DOM
    Updating：正在被重新渲染
    Unmounting：已移出真实 DOM
React 为每个状态都提供了两种处理函数，will 函数在进入状态之前调用，did 函数在进入状态之后调用，三种状态共计五种处理函数。

    componentWillMount()
    componentDidMount()
    componentWillUpdate(object nextProps, object nextState)
    componentDidUpdate(object prevProps, object prevState)
    componentWillUnmount()
此外，React 还提供两种特殊状态的处理函数。

    componentWillReceiveProps(object nextProps)：已加载组件收到新的参数时调用
    shouldComponentUpdate(object nextProps, object nextState)：组件判断是否重新渲染时调用
更多组件生命周期可以点这里[React中文文档](http://reactjs.cn/react/docs/component-specs.html),如果英文水平还可以，而且想跟踪最新文档可以点这里[React Docs](http://facebook.github.io/react/docs/component-specs.html) 不过现阶段生命周期应该不会再有所改变。

### 数据流

#### props
props就是properties的简称，根据google翻译来看，它有性能，特性，属性的含义，可以使用它将任意类型的数据传给组件。一般情况下是在加载状态(挂载组件时)来设置它。比如

	var survey = [{title:"xxxx"}];
	<ListSurveys survey={survey}/>
	
或者调用组件实例的setProps方法（但大多时候不会采取这种方式，因为props一旦设置就不会轻易去改变它。）
    
    var survey = [{title:"xxxx"}];
    var listSurveys = React.render(
       <ListSurveys/>,
       document.querySelector('body');
    );
    listSurveys.setProps({surveys:surveys});
可以通过this.props来访问props 但绝对不能通过这种形式改变它，一个组件绝对不可以自己修改自己的props。（于此对应的可修改自身的是state,详情看下文）
 
可以为组件添加getDefaultProps函数来设置属性的默认值。不过，这应该只针对那些非必须属性。

    var SurveyTable = React.createClass({
    	getDefaultProps : function(){
    	  return {
    	    surveys:[]
    	  };
    	}
    	//...
    
    })
    
getDefaultProps并不是在组件实例化时被调用，而是在createClass时被调用，返回值会被缓存。也就是说，不能在getDefaultProps中使用任何特定的实例数据。

#### state
每一个组件都能拥有自己的state,state和props的区别在于state只存在于组件的内部。还有一个简单的区分方法是this.props 表示那些一旦定义，就不再改变的特性，而 this.state 是会随着用户互动而产生变化的特性。
state可以通过setstate来修改，也可以通过getInitialState方法提供一组默认值，写法和props类似。

千万不能直接修改this.state,永远记得用this.setstate方法修改。

不要在state里面保存计算的值，应该只保存最简单的数据，比如勾选状态checked。

不要尝试把props复制到state中，把props当中数据源。

	State 工作原理
	常用的通知 React 数据变化的方法是调用 setState(data, callback)。这个方法会合并（merge） data 到 	this.state，并重新渲染组件。渲染完成后，调用可选的 callback 回调。大部分情况下不需要提供 callback，因为 	React 会负责把界面更新到最新状态。

更多state的原理可以点这里[More State](http://reactjs.cn/react/docs/interactivity-and-dynamic-uis.html)


### 顶层API（必看）
React 是 React 库的入口。如果使用的是预编译包，则 React 是全局的；如果使用 CommonJS 模块系统，则可以用 require() 函数引入 React。

直接看[官网的api](http://facebook.github.io/react/docs/top-level-api.html)
  
## 其他
     在事件回调中调用preventDefault()来避免浏览器默认地提交表单。
     
     
react实现了autoFocus属性，在组件第一次加载时，没有其它表单域聚焦时，react会自动对焦。

    example:
     //jsx
     <input type=“text” autoFocus =“true” />
     
     

## 总结
第一曲大概就是了解一下React，下面两部曲将直接Show you the code.
     







