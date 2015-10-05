---
layout: post
title: Jquery 实现原理深入学习(2)
category: 技术
tags: [Jquery，Jquery学习,项目小结,前端学习,前端总结，javascript]
keywords: 前端,资料,学习
description: 
--- 
## 前言
```
1.总体结构 √

2.构建函数 √

3.each功能函数实现√

4.map功能函数实现

5.sizzle初步学习

6.attr功能函数实现

7.toggleClass功能函数实现(好伤)

8.val功能函数实现

9.ajax异步请求以及扩展学习
```

##正文
今天是学习jquery的each的实现。

首先先去[官方文档](http://api.jquery.com/each/)看下each的api，看下each具体能做什么(事实上我一直用for-0-).

以前我对each的印象就是能迭代元素，类似for的效果。

先看下最简单的实现。

```
<ul>
  <li>foo</li>
  <li>bar</li>
</ul>

$( "li" ).each(function( index ) {
  console.log( index + ": " + $( this ).text() );
});

result:

0: foo 
1: bar

```
![imgn](http://haoqiao.qiniudn.com/jquerylearn1.png)

可以看见对li进行迭代并且对每一项都传递执行一个函数。说明each中返回参数至少有一个计数的。

官网的例子都大同小异。因此我们开始翻找源码。

```
each: function( callback, args ) {
	return jQuery.each( this, callback, args );
},
	
...

//传递一个对象，一个回调函数，一个参数列表
each: function( obj, callback, args ) {
	var value,
		i = 0,
		length = obj.length, //获得对象的长度
		isArray = isArraylike( obj ); //判断是否是数组

	if ( args ) { //如果有参数
		if ( isArray ) { //如果是数组
			for ( ; i < length; i++ ) {
				value = callback.apply( obj[ i ], args ); //将参数传给回调函数。执行回调时通过apply指定this关键字所引用的对象

				if ( value === false ) {
					break;
				}
			}
		} else {
			for ( i in obj ) {//如果是对象，用户for in 来遍历
				value = callback.apply( obj[ i ], args );//对对象中的属性传参数

				if ( value === false ) {
					break;
				}
			}
		}

	// A special, fast, case for the most common use of each
	} else {//如果没有参数
		if ( isArray ) { //判断如果是数组
			for ( ; i < length; i++ ) {
				value = callback.call( obj[ i ], i, obj[ i ] ); //将计数i传给回调函数

				if ( value === false ) {
					break;
				}
			}
		} else {
			for ( i in obj ) {
				value = callback.call( obj[ i ], i, obj[ i ] );

				if ( value === false ) {
					break;
				}
			}
		}
	}

	return obj; //返回传入的参数Obj 调用时把当前jquery对象作为参数obj传入，这里返回，以支持链式语法
},
```
看到有一个isArraylike，我们来看下它是怎么写的.

```
function isArraylike( obj ) {

	// Support: iOS 8.2 (not reproducible in simulator)
	// `in` check used to prevent JIT error (gh-2145)
	// hasOwn isn't used here due to false negatives
	// regarding Nodelist length in IE
	var length = "length" in obj && obj.length,
		type = jQuery.type( obj );

	if ( type === "function" || jQuery.isWindow( obj ) ) {
		return false;
	}

	if ( obj.nodeType === 1 && length ) {
		return true;
	}

	return type === "array" || length === 0 ||
		typeof length === "number" && length > 0 && ( length - 1 ) in obj;
}
```

Jquery each的写法很简练，因此我想看看underscore中它each写法的不同.

```
// 迭代处理器, 对集合中每一个元素执行处理器方法
var each = _.each = _.forEach = function(obj, iterator, context) {
   // 不处理空值
   if(obj == null)
       return;
   if(nativeForEach && obj.forEach === nativeForEach) {
       // 如果宿主环境支持, 则优先调用JavaScript 1.6提供的forEach方法
       obj.forEach(iterator, context);
   } else if(obj.length === +obj.length) {
       // 对<数组>中每一个元素执行处理器方法
       for(var i = 0, l = obj.length; i < l; i++) {
           if( i in obj && iterator.call(context, obj[i], i, obj) === breaker)
               return;
       }
   } else {
       // 对<对象>中每一个元素执行处理器方法
       for(var key in obj) {
           if(_.has(obj, key)) {
               if(iterator.call(context, obj[key], key, obj) === breaker)
                   return;
           }
       }
   }
};
```

underscore的each函数对对象的处理用到了has.这在最末尾有定义。

```

// 检查一个属性是否属于对象本身, 而非原型链中
_.has = function(obj, key) {
   return hasOwnProperty.call(obj, key);
};

```
而hasOwnProperty.call也在头部有定义

```
// 将内置对象原型中的常用方法缓存在局部变量, 方便快速调用
var slice = ArrayProto.slice, //
unshift = ArrayProto.unshift, //
toString = ObjProto.toString, //
hasOwnProperty = ObjProto.hasOwnProperty;

```

underscore的写法和jquery有些不同。

```
1.它先判断能不能用宿主环境自带的foreach来实现。

2.jquery是用略显长的方法对是否传入参数判断写了两种，可以考虑合并但是合并后需要返回判断isobj因此jquery用略显长的形式来避免性能下降。

而underscore没有考虑参数的传入而是直接对数组/对象进行迭代。

3.jquery判断数组和对象是用自己写的isarray来判断。

而undersore是用这句命令

obj.length === +obj.length

事实上我也不了解为什么这么写，我只知道===全等，这个+号我就不确定是自增还是什么。于是我去查了一下。

```

先说一下全等号的作用，对于==，666 == '666'，这返回结果为true，对于===，666 === '666' 返回false

这是测试结果

![imgn](http://haoqiao.qiniudn.com/jquerylearn2.png)

全等符号是不会将比较的对象进行类型转换的。这里能想到obj.length === +obj.length应该是对象类型进行判断，但是怎么判断的？继续看资料-0-

```
+号的作用不是自增，'+'号其实是将后面跟的操作数转型成了数字类型。
```

![imgn](http://haoqiao.qiniudn.com/jquerylearn3.png)

![imgn](http://haoqiao.qiniudn.com/jquerylearn4.png)

如果obj是一个string类型，如"abc",我们可以拿到length属性，如果是一个function，或者一个数组，我们都可以拿到他们的length属性，但如果是一个object类型的数据，它可能是不包含length属性的。对于非数组、非字符串、非函数类型的数据，我们可以尝试使用for in循环来遍历数据.

比较它和jquery的isArraylike来看，这种写法好像更加简练。不过Jquery判断的更加全面。

从underscore命名形式来看，更容易理解。比如

```
iterator.call(context, obj[i], i, obj)

```
很清楚知道将this指向当前上下文。


今天就到这里。


