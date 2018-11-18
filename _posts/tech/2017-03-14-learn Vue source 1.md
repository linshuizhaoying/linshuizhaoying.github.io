---
layout: post
title: Vue源码学习(从入门到放弃到总结)-Observe
category: 技术
tags: [学习,开发,前端,总结,2018,Vue源码学习,Source Code,面试]
keywords: 总结,2018,Vue源码学习,source code,前端
description: 
---

## 前言

对于源码学习我实话说以前还是挺怵的，一个是代码量很大，逻辑复杂，另一个是怕自己解读不到位然后造成错误观念。
但是很久之前开始尝试过读一些库的源码，比如`underscore`,`jquery`。虽然没有全部细细读完，但是收获还是挺大的。
最近面试的时候被面试官问到你读过Vue源码吗？
当时回答肯定是没有，虽然vue源码阅读计划被我年初已经列到行动表里，但是总觉得自己还差点什么，一直没动。
不过后来我仔细反思了一下，我想我缺少的就是JUST DO IT.

## 正文 

### 自己不会读我会找会的带我读

当然一开始也不会像以前那样读工具库源码那样直接一股脑钻到源码中，像这种框架型的我个人认为一定要把握住主干。但是自己找不到主干怎么办，我的方法是先跟着读过源码的前辈们走一遍他们读过的路，然后一步步印证。

