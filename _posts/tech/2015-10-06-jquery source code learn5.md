---
layout: post
title: Jquery 实现原理深入学习(5)
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

5.sizzle初步学习 √

6.attr功能函数实现 √

7.toggleClass功能函数实现(好伤) √

8.val功能函数实现 √

9.ajax异步请求以及扩展学习

```

## 正文
因为明天不上班，因此今天二更把计划中的几个一起搞定明天研究ajax。<(￣▽￣)> (事实上看着勾勾都被打完也是蛮爽的。)

### attr

[官方文档](http://api.jquery.com/attr/)这么描述

```
Get the value of an attribute for the first element in the set of matched elements or set one or more attributes for every matched element.
```

方法attr用于获取匹配元素集合中第一个元素的Html值，或者为匹配集合中的每个元素设置n个Html属性。

我以为它会很长，没想到代码是这样的

```
jQuery.fn.extend({
	attr: function( name, value ) {
		return access( this, jQuery.attr, name, value, arguments.length > 1 );
	}
	
```
看了下好像是传给access来处理了，找到access源码。

```
// 多功能函数，读取或设置集合的属性值；值为函数时会被执行
var access = jQuery.access = function( elems, fn, key, value, chainable, emptyGet, raw ) {
	var i = 0,
		length = elems.length,
		bulk = key == null;

	// 如果设置多个值那么就迭代设置
	// $('#box').attr({data:1,def:'addd'});
	/* key 为对象，就表明这是一个类似于 { height: 100, width: 200 } 的键值对，是将多步操作合并在了一起，而且这样的操作不能是 get 类方法，因为你没法确定最终的返回值是什么，所以 chainable 设置为 true，并且循环 key 对象来分别调用 access 方法。*/
	
	if ( jQuery.type( key ) === "object" ) { 
		chainable = true;//表示可以链式调用
		for ( i in key ) {
			jQuery.access( elems, fn, i, key[i], true, emptyGet, raw );
		}

  // 设置一个值
  /**
   * $('#box').attr('customvalue','abc')
   * $('#box').attr('customvalue',function (value) {});
   */
   
	} else if ( value !== undefined ) {
		chainable = true; 

		if ( !jQuery.isFunction( value ) ) {
			raw = true;
		}

		if ( bulk ) { //此处的 bulk 指的是批量操作
			// Bulk operations run against the entire set
			if ( raw ) { 
			 // jQuery.attr.call(elems,value); 调用完毕之后，将fn设置为空
				fn.call( elems, value );
				fn = null;

			// ...except when executing function values
	     /**
           * $('#box').attr(undefined,function () {})
           *
           * fn = bulk = jQuery.attr;
           *
           * fn = function (elem, key, value) {
           *  return jQuery.attr.call(jQuery(elem),value);
           * }
           *
           */
           
			} else { //这里的bulk是为了节省一个变量，将fn用bulk存起来，然后封装fn的调用
				bulk = fn;
				fn = function( elem, key, value ) {
					return bulk.call( jQuery( elem ), value );
				};
			}
		}
   //如果fn存在，掉调用每一个元素，无论key是否有值，都会走到这个判断，执行set动作
		if ( fn ) {
			for ( ; i < length; i++ ) {
				fn( elems[i], key, raw ? value : value.call( elems[i], i, fn( elems[i], key ) ) );
			}
		}
	}

/* 如果chainable为true，说明是个set方法，就返回elems
* 否则说明是get方法
* 1.如果bulk是个true，说明没有key值，调用fn，将elems传进去
* 2.如果bulk为false，说明key有值哦，然后判断元素的长度是否大于0
*    2.1 如果大于0，调用fn，传入elems[0]和key，完成get
*    2.2 如果为0，说明传参有问题，返回指定的空值emptyGet
*/
	return chainable ?
		elems :

		// Gets
		bulk ?
			fn.call( elems ) :
			length ? fn( elems[0], key ) : emptyGet;
};

```
正打算分解access,然后我突然脑抽了一下去github找Jquery项目看单独独立开来的attr.js,发现其实下面还有个attr。

```
jQuery.extend({
	attr: function( elem, name, value ) {
		var hooks, ret,
			nType = elem.nodeType;


		//不去获取或者设置节点为文本，注释，空节点的属性
		//ntype我们在上篇文章中已经知道了是js中当前节点的节点类型.
		if ( !elem || nType === 3 || nType === 8 || nType === 2 ) {
			return;
		}

		// 当不支持getAttribute方法时调用prop来设置属性,比如document或者文档碎片
		// 然后我发现很多文章居然说是支持attitude方法, 则调用property方法
	   // Fallback to prop when attributes are not supported
	   // 明明人家注释里写了Not好嘛。。。
	   
		if ( typeof elem.getAttribute === strundefined ) {
			return jQuery.prop( elem, name, value );
		}

		// 所有属性是小写的
		// Grab necessary hook if one is defined
		if ( nType !== 1 || !jQuery.isXMLDoc( elem ) ) {
		//elem不是标签或者elem不在xml中
			name = name.toLowerCase();//属性名大写
		//attrHooks是定义在下面的，我们跳到那里去
		//知道这个钩子大概是兼容处理我们继续往下走
			hooks = jQuery.attrHooks[ name ] ||
				( jQuery.expr.match.bool.test( name ) ? boolHook : nodeHook );
		}

		if ( value !== undefined ) { //这步是用来赋值的

			if ( value === null ) {//如果设的值为null，相当于移除attr ,比如.attr(‘checked‘,null);
				jQuery.removeAttr( elem, name );

			} else if ( hooks && "set" in hooks && (ret = hooks.set( elem, value, name )) !== undefined ) {
				return ret;

			} else {
				elem.setAttribute( name, value + "" ); //如果value是数值，隐式转为字符串
				return value;
			}

		} else if ( hooks && "get" in hooks && (ret = hooks.get( elem, name )) !== null ) {
			return ret;

		} else {
			ret = jQuery.find.attr( elem, name );
        //将null值修正为undefined
			// Non-existent attributes return null, we normalize to undefined
			return ret == null ?
				undefined :
				ret;
		}
	},
	
```
正疑惑为什么有两个处理方式时，看到了这句

`return access( this, jQuery.attr, name, value, arguments.length > 1 );`

事实上处理还是以Jquery.attr为主，access用来重载调用jQuery.attr。因为attr有好几种用法，但attr本身是不去判断的，而是由access处理后再载入它。这个函数重载的思路感觉很棒的样子。现在回到上列代码中开始学习attr的思路。

attrHooks的源码是这样的:

```
	attrHooks: {
		type: {
			set: function( elem, value ) {
			// ie9以下浏览器中 修改浏览器中有父元素的button,input元素的type属性,会报错
				if ( !support.radioValue && value === "radio" && jQuery.nodeName(elem, "input") ) {
	     // 兼容措施
					var val = elem.value;
					elem.setAttribute( "type", value );
					if ( val ) {
						elem.value = val;
					}
					return value;
				}
			}
		}
	}
});

```

我们现在回到最上面看access。access里的例子是我查到的，发现不同人读源码凭阅历真的读到不一样的东西，我现在只能大概理解一下基本的思路却无法领悟到精髓⊙﹏⊙‖∣
prop处理跟attr类似，就不举出来了。

### toggleClass

为什么要单独学习这个函数呢,因为之前我根据event.target来想批量设置上下文元素集合的class走了不少弯路。。。当初忘了有toggleClass这回事，写的心好累。。。

源码如下,先自己理解一遍。
先在官网看下例子
$( "#foo" ).toggleClass( className, addOrRemove );

$thisParagraph.toggleClass( "highlight", count % 3 === 0 );


```
   //根据例子可以看出一个value代表classname.stateval是一个表达式用于增加或删除class
	toggleClass: function( value, stateVal ) {
		var type = typeof value; //获取value类型
      
      //如果是value是字符串,stateval是bool型，直接添加或删除class
		if ( typeof stateVal === "boolean" && type === "string" ) {
			return stateVal ? this.addClass( value ) : this.removeClass( value );
		}
     
     //如果value是一个函数
		if ( jQuery.isFunction( value ) ) {
		//对其内部迭代执行toggleClass
			return this.each(function( i ) {
				jQuery( this ).toggleClass( value.call(this, i, this.className, stateVal), stateVal );
			});
		}

		return this.each(function() {
			if ( type === "string" ) {
				// 切换类名
				var className,
					i = 0,
					self = jQuery( this ),
					//找到定义var rnotwhite = (/\S+/g);
					//正则匹配非空白
					classNames = value.match( rnotwhite ) || [];
					
          //处理多个class
				while ( (className = classNames[ i++ ]) ) {
					// check each className given, space separated list
					if ( self.hasClass( className ) ) {
						self.removeClass( className );
					} else {
						self.addClass( className );
					}
				}

			// 改变整个类名 如果没有传入第一个参数或者第一个参数是undefined或者第一个参数是bool值
			} else if ( type === strundefined || type === "boolean" ) {
				if ( this.className ) {
					// 把类名存起来
					jQuery._data( this, "__className__", this.className );
				}

				// 如果元素已经存在类名或者我们传入false
				// 如果存在就移除
				// 否则添回原来的类名
				// 如果原来没有类名那么返回空
				this.className = this.className || value === false ? "" : jQuery._data( this, "__className__" ) || "";
			}
		});
	},

```
我们可以看到切换类名最重要的是_data把以前的类名存起来。看下_data的源码:

```
	// For internal use only.
	_data: function( elem, name, data ) {
		return internalData( elem, name, data, true );
	},
	
```

又扯出一个internalData，看下源码:

```
function internalData( elem, name, data, pvt /* Internal Use Only */ ) {
	if ( !jQuery.acceptData( elem ) ) {
		return; //这句话过滤掉了无法设置数据的元素。
	}


```

看参数列表就能明白，如果传入 data，表示设置数据，如果不传，则是读取数据，但是最后那个仅限内部使用的 pvt 参数是干什么的？
要知道 jQuery 内部同样会使用 data() 这样的方法来对元素进行数据操作，例如后面会讲到的关于事件处理部分。那这样就引发了一个问题，jQuery 如何来区分当前操作的数据是 jQuery 自己设置的，还是用户设置的？
解决办法就是将 jQuery 自己设置的数据和用户设置的数据分开。
    jQuery.cache[btn1.dataId] = {}；
这是前面用过的示例代码，实际上这个代码并不完整，最接近真实情况的代码是：
    jQuery.cache[btn1.dataId] = { data: {} };
看，最外层的对象是给 jQuery 自己用的，这个对象的 data 属性则是留给用户的，就是说，当 pvt 为 true，jQuery 就去操作外层对象，如果为 false，就去操作里面的 data 对象，如此一来，就不怕操作数据的时候冲突了。

```
	var ret, thisCache,
		internalKey = jQuery.expando,

//由于 IE 6-7 的问题，如果你在 DOM 节点上直接绑定 JavaScript 对象的话，垃圾回收器很可能会因为两者的循环引用而无法回收对象，因此造成内存泄露，而 JavaScript 对象则没有这方面的担忧，所以 jQuery 就直接将数据绑定到对象本身，而并非 jQuery.cache 中。
	if ( (!id || !cache[id] || (!pvt && !cache[id].data)) && data === undefined && typeof name === "string" ) {
		return;
	}

//  我们需要对 DOM 节点和 JS 对象分别处理，因为 IE6-7 无法正确回收 DOM 和 JS 互相引用的对象

//如果元素或对象没有关联过数据，那么就不做操作(这里指读取数据操作)。
	if ( !id ) {
		
		// ends up in the global cache
		if ( isNode ) {
			id = elem[ internalKey ] = deletedIds.pop() || jQuery.guid++;
		} else {
			id = internalKey;
		}
	}
	


	if ( !cache[ id ] ) {

		cache[ id ] = isNode ? {} : { toJSON: jQuery.noop };
	}

	
	if ( typeof name === "object" || typeof name === "function" ) {
		if ( pvt ) {
			cache[ id ] = jQuery.extend( cache[ id ], name );
		} else {
			cache[ id ].data = jQuery.extend( cache[ id ].data, name );
		}
	}

	thisCache = cache[ id ];

	// jQuery data() is stored in a separate object inside the object's internal data
	// cache in order to avoid key collisions between internal data and user-defined
	// data.
	if ( !pvt ) {
		if ( !thisCache.data ) {
			thisCache.data = {};
		}

		thisCache = thisCache.data;
	}

	if ( data !== undefined ) {
		thisCache[ jQuery.camelCase( name ) ] = data;
	}

	// Check for both converted-to-camel and non-converted data property names
	// If a data property was specified
	if ( typeof name === "string" ) {

		// First Try to find as-is property data
		ret = thisCache[ name ];

		// Test for null|undefined property data
		if ( ret == null ) {

			// Try to find the camelCased property
			ret = thisCache[ jQuery.camelCase( name ) ];
		}
	} else {
		ret = thisCache;
	}

	return ret;
}
```

这个源码读到一半我就停了，已经脱离了学习的目的=-=仰望的程度。




### val功能函数实现

直接看源码 

```
val: function( value ) {
		var hooks, ret, isFunction,
			elem = this[0];

		if ( !arguments.length ) {
			if ( elem ) {
		 //考虑元素是checkbox,radio,option或者select的情况，这时有val钩子
				hooks = jQuery.valHooks[ elem.type ] || jQuery.valHooks[ elem.nodeName.toLowerCase() ];
      //如果钩子存在且有get 且获取的值不为undefined，返回该值
				if ( hooks && "get" in hooks && (ret = hooks.get( elem, "value" )) !== undefined ) {
					return ret;
				}

				ret = elem.value;

				return typeof ret === "string" ?
					// handle most common string cases
					ret.replace(rreturn, "") :
					// handle cases where value is null/undef or number
					ret == null ? "" : ret;
			}

			return;
		}
```

粗略一看会发现，咦，思路和attr差不多，毕竟都是赋值。然后这些钩子的处理大多数都是兼容IE的处理。有兴趣的可以去看看。

今天就到这里。


