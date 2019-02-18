---
layout: post
title: 好巧在线前端调试工具
category: 技术
tags: [js,设计模式,jsfiddle,Realtime Playground,折腾,积累,Vue.js,Vue,Webpack]
keywords: 前端,资料,学习,javascript
description: 
---

# 前言

昨天学了下原理，今天直接先做一个能用的成品出来。主要功能如下:

```

1.支持代码高亮
2.支持html自动补全
3.支持代码行数显示
4.支持实时刷新
5.支持代码缩进（折叠）
6.支持sublime各种快捷键(比如ctrl+D)

```

先来个动图看下最终效果：

![imgn](http://img.haoqiao.me/active59.gif)

整个代码我都传到了[github](https://github.com/linshuizhaoying/haoqiao-Fiddle)

# 正文

以上所讲的功能我们只需要100来行代码实现,当然这是因为我们引用了非常实用的js库`CodeMirror`才能达到这个效果。`CodeMirror`许多大名鼎鼎的在线代码编辑器的基础库。而且它提供的功能非常多。不过它原生是不支持类似按下tab键自动补全html代码的,但是我们可以在github上找到专门为CodeMirror编写的emmet插件。

为了以后能够方便调用我将其做成了vue组件。调用时只需要传入类型就可以了。
比如` <editor type="html"></editor>`

看代码之前建议先去将[CodeMirror官网](http://codemirror.net/)的例子都过一遍.

我们来看editor.vue里面关键代码:

```

 //选中目标节点
	  var _html = document.getElementById(that.type);
    //设置默认内容
	  var _value = {
	  	html:function(){
	  		return "<div class='test'>LinshuiZhaoying</div>"
	  	},
	  	css:function(){
	  		return ".test{color:red}"
	  	},
	  	javascript:function(){
	  		return "var LinshuiZhaoying = document.querySelectorAll('.test');for (var i = 0; i < LinshuiZhaoying.length; i++) { LinshuiZhaoying[i].addEventListener('click', function(event) { alert('ok')});}"
	  	}
	  };
    //根据传来的参数进行赋值
	  _html.value = _value[that.type]();

    //配置文件
	  var _config = {
	   // mode: "text/html",
	    mode: "text/html",
	    lineNumbers: true,
	    lineWrapping: true,
	    extraKeys: {"Ctrl-Q": function(cm){ cm.foldCode(cm.getCursor()); }},
	    foldGutter: true,
	    gutters: ["CodeMirror-linenumbers", "CodeMirror-foldgutter"],
	    keyMap: "sublime",
	    profile: 'xhtml' /* define Emmet output profile */
	  };

	  var _modes = {
	  	html:function(){
	  		return "text/html"
	  	},
	  	css:function(){
	  		return "css"
	  	},
	  	javascript:function(){
	  		return "javascript"
	  	}
	  };
	  _config.mode = _modes[that.type]();

    //将其绑定到window下,方便后面调用
	  window["editor_"+that.type] = CodeMirror.fromTextArea(_html,_config);


	  emmetCodeMirror(window["editor_"+that.type], {
	      'Tab': 'emmet.expand_abbreviation_with_tab',
	      'Cmd-Alt-B': 'emmet.balance_outward'
	  });

    //设置界面风格
	  window["editor_"+that.type].setOption("theme", "mdn-like");

    //每当内容改变自动渲染
    window["editor_"+that.type].on('change', function () {
      that.render();
    });
    
```

渲染过程写在vue的methods里面，代码和昨天原理类似，只需要改动部分获取节点内容的代码即可。

# 结尾

这个只是能用版本，事实上在计划中它应该还有以下功能:

```

1.支持动态引入js库（本地列表集合）
2.支持sass,less
3.更好的界面
4.支持demo输出打包
5.支持本地demo管理

```
当然这些功能将在未来慢慢实现，先挖个坑。。。如果觉得还可以请在[github](https://github.com/linshuizhaoying/haoqiao-Fiddle)点个star。