这里我选择[梁少峰的Vue源码学习系列](https://github.com/youngwind/blog)

一路跟着作者先把准备工作做好，选择vue早期某一版本，然后在`src/internal/init.js`里面加上

```

var Observer = require('../observe/observer')
var _ = require('../util')

```

然后`npm i grunt -g`

因为早期是用grunt打包，我们想要调试肯定要自己打包一份出来。
作者的思路是:

```

从成熟框架的早期源码开始看起，从作者的第一个commit开始看起，然后逐个的往前翻。这样一开始的代码量不多，多看几遍还是可以理解的。而且在这个过程中，就像电影回放一样，我们可以看到作者先写什么，后写什么，在哪些地方进行了什么样的改良，其中又不小心引入了什么bug，等等

```

我们跟随git的commit来看下，最早实现的就是`observer object`

当然我现在是连最初版本也不知道该如何下手的，因此继续跟随作者先实现一个`obsever`

Vue的双向绑定是基于`Object.defineProperty`这个我是了解的，但是具体实现我是没正式接触过。看下作者给的代码.

```

// 观察者构造函数
function Observer(data) {
    this.data = data;
    this.walk(data)
}

let p = Observer.prototype;

// 此函数用于深层次遍历对象的各个属性
// 采用的是递归的思路
// 因为我们要为对象的每一个属性绑定getter和setter
p.walk = function (obj) {
    let val;
    for (let key in obj) {
        // 这里为什么要用hasOwnProperty进行过滤呢？
        // 因为for...in 循环会把对象原型链上的所有可枚举属性都循环出来
        // 而我们想要的仅仅是这个对象本身拥有的属性，所以要这么做。
        if (obj.hasOwnProperty(key)) {
            val = obj[key];

            // 这里进行判断，如果还没有遍历到最底层，继续new Observer
            if (typeof val === 'object') {
                new Observer(val);
            }

            this.convert(key, val);
        }
    }
};

p.convert = function (key, val) {
    Object.defineProperty(this.data, key, {
        enumerable: true,
        configurable: true,
        get: function () {
            console.log('你访问了' + key);
            return val
        },
        set: function (newVal) {
            console.log('你设置了' + key);
            console.log('新的' + key + ' = ' + newVal)
            if (newVal === val) return;
            val = newVal
        }
    })
};

let data = {
    user: {
        name: "liangshaofeng",
        age: "24"
    },
    address: {
        city: "beijing"
    }
};

let app = new Observer(data);

```

![imgn](http://img.haoqiao.me/vuesource1.png)

我们回顾下上面代码做了什么

```

1.创建一个Observer函数,它作为构造函数
2.定义一个对象指向Observer的原型。
3.给对象添加两个函数walk和convert,这样相当于直接在原型上加了两个函数。
4.walk函数来遍历处理对象，如果深层次的递归处理。这里有意思的是new Observer(val),我们知道new关键词的处理过程原理，因此也就明白这里用new给每一个对象属性添加监听回调的意思了。
5.convert就是利用defineProperty来给属性修改set和get函数的内容。关于Object.defineProperty()可以去mdn上查阅更详细的用法。

```

这段代码简单的告诉我们怎么对一个深层次对象进行属性监听。但这似乎还不够。我们继续跟着作者的思路走。

作者评价刚才那段代码有两个缺陷:

```

1.无法处理数组
2.如果给原本监听的对象新增属性,重新设置的属性不带set和get回调。

```

但是很奇怪的是作者之后的文章有一种奇怪的风向，就是不连贯，比如处理数据就讲到了包装再处理，第二点也没提到。不过没关系我们可以借鉴别的文章的思路来。

顺着作者github issue里面有人提出作者当时的文章有问题并举出大片例子来证明时我就预感到跟着他的github走肯定可以找到我所需要的。果然他也分析过vue源码而且还要中文注释版
[Ma63d的vue源码注解](https://github.com/Ma63d/vue-analysis)

这里我们跟着这份资料走，首先去找到处理数组的部分

```

function touch (obj) {
    if (typeof obj === 'object')
      if (Array.isArray(obj)) {
        for (let i = 0,l = obj.length; i < l; i++) {
          touch(obj[i])
        }
      } else {
        let keys = Object.keys(obj)
        for (let key of keys) touch(obj[key])
      }
    console.log(obj)
  }

```

遇到普通数据属性，直接处理，遇到对象，遍历属性之后递归进去处理属性，遇到数组，递归进去处理数组元素。console.log这里作为convert的替代。
然而当前依旧没有处理数组如果更新变动了该如何解决。

作者提到

```

Vue的方法是，改写数组的push、pop等8个方法，让他们在执行之后通知我数组更新了

改进之后我就不需要对数组元素进行响应式处理，只是遇到数组的时候把数组的方法变异即可。于是在用户使用数组的push、pop等方法会改变数组本身的方法时，可以监听到数组变动

vue为数组提供了$set和$remove，方便我们可以通过下标去响应式的改动数组元素

```

看到上面的可能迷迷糊糊就会混过去，认为好像就应该这么解决。庆幸的这位作者还给出了为什么这么做，这么做的考虑的原因：

```

如果遇到数组data中的数组实例增加了一些“变异”的push、pop等方法，这些方法会在数组原本的push、pop方法执行后发出消息，表明发生了改动。听起来这好像可以用继承的方式实现: 继承数组然后在这个子类的原型上附加上变异的方法。

但是你需要知道的是在es5及更低版本的js里，无法完美继承数组，主要原因是Array.call(this)时，Array根本不是像一般的构造函数那样对你传进去this进行改造，而是直接返回一个新的数组。所以一般的继承方式就没法实现了


```

作者还给出一篇英文文章[es5及更低版本的js里，无法完美继承数组](http://perfectionkills.com/how-ecmascript-5-still-does-not-allow-to-subclass-an-array/)

将这篇文章好好理解一下肯定对我们对理解如何observer数组有帮助。这里我就简单把这篇文章的重点提出来.


### es5及更低版本的js里，无法完美继承数组

首先明确一点,ECMAScript 5依旧无法完美继承一个数组。

虽然通过一些手段我们可以继承一个数组，但是一些基本原则造成我们无法真正的继承它。

举个例子：

我们需要一个数组继承是这样的

```


var sub = new SubArray(1, 2, 3);
sub; // [1, 2, 3]

sub.length; // 3
sub[1]; // 2

sub.push(4);
sub; // [1, 2, 3, 4]

```
把上面的代码转成人话来说就是

我们需要一个`SubArray`构造函数创建一个`sub`对象,它拥有Length属性，可以通过下标访问元素，而且继承了`Array.prototype.*`所有原生方法。而且最重要的是`sub`的继承原型是`SubArray`而不是`Array`

实现这个继承需要遵循两个基本原则

```

1.不能污染全局Array

比如下面的代码是不行的


Array.prototype.last = function () {
  return this[this.length-1];
};
// ...
[1, 2, 3].last(); // 3

虽然对Array的prototype修改是最效率的，但是对Prototype的污染也是最大的。

因此用构造函数而不是原生Array将会避免冲突，而且是最好的方法。因为你扩展SubArray.prototype而不是原生Array.prototype方法将会使依赖于原生的第三方代码依旧安全。

2.拥有继承数组的特性-构造数据结构
数组继承需要能够很自然的创建数据结构类似：Stack,List,Queue等等


```

接下来我们可以看到传统朴素的方法

众所周知的`clone()函数`

```

function clone(obj) {
  function F() { }
  F.prototype = obj;
  return new F();
}

```

这在高程中原型式继承可以找到。

之后就是利用这个函数继承

```


function Child() { }
Child.prototype = clone(Parent.prototype);

```

然后我们可以获得原型链如下：

```

new Child()
    |
    | [[Prototype]]
    |
    v
Child.prototype
    |
    | [[Prototype]]
    |
    v
Parent.prototype
    |
    | [[Prototype]]
    |
    v
Object.prototype
    |
    | [[Prototype]]
    |
    v
   null

```

接下来我们用这个方法来继承数组

```

function SubArray() {
  //将接受的参数push到实例中
  this.push.apply(this, arguments);
}
SubArray.prototype = clone(Array.prototype);

var sub = new SubArray(1, 2, 3);

```

乍一看可能好像没有多大问题，但是凡事就怕有个比较，我们用原生的Array作为构造函数和SubArray构造函数进行对比:


```
var arr = new Array(2, 3, 4);
var sub = new SubArray(2, 3, 4);

console.log(arr + " " + arr.length)  //2,3,4 3

console.log(sub + " " +sub.length)
//2,3,4 3

arr.length = 2;
sub.length = 2;

console.log(arr)
// [2, 3]


console.log(sub)

// 2,3,4 length:2




arr[10] = 'foo';



sub[10] = 'foo';


console.log(arr + " " + arr.length)
//2,3,,,,,,,,,foo 11

console.log(sub + " " +sub.length)

//2,3 2 实际上sub[2]依旧有值为4

```

直接看chrome下结果显示

![imgn](http://img.haoqiao.me/vuesource2.png)

在sub中Length和数组下标已经无法一一对应了。

我们需要明白问题出在哪里。

首先在数组中length和它处理下标属性是息息相关的，比如添加一个删除一个元素都会自动引起length的变化。如果你把length设置为0，那么数组会自动清空元素。

很明显SubArray虽然继承自数组，但是没有把数组的特性给继承下来。SubArray实例只是一个普通的Object对象。和我们通过字面量创建一个对象没啥区别。

但明明我们用clone去继承了Array对象为什么最终获得的是Object对象呢？

我们回到上面代码中

`var sub = new SubArray(2, 3, 4);`

当我们用`new`操作符来创建一个对象，会调用构造函数，SubArray.constructor是function,而默认function的构造函数是Object。于是最终返回的是Object对象作为返回值。

来个精确的类型返回就是

```
Object.prototype.toString.call(arr)
"[object Array]"
Object.prototype.toString.call(sub)
"[object Object]"

```

当你试图

```

function SubArray() {
  //将接受的参数push到实例中
  this.push.apply(this, arguments);
}

```

Array.call(this)时，Array根本不是像一般的构造函数那样对你传进去this进行改造，而是直接返回一个新的数组.

所以从根本上堵死了你试图完美继承数组的想法。

我们为什么试图想要正确继承Length这个特性呢，举个例子就是当你对一个数组用`Push`方法时，push方法是要自动找到Length，在正确长度后再增加元素。当你length这个特性不准确时，添加位置就是错误的。而利用Length这个特性来操作的原生方法如`Join`,`concat`更是依赖性强。

曾经有很多人去试图解决这个问题，他们提出各种各样的方法，如创建一个stack方法，然而这个方法对length操作并没有同步到数组。
还有人脑洞大开，创建一个iframe，从iframe里面借出一个Array.但是不同iframe里面Array对象的继承关系是不存在的。因此即使借出来也不是本地Array的实例。原型链都是另一条的。

ECMAScript 5有个属性访问器来帮助我们子类化数组。
具体思路是在makeSubArray中通过的mthods.length的set和get处理，来实现原生length的效果。

[makeSubArray代码](https://github.com/kangax/array_subclassing/blob/master/subarray.js)

### 回到Observe数组中

通过上面的学习我们知道了为什么Observe数组为什么这么困难。

跟着作者思路走:

```

如果当前浏览器里存在__proto__这个非标准属性的话（大部分都有），那又可以有方法继承，就是创建一个继承自Array.prototype的Object: Object.create(Array.prototype)，在这个继承了数组原生方法的对象上添加方法或者覆盖原有方法，然后创建一个数组，把这个数组的__proto__指向这个对象，这样这个数组的响应式的length属性又得以保留，又获得了新的方法，而且无侵入，不会改变本来的数组原型。


Vue中先判断__proto__能不能用(hasProto)，如果能用，则把那个一个继承自Array.prototype的并且添加了变异方法的Object (arrayMethods)，设置为当前数组的__proto__，完成改造，如果__proto__不能用，那么就只能遍历arrayMethods就一个个的把变异方法def到数组实例上面去，这种方法效率不高，所以优先使用改造__proto__的那个方法。


```

然后在阅读源码`observe`里`array-augmentations`文件里的具体实现:

```

var _ = require('../util')
var slice = [].slice
// 为了避免污染全局Array
var arrayAugmentations = Object.create(Array.prototype)


// 把常用的函数整理出来
;[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
.forEach(function (method) {
  // cache original method
  var original = Array.prototype[method]
  // define wrapped method
  //将其一个个添加到arrayAugmentations
  // define等同于Object.defineProperty的简写,它将原生的方法给绑到新的arrayAugmentations对象上，并且监听可能引起数组变化的几个方法
  
   _.define(arrayAugmentations, method, function () {
    
    var args = slice.call(arguments)
    //调用原生方法
    var result = original.apply(this, args)
    var ob = this.$observer
    var inserted, removed, index

// 监听所有可能引起数组变化的方法，这里和后面版本只监听引起数组增加的方法不一致，这里先标记一下。

    switch (method) {
      case 'push':
        inserted = args
        index = this.length - args.length
        break
      case 'unshift':
        inserted = args
        index = 0
        break
      case 'pop':
        removed = [result]
        index = this.length
        break
      case 'shift':
        removed = [result]
        index = 0
        break
      case 'splice':
        inserted = args.slice(2)
        removed = result
        index = args[0]
        break
    }

    // 对新插入的元素添加observe
    if (inserted) ob.link(inserted, index)
    // 对删除的元素取消observe
    if (removed) ob.unlink(removed)

    // update indices
    if (method !== 'push' && method !== 'pop') {
      ob.updateIndices()
    }

    // 手工去触发Length改变方法
    if (inserted || removed) {
      ob.notify('set', 'length', this.length)
    }

    // empty path, value is the Array itself
    ob.notify('mutate', '', this, {
      method   : method,
      args     : args,
      result   : result,
      index    : index,
      inserted : inserted || [],
      removed  : removed || []
    })

    return result
  })
})

/**
 * Swap the element at the given index with a new value
 * and emits corresponding event.
 *
 * @param {Number} index
 * @param {*} val
 * @return {*} - replaced element
 */

_.define(arrayAugmentations, '$set', function (index, val) {
  if (index >= this.length) {
    this.length = index + 1
  }
  return this.splice(index, 1, val)[0]
})

/**
 * Convenience method to remove the element at given index.
 *
 * @param {Number} index
 * @param {*} val
 */

_.define(arrayAugmentations, '$remove', function (index) {
  if (typeof index !== 'number') {
    index = this.indexOf(index)
  }
  if (index > -1) {
    return this.splice(index, 1)[0]
  }
})

module.exports = arrayAugmentations



```

这里提出了一个问题，早期Vue里面数组所有方法都被监听，后来版本里面只监听了数组增加这块，后来我邮件问了下新版本源码注释的作者,得到了答案

```

删除，就直接删除，然后他们被pop，被unshift掉，如果是一个普通数组，你pop了元素，这个元素如果是个对象且没有任何东西引用他的话，他在下一次垃圾收集时就会被js的垃圾收集给干掉了，你什么都不用做对吧，想想你平时执行数组pop的时候有额外的操作吗？肯定是没有的。

现在存在的问题时，比如我v-for一个数组，那每个元素都会创建一个watcher，监听数组具体的元素的值。
也就是说这个watcher还订阅着数组元素的，watcher保存了数组元素的dep 的引用。所以watcher的afterGet中就会去判断哪些是新的依赖，哪些是旧的依赖，旧的依赖比如你刚刚pop出去的这个数组，我就需要watcher不再引用他。不引用他了之后，他就没有被Vue引用了，下一轮垃圾清除时就会把他干掉了。

```

这样就懂了，旧版本中是手工去执行数组元素删除绑定，后面版本是旧元素删除引起时对应的依赖退订在watcher的afterGer里统一搞的.


到这里大致搞懂了Observe最基础版,接下来一篇是watch。


