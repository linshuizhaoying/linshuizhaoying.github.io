---
layout: post
title: Jquery 实现原理深入学习(1)
category: 技术
tags: [Jquery，Jquery学习,项目小结,前端学习,前端总结，javascript]
keywords: 前端,资料,学习
description: 
--- 

## 前言
最近有点小浮躁，感觉自己略水，看了几本书觉得自己可能在基础到实践这块有点问题。因此打算将《jQuery技术内幕 深入解析jQuery架构设计与实现原理》里做的书摘感兴趣的部分进行学习。大概分这几块

```
1.总体结构 √

2.构建函数 √

3.each功能函数实现

4.map功能函数实现

5.sizzle初步学习

6.attr功能函数实现

7.toggleClass功能函数实现(好伤)

8.val功能函数实现

9.ajax异步请求以及扩展学习
```

书上是1.7的版本但是为了学习还是找到最新版本下载，学习工具一个sublime足矣。

## 开始学习总体结构
首先应该看下大概的思路，粗略瞄了一下，一万多行代码，深深感觉到内心的激动\(≧▽≦)/。

```
(function( global, factory ) {
...
// Expose jQuery and $ identifiers, even in
// AMD (#7102#comment:10, https://github.com/jquery/jquery/pull/557)
// and CommonJS for browser emulators (#13566)
if ( typeof noGlobal === strundefined ) {
	window.jQuery = window.$ = jQuery;
}
return jQuery;

}));
```

一开始自调用匿名函数，立刻执行并初始化所有模块。

书上提到这样的好处。

```
1、 创建了一个特殊的函数作用域，不会和其它库冲突。
2、 不会污染全局变量
```
代码中它最后还手动把Jquery添加到windows对象。所以一开始它会检查

```
	if ( !w.document ) {
	  throw new Error( "jQuery requires a window with a document" );
	}
```

自调用匿名函数和茴字一样有多种写法:

```
1.

(function(){
  //...
}) ();

2.

(function(){
  //...
}() );

3.

!function(){
  //...
}()
```

## 构造函数


```
	jQuery = function( selector, context ) {
		// The jQuery object is actually just the init constructor 'enhanced'
		// Need init if jQuery is called (just allow error to be thrown if not included)
		return new jQuery.fn.init( selector, context );
	},
```
jquery会判断传入的是选择器表达式还是html代码。默认从根元素document对象开始。不过也可以通过传入的context来限定查找范围。

书上说Jquery会自己调用find来查找，但是新版本我看不懂哪里调用了-0-但是找到了这段

```
//首先看看支不支持getElementById

if ( support.getById ) {
		Expr.find["ID"] = function( id, context ) {
			if ( typeof context.getElementById !== "undefined" && documentIsHTML ) {
				var m = context.getElementById( id );
				// Check parentNode to catch when Blackberry 4.6 returns
				// nodes that are no longer in the document #6963
				return m && m.parentNode ? [ m ] : [];
			}
		};
		Expr.filter["ID"] = function( id ) {
			var attrId = id.replace( runescape, funescape );
			return function( elem ) {
				return elem.getAttribute("id") === attrId;
			};
		};
	} else {
		// Support: IE6/7
		// getElementById is not reliable as a find shortcut
		delete Expr.find["ID"];

		Expr.filter["ID"] =  function( id ) {
			var attrId = id.replace( runescape, funescape );
			return function( elem ) {
				var node = typeof elem.getAttributeNode !== "undefined" && elem.getAttributeNode("id");
				return node && node.value === attrId;
			};
		};
	}

```
不过找了一下发现好像蛮接近的

```
	find: function( selector ) {
		var i,
			ret = [],
			self = this,
			len = self.length;

		if ( typeof selector !== "string" ) {
			return this.pushStack( jQuery( selector ).filter(function() {
				for ( i = 0; i < len; i++ ) {
					if ( jQuery.contains( self[ i ], this ) ) {
						return true;
					}
				}
			}) );
		}

		for ( i = 0; i < len; i++ ) {
			jQuery.find( selector, self[ i ], ret );
		}

		// Needed because $( selector, context ) becomes $( context ).find( selector )
		ret = this.pushStack( len > 1 ? jQuery.unique( ret ) : ret );
		ret.selector = this.selector ? this.selector + " " + selector : selector;
		return ret;
	},
```

今天先到这里。


