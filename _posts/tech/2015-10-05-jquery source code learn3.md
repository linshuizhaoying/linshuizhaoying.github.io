---
layout: post
title: Jquery 实现原理深入学习(3)
category: 技术
tags: [Jquery，Jquery学习,项目小结,前端学习,前端总结，javascript]
keywords: 前端,资料,学习
description: 
--- 
## 前言

```
1.总体结构 √

2.构建函数 √

3.each功能函数实现 √

4.map功能函数实现 √

5.sizzle初步学习

6.attr功能函数实现

7.toggleClass功能函数实现(好伤)

8.val功能函数实现

9.ajax异步请求以及扩展学习

```

## 正文
如果仔细看日期你会发现今天写了两篇，是不是感觉奇怪为什么会爆发？因为这几天攒了好多学习资料准备写Blog,而且近期看到不少激励自己的东西因此激动不已的开始了今天的第二篇。

╮(╯▽╰)╭

根据当初的设定这篇应该是map函数了。

重新打开sublime然后载入jquery1.11.3

迅速的定位到了map。然后发现自己好像也从来没怎么用过Map=-=(泪目)。于是按规矩去官网看看[文档](http://api.jquery.com/map/)。

```

 Pass each element in the current matched set through a function, producing a new jQuery object containing the return values.
 
```

看描述遍历当前对象的每一个元素，在其上执行回调函数，并产生一个包含返回值的jquery对象。该方法常用于获取dom元素集合的值或者设置值。首先自己解读下源码。


```

// arg is for internal usage only
map: function( elems, callback, arg ) {
	var value,
		i = 0,
		length = elems.length,
		isArray = isArraylike( elems ),
		ret = [];

	// Go through the array, translating each of the items to their new values
	if ( isArray ) { //判断是数组还是对象，数组用for循环，对象用for in遍历其属性.
		for ( ; i < length; i++ ) {
			value = callback( elems[ i ], i, arg );

			if ( value != null ) {
				ret.push( value ); //将每一个执行回调函数的元素的返回值添加到数组中。
			}
		}

	// Go through every key on the object,
	} else {
		for ( i in elems ) {
			value = callback( elems[ i ], i, arg );

			if ( value != null ) {
				ret.push( value );
			}
		}
	}

	// Flatten any nested arrays
	return concat.apply( [], ret ); //这里是一个难点
	
```

思路其实很好理解。但难的其中一些写法能够深深的伤害到基础薄弱的我。比如

`return concat.apply( [], ret )`

对于concat() 方法用于连接两个或多个数组。该方法不会改变现有的数组，而仅仅会返回被连接数组的一个副本。

但是一般例子都是

```
var arr =[1,2,3];

var arr2 = [4,5];

arr.concat(arr2);//return [1,2,3,4,5]

```

那为毛是apply的调用方式呢？无法理解于是继续查。

查看源码发现在开头

```
var deletedIds = [];

var slice = deletedIds.slice;

var concat = deletedIds.concat;

...

```

deletedIds 看名字好像只是一个中间的变量，而且翻了一下上下文，并没有啥赋值。那么姑且可以当做空数组看待。

然后看了下书上的。1.7的版本是这样的

```
var ret = [];
...
ret[ret.length] = value;
...

return ret.concat.apply([],ret);

```

在空数组[]上调用方法concat扁平化结果集ret中的元素。看到这个解释我的表情是这样的(⊙o⊙)…

扁平化结果集是神马。。。

虽然能大概理解但是我还是google了一下。

```
关键作用的是apply,因为apply的第二个参数把ret的数组分成多个参数传入给concat

作用:把二维数组转化为一维数组

```

并且在其它地方找到了一个例子

```
$.map( [0,1,2], function(n){
  return [ n, n + 1 ];
});
//输出：[0, 1, 1, 2, 2, 3]
//如果是return ret的话，输出将会是：[[0,1], [1,2], [2,3]]

```

其实这个应该是自己慢慢试出来，不过没关系，我们可以弥补一下去测试看看。

![imgn](http://haoqiao.qiniudn.com/concat1.png)

![imgn](http://haoqiao.qiniudn.com/concat2.png)

发现结果确实如此。

然后呢，按理说应该到这里结束了，但是我"突然"想起underscore里好像也有map的说于是。。。

```
   _.map = _.collect = function(obj, iterator, context) {
        // 用于存放返回值的数组
        var results = [];
        if(obj == null)
            return results;
        // 优先调用宿主环境提供的map方法
        if(nativeMap && obj.map === nativeMap)
            return obj.map(iterator, context);
        // 迭代处理集合中的元素
        each(obj, function(value, index, list) {
            // 将每次迭代处理的返回值存储到results数组
            results[results.length] = iterator.call(context, value, index, list);
        });
        // 返回处理结果
        if(obj.length === +obj.length)
        //如果是数组的话，一定要返回相同长度的新数组
            results.length = obj.length;
        return results;
    };
```

可以发现Underscore依旧那么省。。。而且还那么自信，直接把之前的each拿过来用。。。

而且看它的results结果怪怪的。于是在jsfiddle上加载最新版本的underscore。并执行

```
console.log(_.map([1, 2, 3], function(n){ return [ n, n + 1 ]; }));

```

![imgn](http://haoqiao.qiniudn.com/concat3.png)

发现结果如上图，并没有做jquery的扁平化结果集。

嗯，两者好坏我现在是没资格评论。不过按照写法而言，更喜欢underscore的。

今天就到这里。现在时间晚上11:11。好虐的时间。


