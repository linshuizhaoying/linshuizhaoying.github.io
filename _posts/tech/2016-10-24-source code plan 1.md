---
layout: post
title: 源码阅读计划之underscore数组篇
tags: [学习,整理,前端,实战总结,源码学习,source code]
keywords: web,代码,前端,学习总结,源码阅读,源码学习,源码细节
description: 
---

# 讲在前面

之前因为一些项目暂停了学习的计划，然后完成这些项目打算放松一下,完成之前列的一些计划再去搞优化。
这次选的计划是源码阅读计划,因为之前花时间x重新再过了高程和权威,觉得得看一些实在的东西来提高自己。因此选择了阅读一些优秀的库和插件还有框架来打磨打磨。
这次学习计划肯定不能仅仅只阅读代码写注释那么简单。我打算更加细致一点。

## 准备工作

### 第一个是工具，对于工具很多人可能会直接操起手中的代码编辑器直接来阅读，这的确是阅读的手段之一，但是我这里要安利一个工具`lambda-view` [github](https://github.com/Jianru-Lin/lambda-view)

![imgn](http://haoqiao.qiniudn.com/lv.png)

### 第二个是中文文档,当然如果只有英文文档也是可以的。

### 第三个是测试用例，这个和第二个可能会有点重复，但是如果想看一些细节还是看测试比较方便。

### 第四个比较重要，当然有一些框架或者库可能没有，那就是完整的源码解析，这是用来作为最后看完对照自己思路是否正确的一个好方法。这个方法必须在你先看完所有源码之后再来看，不然不会有太大效果。

### 第五个 一个调试工具，可以是chrome,可以是codepen，也可以是自己写的前端调试工具。

## Underscore源码学习

第一份我先挑一个简单的来学习。一个工具库的写法足以让我们入门。

先来看下简介:

`Underscore一个JavaScript实用库，提供了一整套函数式编程的实用功能，但是没有扩展任何JavaScript内置对象`

而且作为一个比较成熟的库它其实已经有很详细的注释了,可以点这里看[对照](http://www.css88.com/doc/underscore/docs/underscore.html)

作为这个库的新手我们先看下这个库感兴趣的部分，我比较感兴趣的是数组这部分,因为它看上去比较简单

```

数组(Arrays)
- first
- initial
- last
- rest
- compact
- flatten
- without
- union
- intersection
- difference
- uniq
- zip
- unzip
- object
- indexOf
- lastIndexOf
- sortedIndex
- findIndex
- findLastIndex
- range


```

我们也不从什么大局观先看，就从简单的这部分看一下，每个函数对应的方法名都很清晰。

从`first`开始,先看官方文档调用用例。

```

first_.first(array, [n]) Alias: head, take 
返回array（数组）的第一个元素。传递 n参数将返回数组中从第一个元素开始的n个元素。

返回数组中前 n 个元素

```

然后用`sublime`打开从`github`上下载下来的文件夹，查看test目录里的arrays.js

![imgn](http://haoqiao.qiniudn.com/underscore-array1.png)

然后看`first`的断言

```

    assert.strictEqual(_.first([1, 2, 3]), 1, 'can pull out the first element of an array');
    assert.strictEqual(_([1, 2, 3]).first(), 1, 'can perform OO-style "first()"');
    assert.deepEqual(_.first([1, 2, 3], 0), [], 'returns an empty array when n <= 0 (0 case)');
    assert.deepEqual(_.first([1, 2, 3], -1), [], 'returns an empty array when n <= 0 (negative case)');
    assert.deepEqual(_.first([1, 2, 3], 2), [1, 2], 'can fetch the first n elements');
    assert.deepEqual(_.first([1, 2, 3], 5), [1, 2, 3], 'returns the whole array if n > length');
    
```

这样我们好像可以根据这个断言写一个猜测的代码了

```

function _first(array,n) {
   var temp = [];
   for(var i = 0 ; i <= n; i++){
     temp.push(array[i])
   }
   return temp
}

```

然后用chrome跑一下,发现

![imgn](http://haoqiao.qiniudn.com/underscore-array2.png)

当测试n>0的时候正常，n=0的时候发现边界不对。我们改改

```

function _first(array,n) {
   var temp = [];
   for(var i = 1 ; i <= n; i++){
     temp.push(array[i-1])
   }
   return temp
}

```

然后我们再测试一下大于数组长度的数值时会发现:

`[1, 2, 3, 4, undefined]`

返回的数组里面有个`undefined`这说明我们写的还是不够严谨，再改一下

```

function _first(array,n) {
   var temp = [];
   var t = Math.min(array.length,n)
   for(var i = 1 ; i <= t; i++){
     temp.push(array[i-1])
   }
   return temp
}

```

这样根据断言来看我们写的函数已经符合了，现在我们再来看看源码是怎么写的。

```

为了便于浏览我从lambda-view中截取源码

_.first = _.head = _.take = 
function (array, n, guard) 
{
if (((array) == (null)) || ((array.length) < (1))) 
return void (0) ;
if (((n) == (null)) || (guard)) 
return array[0] ;
return _.initial (array, (array.length) - (n)) ;

```

是不是乍一看和想象的不一样。我们可以看到最后实现功能的是`_.initial`

我们看下官方文档对它的描述

```

initial_.initial(array, [n]) 
返回数组中除了最后一个元素外的其他全部元素。 在arguments对象上特别有用。传递 n参数将从结果中排除从最后一个开始的n个元素.排除数组后面的 n 个元素

```

然后我们看下`_.initial`的源码

```

  _.initial = function(array, n, guard) {
    return slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
  };
 
```

我们先要找一下slice的定义，在源码最开头搜索

```

  var ArrayProto = Array.prototype, ObjProto = Object.prototype;
  var SymbolProto = typeof Symbol !== 'undefined' ? Symbol.prototype : null;

  // Create quick reference variables for speed access to core prototypes.
  var push = ArrayProto.push,
      slice = ArrayProto.slice,
      toString = ObjProto.toString,
      hasOwnProperty = ObjProto.hasOwnProperty;
      
```

所以`slice` 就是 `Array.prototype.slice`

然后我们跑一下改编后的代码

```

var a =[1,2,3,4]
var n = 5
  _first = _head = _take = function(array, n, guard) {
    if (array == null || array.length < 1) return void 0;
    if (n == null || guard) return array[0];
    return _initial(array, array.length - n);
  };

  _initial = function(array, n, guard) {
    return Array.prototype.slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
  };

_first(a,n)

[1, 2, 3, 4]

```

先不提这个实现好像复杂化了一点东西，但是我们还是看看这个实现。一层一层来看

因为guard我们是没有传东西进去的,一开始我也不了解为啥有这个参数。
然后我们可以看到注释里有一句The guard check allows it to work with _.map. 具体可以去看[stackoverflow.com回复](http://stackoverflow.com/questions/18639936/what-does-the-passed-parameter-guard-check-in-underscore-js-functions的回复)。这里先不细展开

`Math.max(0, array.length - (n == null || guard ? 1 : n))`

等价于

`Math.max(0,array.length - n）`

然后我们知道`slice()` 接受一个或两个参数（返回项的起始位置和结束位置） 从当前数组中按要求返回新数组。

`call()`  接受两个参数，一个是在其中运行函数的作用域（this）,一个是参数列表

所以

`Array.prototype.slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
`

我们就知道了它处理的方式和我们写的是相反的，又因为在`first`里面`_initial(array, array.length - n)` 这样负负得正。

这个虽然写的比我们复杂一点，但是它还是很严谨的。比如我们没有考虑到判断传进来的数组一开始就是null，不存在的情况。

接下来我们就不需要像之前那么繁琐的看`array`相关后面的代码了。因为基本上都是用原生的`slice`方法来处理的。

## 从头看起

当我们了解了一个内部函数的大致写法我们应该学习一下整体然后再从看下来有个更清晰的认知。

从这部开始我们需要一个已经加载了`underscore`的静态页面，这样方便我们从chrome直接调用里面的参数。

事实上我在上课的时候用ipad浏览过两遍整个源码，我觉得新人不应该去抓那些比较复杂的函数的实现，而是先把整个架构搞搞懂，然后很多大牛的文章其实是已经跳过这部分了，因此我将比较扩展的把`头`给理清楚。先简单化整个流程。


```
//用闭包保存整个库

(function() {
  // 在1.8.2版本其实下面这句只有 var root = this;
  // 也就是只是把 this 赋值给局部变量 root
  //但是1.8.3更新了是为了适应Node环境下引用，确认环境的全局命名空间,浏览器下是window,服务器上是globa，用self代替这两者
  
  var root = typeof self == 'object' && self.self === self && self ||
            typeof global == 'object' && global.global === global && global ||
            this;
// 原来全局环境中的变量 `_` 赋值给变量 previousUnderscore 进行缓存，这里不用管它，它是为了以后noConflict才用
  var previousUnderscore = root._;

//很自然的能看出这里把原生的Array和Object保存到变量，一个是为了方便引用，一个是为了压缩代码。
  var ArrayProto = Array.prototype, ObjProto = Object.prototype;
  var SymbolProto = typeof Symbol !== 'undefined' ? Symbol.prototype : null;

// 把ES5原生的方法缓存一下，以后函数调用的时候先判断是不是有原生的方法，有就调用，没有就用自己实现的那套~

  var push = ArrayProto.push,
      slice = ArrayProto.slice,
      toString = ObjProto.toString,
      hasOwnProperty = ObjProto.hasOwnProperty;

  var nativeIsArray = Array.isArray,
      nativeKeys = Object.keys,
      nativeCreate = Object.create;

//用于baseCreate函数里面，我也不是很清楚为什么这么提前声明，用于代理原型交换的空函数
  var Ctor = function(){};
// 这个就是安全引用对象的方法，先判断入对象是不是_的实例，如果是就直接返回obj，不然用new来实例化再返回,最后要把对象赋值给wrapped，其他函数里有用到

  var _ = function(obj) {
    if (obj instanceof _) return obj;
    if (!(this instanceof _)) return new _(obj);
    this._wrapped = obj;
  };
  
 // 导出Underscore对象给node.js，如果在浏览器环境中，顺便把`_`添加给全局对象root
 
  if (typeof exports != 'undefined' && !exports.nodeType) {
    if (typeof module != 'undefined' && !module.nodeType && module.exports) {
      exports = module.exports = _;
    }
    exports._ = _;
  } else {
    root._ = _;
  }

//当前版本号，
  _.VERSION = '1.8.3';

// 这段代码一开始我也看晕了，但仔细一看就是判断参数个数然后返回调用，返回一些回调、迭代方法，这里我把代码收缩在下文慢慢分析

  var optimizeCb = function(func, context, argCount) {  };


  var builtinIteratee;
  
 // callback 的缩写 回调生成方法 很多地方用到，可以说理解这个和上面的就可以写一个简单的库了
 
  var cb = function(value, context, argCount) {  };

// 对cb的封装，默认的迭代器，我们可以看到它是传递了一个无穷的值作为argCount传入cb 所以具体我们要看下面的cb的分析

  _.iteratee = builtinIteratee = function(value, context) {
    return cb(value, context, Infinity);
  };

//等价于ES6的rest参数。它将起始索引后的参数放入一个数组中。这里我找个栗子举一下让大家理解
/**
var f = function(a, b, ...theArgs) { 
    ...
}

f(1, 2, 3, 4, 5) // a=1, b=2, theArgs=[3, 4, 5]
请注意f的第三个参数，在声明时以'...'开头。这样在实际调用时，函数的前两个参数分别映射成a、b,从第三个参数开始，这些参数按照顺序映射成名为theArgs的数组。

**/
//具体我们也单独拉出来讲
  var restArgs = function(func, startIndex) { };

//创建一个继承其他函数的新对象,就是一个原型式继承
  var baseCreate = function(prototype) {
  };
  
//获取对象的属性的键值
  var shallowProperty = function(key) {
  };
  
  var MAX_ARRAY_INDEX = Math.pow(2, 53) - 1;
  var getLength = shallowProperty('length');
  
  //判断是不是类数组，所谓类数组就是即拥有 length 属性并且 length 属性值为 Number 类型的元素，数组，包括类似 {length: 10} 这样的对象，字符串、函数
  
  var isArrayLike = function(collection) {
    var length = getLength(collection);
    return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
  };
  
  ....一堆函数
 
// 兼容 AMD 规范
/* 这么写是因为amd是这么调用的：
define(['underscore'], function ( _) {
function a(){}; // 私有方法，因为没有被返回(见下面)
function b(){}; // 公共方法，因为被返回了
function c(){}; // 公共方法，因为被返回了
     //    暴露公共方法
    return {
        b: b,
        c: c
    }
});
*/

if (typeof define == 'function' && define.amd) {
    define('underscore', [], function() {
      return _;
    });
  }
}());

  
```
  
快速过了一遍整个`underscore`的结构，接下来我们就对之前缩放的内容进行更详细的学习.


```
optimizeCb:

var optimizeCb = function(func, context, argCount) {
    // 没有上下文直接返回函数
    if (context === void 0) return func;
    
    // 对传进来的参数个数进行判断，不同个数不同调用方式
    // 接下来的switch其实只是一个传递参数规范例子0-0没有什么软用。自己写的时候肯定会把这段去掉，因为也不影响。为了在已知参数数量的情况下让 js 引擎做出优化（避免使用 arguments）

    switch (argCount == null ? 3 : argCount) {
      case 1: return function(value) {
        return func.call(context, value);
      };

      case 3: return function(value, index, collection) {
        return func.call(context, value, index, collection);
      };
      case 4: return function(accumulator, value, index, collection) {
        return func.call(context, accumulator, value, index, collection);
      };
    }
    return function() {
      return func.apply(context, arguments);
    };
  };

```

再来看cb

```
cb:

 var cb = function(value, context, argCount) {
  //如果用户修改了迭代器，则使用新的迭代器 因为builtinIteratee我们可以在下面看到是有定义的，只有当默认迭代器被修改了_.iteratee !== builtinIteratee才会返回true
    if (_.iteratee !== builtinIteratee) return _.iteratee(value, context);
    /*
      _.identity = function(value) {
		    return value;
		  };
    /*
    // 如果传入的值是空，那么就表示返回等价的自身
    if (value == null) return _.identity;
    // 如果是函数，就返还该函数的调用
    if (_.isFunction(value)) return optimizeCb(value, context, argCount);
    // 如果是对象或者数组，寻找匹配的属性值
    if (_.isObject(value) && !_.isArray(value)) return _.matcher(value);
    //如果都不是，返相应的属性访问器
    return _.property(value);
  };

```

这段代码其它都挺好理解，就是`value`相当于传进来的一个附加条件，可能这个`value`名字起的太有迷惑性，一开始我以为就是个值或者对象。但是看到后面以及看到其它函数调用它的方式就明白了：

```
  _.filter = _.select = function(obj, predicate, context) {
    var results = [];
    predicate = cb(predicate, context);
    
    _.filter([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });
    
```

至于为什么有`if (value == null) return _.identity;`这句是因为有种情况下你可能不传值进来，比如

```
_.filter([1,2,3]) =>[1, 2, 3]

```

接下来是restArgs

```

restArgs:
  
  // 传入一个函数，一个开始位置标志
  var restArgs = function(func, startIndex) {
  
  //这个函数可以把一个函数func的参数"改造"成Rest Parameters,如果不传第二个参数startIndex，默认用最后一个参数收集其余参数
  /*
    _.delay = restArgs(function(func, wait, args) {
    return setTimeout(function() {
      return func.apply(null, args);
    }, wait);
  });
  
  比如这个函数一开始就用restArgs处理，而且没有传startIndex，因此将会默认用args来收集"剩余参数"
  
  */
    startIndex = startIndex == null ? func.length - 1 : +startIndex;
    return function() {
      var length = Math.max(arguments.length - startIndex, 0),
          rest = Array(length),
          index = 0;
      //收集一个个剩余参数
      for (; index < length; index++) {
        rest[index] = arguments[index + startIndex];
      }
      //估摸着也是为了优化先判断几个startIndex值比较小的情况
      switch (startIndex) {
        case 0: return func.call(this, rest);
        case 1: return func.call(this, arguments[0], rest);
        case 2: return func.call(this, arguments[0], arguments[1], rest);
      }
      var args = Array(startIndex + 1);
      for (index = 0; index < startIndex; index++) {
        args[index] = arguments[index];
      }
      args[startIndex] = rest;
      // 发现值一个个判断好像有点蠢，那不如一次性apply掉。
      return func.apply(this, args);
    };
  };


```

最后我们看下baseCreate

```
baseCreate:

  var baseCreate = function(prototype) {
    if (!_.isObject(prototype)) return {};
    if (nativeCreate) return nativeCreate(prototype);
    Ctor.prototype = prototype;
    var result = new Ctor;
    Ctor.prototype = null;
    return result;
  };

Ctor我们之前提到了这个是空对象，用于代理原型交换的空函数。可能这样不是很眼熟，我扔一个例子

原型式继承


function inheritObject(o){
  function F(){}
  F.prototype = o
  return new F()
}

var book = {
  name:"books",
  allbooks:['css','html']
}
var a1 = inheritObject(book)
a1.allbooks.push('js')

var a2 = inheritObject(book)
console.log(a2.allbooks) //["css", "html", "js"]

这个估计就能看懂。了啥也不用说了0-0，就一个正规的原型式继承。

```

接下来我们可要进入正文了，是的前面基本属于铺垫2333

我们学习一个库的源码会发现其中一些功能在自己日常中肯定是用不到的，自己造一个轮子又无从下手，那么基于一个轮子的改造我觉得至少我们都会。因此接下来是对整个库的函数分析提取自己想要的，然后看看是不是能根据之前的学习搞出一个迷你版的`underscore`.当然不能因为比较简单就复制粘贴。至少要手打一遍来加深理解。
而且敲的过程中我们可以尝试去掉一些代码看看能不能简化。

首先我们为了测试需要建立一个`test.html` 然后引入我们的`tools.js`文件，然后我们先把整个架构仿照`Underscore`搭起来。

```

/**
 * Easy Tools Function
 * Learning Underscore
 * 2016-10-26
 */
(function() {
  var root = typeof self =='object' && self.self === self && self || typeof global == 'object'
             && global.global === global && global || this;

  // 先缓存一下，万一以后要用到呢
  var preRoot = root._;
  // 保存Native方法,为了方便引用和压缩代码，把ES5原生的方法缓存一下，以后函数调用的时候先判断是不是有原生的方法，有就调用，没有就用自己实现的那套
  var ArrayProto = Array.prototype, ObjProto = Object.prototype;

  var push = ArrayProto.push,
      slice = ArrayProto.slice,
      toString = ObjProto.toString,
      hasOwnProperty = ObjProto.hasOwnProperty;

  var nativeIsArray = Array.isArray,
      nativeKeys = Object.keys,
      nativeCreate = Object.create;
  /**
   * Main Function
   */
  // 安全引用对象的方法，先判断传入对象是不是_的实例，如果是就直接返回obj，不然用new来实例化再返回,最后要把对象赋值给wrapped，其他函数里有用到
  var _ = function(obj) {
    if (obj instanceof _) return obj;
    if (!(this instanceof _)) return new _(obj);
    this._wrapped = obj;
  };

  _.VERSION = '0.0.1';
  // 导出Underscore对象给node.js，如果在浏览器环境中，顺便把`_`添加给全局对象root
  if (typeof exports != 'undefined' && !exports.nodeType) {
    if (typeof module != 'undefined' && !module.nodeType && module.exports) {
      exports = module.exports = _;
    }
    exports._ = _;
  } else {
    root._ = _;
  }

  /** 基础判断函数  */

  // 处理一些浏览器下的Bug
  var nodelist = root.document && root.document.childNodes;
  if (typeof /./ != 'function' && typeof Int8Array != 'object' && typeof nodelist != 'function') {
    _.isFunction = function(obj) {
      return typeof obj == 'function' || false;
    };
  }

  _.isObject = function(obj) {
    var type = typeof obj;
    return type === 'function' || type === 'object' && !!obj;
  };



  /** 库处理函数 */

  // 判断参数个数然后返回调用 删去了原来那个优化代码,使其看起来更加简便
  var optimizeCb = function(func, context, argCount) {
  	if (context === void 0) return func;
    return function() {
      return func.apply(context, arguments);
    };
  };

  var originIteratee;
  // callback 回调生成方法
  var cb = function(value, context, argCount){
  	if (_.iteratee !== originIteratee) return _.iteratee(value, context);
    if (value == null) return _.identity;
    if (_.isFunction(value)) return optimizeCb(value, context, argCount);
    if (_.isObejct(value) && !_.isArray(value)) return _.matcher(value);
    return _.property(value);
  };
  // 对cb的封装，默认的迭代器，我们可以看到它是传递了一个无穷的值作为argCount传入cb
  _.iteratee = originIteratee = function(value, context) {
    return cb(value, context, Infinity);
  };
  // 等价于ES6的rest参数。它将起始索引后的参数放入一个数组中
  var restArgs = function(func, startIndex) {
    startIndex = startIndex == null ? func.length - 1 : +startIndex;
    return function() {
      var length = Math.max(arguments.length - startIndex, 0),
          rest = Array(length),
          index = 0;
      for (; index < length; index++) {
        rest[index] = arguments[index + startIndex];
      }
      var args = Array(startIndex + 1);
      for (index = 0; index < startIndex; index++) {
        args[index] = arguments[index];
      }
      args[startIndex] = rest;
      return func.apply(this, args);
    };
  };
  // 原型式继承
  var Ctor = {};
  var baseCreate = function(prototype) {
    if (!_.isObject(prototype)) return {};
    if (nativeCreate) return nativeCreate(prototype);
    Ctor.prototype = prototype;
    var result = new Ctor;
    Ctor.prototype = null;
    return result;
  };
  // 获取对象的属性的键值
  var shallowProperty = function(key) {
    return function(obj) {
      return obj == null ? void 0 : obj[key];
    };
  };

  var MAX_ARRAY_INDEX = Math.pow(2, 53) - 1;
  var getLength = shallowProperty('length');
  //判断是不是类数组，所谓类数组就是即拥有 length 属性并且 length 属性值为 Number 类型的元素，数组，包括类似 {length: 10} 这样的对象，字符串、函数
  var isArrayLike = function(collection) {
    var length = getLength(collection);
    return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
  };

  // 数组处理函数

  /*
   * [返回数组中除了最后一个元素外的其他全部元素]
   * [传递 n参数将从结果中排除从最后一个开始的n个元素]
   * _.initial([5, 4, 3, 2, 1]);
   * =>[5, 4, 3, 2]
   *
   * _.initial([1, 2, 3, 4], 2)
   * =>[1, 2]
   */
  _.initial = function(array, n, guard) {
    return slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
  };

  /**
   * [返回array（数组）的第n个元素]
   *  _.first([1, 2, 3], 2)
   *  =>[1, 2]
   */
  _.first = function(array, n, guard) {
    if (array == null || array.length < 1) return void 0;
    if (n == null || guard) return array[0];
    return _.initial(array, array.length - n);
  };


}());


```

可以看到简单的整理了一下，然后就是一个个函数看过来，觉得有用的把它一个个添加进来就可以了。或许看到最后我们还可以把整个不需要的部分再删去一部分。

这样我们从数组这边的函数一个个翻觉得自己能在开发中实用的函数会遇到一个问题，那就是我们之前没有仔细看过`集合`那块的函数，而数组这边有不少复杂的处理都会用到，因此我们在看代码的时候需要回翻哪个函数依赖了哪些`集合`的处理函数。而且我们需要暂时姑且认为这个库的解决方式是比较最优的解，当我们以后看别的库发现有不同实现方式时，我们需要回到这个库再进行对比。


首先我们来看一个典型例子:

```

 _.uniq = function(array, isSorted, iteratee, context) {
    // 判断是不是已经排序的，如果不是那么改变参数变成
    // _.uniq(array, false, undefined, iteratee)
    if (!_.isBoolean(isSorted)) {
      context = iteratee;
      iteratee = isSorted;
      isSorted = false;
    }
    // 这里iteratee是一个自定义的迭代函数,它可以指定你想要排重的那一项
    // 在单元测试arrays文件里中有这么一条,我将它提出来
   /* var list = [{name: 'Moe'}, {name: 'Curly'}, {name: 'Larry'}, {name: 'Curly'}];
    var expected = [{name: 'Moe'}, {name: 'Curly'}, {name: 'Larry'}];
    var iterator = function(stooge) { return stooge.name; };
_.uniq(list, false, iterator)
  
  函数将对这个指定的对象里的name属性进行迭代排重
*/
    if (iteratee != null) iteratee = cb(iteratee, context);
    // 保存最终结果
    var result = [];
    // 保存上一个元素
    var seen = [];
    for (var i = 0, length = getLength(array); i < length; i++) {
      var value = array[i],
         // 如果指定了迭代函数则对每一个数组里的值进行迭代
          computed = iteratee ? iteratee(value, i, array) : value;
      // 如果已经排序了
      if (isSorted) {
        // 将迭代后的值和保存的上一个变量进行对比看是不是重复的
        if (!i || seen !== computed) result.push(value);
        seen = computed;
     // 如果存在自定义的迭代
      } else if (iteratee) {
        // 不然直接在整个seen[]数组中找这个元素是不是存在,不存在就保存到结果里，这样能保证每一个元素都是唯一的
        if (!_.contains(seen, computed)) {
          seen.push(computed);
          result.push(value);
        }
     // 不存在自定义的迭代那么直接对整个传入的数组的元素来判断是否唯一
      } else if (!_.contains(result, value)) {
        result.push(value);
      }
    }
    return result;
  };
  
```

我们看到里面有一个`_.contains`，这个就是`集合`里的一个函数我们可以看到它是这么写的：

```
 // 最终是通过indexOf来判断这个元素是不是在数组或者对象里面。
 // Underscore自己实现了一个createIndexFinder来创建indexOf和lastIndexOf，非常的长，这是为了针对不存在ES5原生函数IndexOf和lastIndexOf的处理方式，但是我们基本就是依赖浏览器来开发的,而且我在node里也发现IndexOf是存在的，那么我们就不需要考虑用它的方案直接调用对象的indexOf即可。
 
 _.contains  = function(obj, item, fromIndex, guard) {
    // 如果是类数组就把整个类数组的值放进一个数组里，然后用数组的IndexOf方法来判断是否包含
    if (!isArrayLike(obj)) obj = _.values(obj);
    if (typeof fromIndex != 'number' || guard) fromIndex = 0;
    // 原生
    // return _.indexOf(obj, item, fromIndex) >= 0;
    // 改造后
    return obj.indexOf(item, fromIndex) >= 0;
  };

  // 将一个对象的所有 values 值放入数组中
  _.values = function(obj) {
    var keys = _.keys(obj);
    var length = keys.length;
    var values = Array(length);
    for (var i = 0; i < length; i++) {
      values[i] = obj[keys[i]];
    }
    return values;
  };

```

## 结尾

大概也是把自己想要的部分给过了一遍，然而`Underscore`其实还有很多部分在本文并没有提及，因此感兴趣的可以自己按照我上面的方式去阅读源码。当然我在之后的日子里会慢慢将这个系列给填完。

下面是关于本文参考的资料

[Underscore 1.8.3中文文档](http://www.css88.com/doc/underscore/#contains)


[underscore.js 源码解读](https://github.com/hanzichi/underscore-analysis)
	
推荐看源码分析中的产物。



