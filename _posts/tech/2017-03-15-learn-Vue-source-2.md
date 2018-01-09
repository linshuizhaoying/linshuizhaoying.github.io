---
layout: post
title: Vue源码学习(从入门到放弃到总结)-Watcher到MvvM
category: 技术
tags: [学习,开发,前端,总结,2018,Vue源码学习,Source Code,面试]
keywords: 总结,2018,Vue源码学习,source code,前端
description: 
---

## 前言
之前花了一天学习`Observe`，但是感觉连大门也没迈入。但是学习就得坚持，坚持坚持着说不定就成了~

## 正文

### 谈源码之前我们谈谈发布者-观察者模式

在接下去学习之前我们得先掌握一个基础知识点，那就是设计模式里的发布者-观察者模式。

```

观察者模式/发布-订阅模式/消息机制
定义了一种依赖关系，解决主体对象与观察者之间功能的耦合。解决两个互相依赖的对象，使其依赖于观察者的消息机制。

```

摘抄一个`<<javascript设计模式>>`书上的例子

```

'use strict';

// 背景：评论+消息通知

/*
 * 观察者模式
 *
 * 将观察者放在闭包中，当页面加载就立即执行
 */
var Observer = (function() {
  // 防止消息队列暴漏而被篡改，故将消息容器作为静态私有变量保存。
  var __message = {};

  return {
    // 订阅
    subscribe: function(type, fn) {
      // 如果消息不存在则创建一个消息类型
      if (!__message[type]) {
        __message[type] = [fn];
      } else {
        // 将动作方法推送到消息对应的动作执行序列中
        __message[type].push(fn);
      }
    },

    // 取消订阅
    unsubscribe: function(type, fn) {
      if (!__message[type] || Object.prototype.toString.call(__message[type]) !== '[object Array]') return;

      for (var i = __message[type].length - 1; i >= 0; i--) {
        __message[type][i] === fn && __message[type].splice(i, 1);
      };
    },

    // 发布信息
    publish: function(type, args) {
      if (!__message[type]) return;

      // 定义消息信息
      var events = {
        type: type,
        args: args
      };

      // 执行注册的消息所对应的所有动作序列
      for (var i = 0; i < __message[type].length; i++) {
        __message[type][i].call(this, events);
      }
    }
  }
})();


/*
 * 拉出来溜溜
 */

// 订阅
Observer.subscribe('test', function(e) {
  console.log(e);
});

// 发布
Observer.publish('test', {msg: '传递参数'});

```

整个过程是Observer维护一个_message队列，然后订阅者订阅内容，当发布消息的时候循环执行_message队列里的动作。

### 读几个简单的mvvm的代码

```

执行 new Vue() 时，Vue 就进入了初始化阶段，一方面Vue 会遍历 data 选项中的属性，并用 Object.defineProperty 将它们转为 getter/setter，实现数据变化监听功能；另一方面，Vue 的指令编译器Compile 对元素节点的指令进行扫描和解析，初始化视图，并订阅Watcher 来更新视图， 此时Wather 会将自己添加到消息订阅器中(Dep),初始化完毕。

当数据发生变化时，Observer 中的 setter 方法被触发，setter 会立即调用Dep.notify()，Dep 开始遍历所有的订阅者，并调用订阅者的 update 方法，订阅者收到通知后对视图进行相应的更新。

```

从上面可以看到watcher只是作为一个订阅器的作用。如果想自己实现一个双向数据绑定的demo,还需要一个`compile`和对整个代码进行组织。

当然以现在的水平即使是模仿写出一个也是很吃力的，而且勉强写出来的质量肯定有问题。因此我打算用别的方式来学习一个。读几份不同mvvm实现源码来了解一下。

![imgn](http://haoqiao.qiniudn.com/mvvm-detail1.png)

先手动跟着画一下mvvm整个流程图,初步了解一下构筑流程。这里我先读的是`DMQ`的mvvm实现，他这份更接近vue最初的双向绑定。

先从他最终demo里的代码来逆推回去

```
html:
<div id="mvvm-app">
    <input type="text" v-model="someStr">
    <input type="text" v-model="child.someStr">
    <p v-class="className" class="abc">
        {{someStr}}
        <span v-text="child.someStr"></span>
    </p>
    <p v-html="child.htmlStr"></p>
    <button v-on:click="clickBtn">change model</button>
</div>

javascript：

 var vm = new MVVM({
        el: '#mvvm-app',
        data: {
            someStr: 'hello ',
            className: 'btn',
            htmlStr: '<span style="color: #f00;">red</span>',
            child: {
                someStr: 'World !'
            }
        },

        methods: {
            clickBtn: function(e) {
                var randomStrArr = ['childOne', 'childTwo', 'childThree'];
                this.child.someStr = randomStrArr[parseInt(Math.random() * 3)];
            }
        }
    });

    vm.$watch('child.someStr', function() {
        console.log(arguments);
        

```

从上面这段代码我们可以看到MVVM作为构造函数接受`el`,`data`,`methods`三个参数。而且还能通过指定`$watch`函数来监听data里的`child`变动。

来看下js目录的文件

```
├── compile.js
├── mvvm.js
├── observer.js
└── watcher.js

```

和刚刚的流程图一一对应，我们从mvvm入口处来看。先不管属性代理这块，我们看下代码

```
function MVVM(options) {
    this.$options = options;
    var data = this._data = this.$options.data;
    observe(data, this);

    this.$compile = new Compile(options.el || document.body, this)
}

MVVM.prototype = {
    $watch: function(key, cb, options) {
        new Watcher(this, key, cb);
    }
    
 }

```

那么整个流程就是

```

1.接受options参数，然后用Observe去监听
2.调用Compile构造函数对传入的元素进行指令解析

```

这就回到了我们之前学过的`Observer`,但我们需要学习的是怎么组织代码。并不是功能实现就行了。我们来看看他的`Observer.js`是怎么写的：

```
//构造函数Observer
function Observer(data) {
    this.data = data;
    this.walk(data);
}
// 添加私有方法
Observer.prototype = {
    walk: function(data) {
        var me = this; //保存上下文
        Object.keys(data).forEach(function(key) {//对对象所有属性进行遍历
            me.convert(key, data[key]);
        });
    },
    convert: function(key, val) {
        this.defineReactive(this.data, key, val);//利用defineReactive进行对象属性的监听
    },

    defineReactive: function(data, key, val) {
        var dep = new Dep(); //执行一次Dep,Dep内部id++
        var childObj = observe(val);//判断对象内部还有没有内嵌对象，如果有的话，递归监听

        Object.defineProperty(data, key, {
            enumerable: true, // 可枚举
            configurable: false, // 不能再define
            get: function() {
                if (Dep.target) {
                    dep.depend();
                }
                return val;
            },
            set: function(newVal) {
                if (newVal === val) {
                    return;
                }
                val = newVal;
                // 新的值是object的话，进行监听
                childObj = observe(newVal);
                // 通知订阅者
                dep.notify();
            }
        });
    }
};

function observe(value, vm) {
    if (!value || typeof value !== 'object') {
        return;
    }

    return new Observer(value);
};


var uid = 0;

function Dep() {
    this.id = uid++;
    this.subs = [];
}

Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },

    depend: function() {
        Dep.target.addDep(this);
    },

    removeSub: function(sub) {
        var index = this.subs.indexOf(sub);
        if (index != -1) {
            this.subs.splice(index, 1);
        }
    },

    notify: function() {
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};

Dep.target = null;

```

这份`Observer`没有监听数组的变动，只对对象进行了监听。遍历传入的对象属性，然后递归遍历的将其全部用`Obeject.defineProperty()来监听属性变动`。

在仔细分析监听之前我们可以看到这么一段代码

```

var uid = 0;

// 依赖数组,每运行一次构造函数id都自增
function Dep() {
    this.id = uid++;
    this.subs = [];
}

Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },

    depend: function() {
        Dep.target.addDep(this);
    },

    removeSub: function(sub) {
        var index = this.subs.indexOf(sub);
        if (index != -1) {
            this.subs.splice(index, 1);
        }
    },

    notify: function() {
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};

// 由于需要在闭包内添加watcher，所以通过Dep定义一个全局target属性，暂存watcher, 添加完移除
Dep.target = null;

```

上面的代码就是维护了一个依赖队列。`notify()`每运行一次都遍历全部队列执行`update()`

上面我们看到了订阅器，但是订阅者在哪里处理的还不知道。我们来看`watcher.js`

```

function Watcher(vm, exp, cb) {
    this.cb = cb;
    this.vm = vm;
    this.exp = exp;
    this.depIds = {};
    // 此处为了触发属性的getter，从而在dep添加自己，结合Observer更易理解
    this.value = this.get();
}

Watcher.prototype = {
    update: function() {
        this.run();// 属性值变化收到通知
        console.log("属性值变化")
    },
    run: function() {
        var value = this.get(); // 取到最新值
        var oldVal = this.value;
        if (value !== oldVal) {
            this.value = value;
            this.cb.call(this.vm, value, oldVal);// 执行Compile中绑定的回调，更新视图
        }
    },
    addDep: function(dep) {
        // 1. 每次调用run()的时候会触发相应属性的getter
        // getter里面会触发dep.depend()，继而触发这里的addDep
        // 2. 假如相应属性的dep.id已经在当前watcher的depIds里，说明不是一个新的属性，仅仅是改变了其值而已
        // 则不需要将当前watcher添加到该属性的dep里
        // 3. 假如相应属性是新的属性，则将当前watcher添加到新属性的dep里
        // 如通过 vm.child = {name: 'a'} 改变了 child.name 的值，child.name 就是个新属性
        // 则需要将当前watcher(child.name)加入到新的 child.name 的dep里
        // 因为此时 child.name 是个新值，之前的 setter、dep 都已经失效，如果不把 watcher 加入到新的 child.name 的dep中
        // 通过 child.name = xxx 赋值的时候，对应的 watcher 就收不到通知，等于失效了
        // 4. 每个子属性的watcher在添加到子属性的dep的同时，也会添加到父属性的dep
        // 监听子属性的同时监听父属性的变更，这样，父属性改变时，子属性的watcher也能收到通知进行update
        // 这一步是在 this.get() --> this.getVMVal() 里面完成，forEach时会从父级开始取值，间接调用了它的getter
        // 触发了addDep(), 在整个forEach过程，当前wacher都会加入到每个父级过程属性的dep
        // 例如：当前watcher的是'child.child.name', 那么child, child.child, child.child.name这三个属性的dep都会加入当前watcher
        if (!this.depIds.hasOwnProperty(dep.id)) {
            dep.addSub(this);
            this.depIds[dep.id] = dep;
            console.log("id变化，新增监听对象")
        }
    },
    get: function() {
        Dep.target = this;// 将当前订阅者指向自己
        var value = this.getVMVal();// 触发getter，添加自己到属性订阅器中
        Dep.target = null;// 添加完毕，重置
        return value;
    },

    getVMVal: function() {
        var exp = this.exp.split('.');
        var val = this.vm._data;
        console.log("此时getVMVal中exp值为 " + exp  )
        console.log("此时getVMVal中val值为 " + val  )
        exp.forEach(function(k) {
            val = val[k];
        });
         console.log("此时getVMVal中遍历后的val值为 ")
         console.log(val)
        return val;
    }
};

```

上面注释中细节讲了不少，我们先大概了解一下，然后来看`compile.js`的源码，这里先跳过是因为我们需要了解`watch`具体是在哪被调用的，这样我们才能把整个流程思路理清。

接下来我们看`compile.js`

```

function Compile(el, vm) {
    // vm是传入的data和methods的集合
    this.$vm = vm; 
    // 指定绑定节点el
    this.$el = this.isElementNode(el) ? el : document.querySelector(el);
    
    //如果指定节点存在
    if (this.$el) {
        //因为遍历解析的过程有多次操作dom节点，为提高性能和效率，
        //会先将跟节点el转换成文档碎片fragment进行解析编译操作，
        //解析完成，再将fragment添加回原来的真实dom节点中
        this.$fragment = this.node2Fragment(this.$el);
        this.init();
        this.$el.appendChild(this.$fragment);
    }
}

Compile.prototype = {
    node2Fragment: function(el) {
        var fragment = document.createDocumentFragment(),
            child;

        // 将原生节点拷贝到fragment
        while (child = el.firstChild) {
            fragment.appendChild(child);
        }

        return fragment;
    },

    init: function() {
        // 对节点进行解析
        this.compileElement(this.$fragment);
    },
    //compileElement方法将遍历所有节点及其子节点，
    // 进行扫描解析编译，调用对应的指令渲染函数进行数据渲染，
    // 并调用对应的指令更新函数进行绑定
    compileElement: function(el) {
        var childNodes = el.childNodes,
            me = this;
        //[].slice.call 主要用于将伪数组转为真正的数组，这样才能调用数组才有的forEach方法
        [].slice.call(childNodes).forEach(function(node) {
            var text = node.textContent;
            var reg = /\{\{(.*)\}\}/;// 表达式文本,从{{val}}中把val提取出来

            // 如果是元素节点
            if (me.isElementNode(node)) {
                me.compile(node);
                console.log("正在编译元素节点:" + node)
            // 如果是文本节点
            } else if (me.isTextNode(node) && reg.test(text)) {
                me.compileText(node, RegExp.$1);
                console.log("正在编译文本节点:" + node)
            }
            // 遍历编译子节点
            if (node.childNodes && node.childNodes.length) {
                me.compileElement(node);
                console.log("发现存在子节点，准备遍历" + node + "的子节点:" + node.childNodes)
            }
        });
    },

    compile: function(node) {
        var nodeAttrs = node.attributes,
            me = this;
        // 对节点属性进行遍历
        [].slice.call(nodeAttrs).forEach(function(attr) {
            var attrName = attr.name;
            //规定：指令以 v-xxx 命名
            if (me.isDirective(attrName)) {
                var exp = attr.value;
                var dir = attrName.substring(2);
                // 事件指令, 如 v-on:click
                if (me.isEventDirective(dir)) {
                    compileUtil.eventHandler(node, me.$vm, exp, dir);
                    // 普通指令
                } else {
                    compileUtil[dir] && compileUtil[dir](node, me.$vm, exp);
                }

                node.removeAttribute(attrName);
            }
        });
    },

    compileText: function(node, exp) {
        compileUtil.text(node, this.$vm, exp);
    },
    // 判断v-开头的编译指令
    isDirective: function(attr) {
        return attr.indexOf('v-') == 0;
    },

    isEventDirective: function(dir) {
        return dir.indexOf('on') === 0;
    },

    isElementNode: function(node) {
        return node.nodeType == 1;
    },

    isTextNode: function(node) {
        return node.nodeType == 3;
    }
};

// 指令处理集合
var compileUtil = {
    text: function(node, vm, exp) {
        console.log("绑定text类型")
        this.bind(node, vm, exp, 'text');
    },

    html: function(node, vm, exp) {
        console.log("绑定html类型")
        this.bind(node, vm, exp, 'html');
    },

    model: function(node, vm, exp) {
        console.log("绑定model类型")
        this.bind(node, vm, exp, 'model');

        var me = this,
            val = this._getVMVal(vm, exp);
        node.addEventListener('input', function(e) {
            var newValue = e.target.value;
            if (val === newValue) {
                return;
            }
            // 手动设置值
            console.log("正在手工setVMval:" + exp)
            me._setVMVal(vm, exp, newValue);
            val = newValue;

        });
    },

    class: function(node, vm, exp) {
        console.log("绑定class类型")
        this.bind(node, vm, exp, 'class');
    },

    bind: function(node, vm, exp, dir) {
        var updaterFn = updater[dir + 'Updater'];
        // 第一次初始化视图
        updaterFn && updaterFn(node, this._getVMVal(vm, exp));
        // 实例化订阅者，此操作会在对应的属性消息订阅器中添加了该订阅者watcher
        console.log("准备绑定"+ node)
        new Watcher(vm, exp, function(value, oldValue) {
            updaterFn && updaterFn(node, value, oldValue);
        });
    },

    // 事件处理
    eventHandler: function(node, vm, exp, dir) {
        var eventType = dir.split(':')[1],
            fn = vm.$options.methods && vm.$options.methods[exp];

        if (eventType && fn) {
            console.log("监听"+eventType+fn)
            node.addEventListener(eventType, fn.bind(vm), false);
        }
    },

    _getVMVal: function(vm, exp) {
        var val = vm._data;
        exp = exp.split('.');
        exp.forEach(function(k) {
            val = val[k];
        });
        return val;
    },

    _setVMVal: function(vm, exp, value) {
        var val = vm._data;
        exp = exp.split('.');
        exp.forEach(function(k, i) {
            // 非最后一个key，更新val的值
            if (i < exp.length - 1) {
                val = val[k];
            } else {
                val[k] = value;
            }
        });
    }
};


var updater = {
    textUpdater: function(node, value) {
        console.log("text内容更新")
        node.textContent = typeof value == 'undefined' ? '' : value;
    },

    htmlUpdater: function(node, value) {
        console.log("html内容更新")
        node.innerHTML = typeof value == 'undefined' ? '' : value;
    },

    classUpdater: function(node, value, oldValue) {
        console.log("class内容更新")
        var className = node.className;
        className = className.replace(oldValue, '').replace(/\s$/, '');

        var space = className && String(value) ? ' ' : '';

        node.className = className + space + value;
    },

    modelUpdater: function(node, value, oldValue) {
        console.log("model内容更新")
        node.value = typeof value == 'undefined' ? '' : value;
    }
};

```
上面代码我加了点注释方便理解。

这里通过递归遍历保证了每个节点及子节点都会解析编译到，包括了{{}}表达式声明的文本节点。指令的声明规定是通过特定前缀的节点属性来标记，如<span v-text="content" other-attr中v-text便是指令，而other-attr不是指令，只是普通的属性。 监听数据、绑定更新函数的处理是在compileUtil.bind()这个方法中，通过new Watcher()添加回调来接收数据变化的通知。

数据对象是options.data，每次需要更新视图，则必须通过var vm = new MVVM({data:{name: 'kindeng'}}); vm._data.name = 'dmq';这样的方式来改变数据。

显然不符合我们一开始的期望，我们所期望的调用方式应该是这样的： var vm = new MVVM({data: {name: 'kindeng'}}); vm.name = 'dmq';

所以这里需要给MVVM实例添加一个属性代理的方法，使访问vm的属性代理为访问vm._data的属性，改造后的代码如下：

```

function MVVM(options) {
    this.$options = options;
    var data = this._data = this.$options.data, me = this;
    // 属性代理，实现 vm.xxx -> vm._data.xxx
    Object.keys(data).forEach(function(key) {
        me._proxy(key);
    });
    observe(data, this);
    this.$compile = new Compile(options.el || document.body, this)
}

MVVM.prototype = {
	_proxy: function(key) {
		var me = this;
        Object.defineProperty(me, key, {
            configurable: false,
            enumerable: true,
            get: function proxyGetter() {
                return me._data[key];
            },
            set: function proxySetter(newVal) {
                me._data[key] = newVal;
            }
        });
	}
};


```

这里作者的整个完成mvvm思路就成型了。

但我们的分析才刚刚开始，之前贴的代码中我加了很多`console.log`这是为了在这一连串调用中，我们需要知道数据是怎么被处理，绑定的。

而且为了验证我将demo改成如下内容：

```

<div id="mvvm-app">
    <input type="text" v-model="someStr">
    <input type="text" v-model="child.someStr">
    <p v-class="className" class="abc">
        {{someStr}}
        <span v-text="child.someStr"></span>

    </p>
     <input type="text" v-model="child.test.linshui">
    <p>
       {{someStr}}
        <span v-text="child.test.linshui"></span>
    </p>
    <p v-html="child.htmlStr"></p>
    <button v-on:click="clickBtn">change model</button>
</div>


  var vm = new MVVM({
        el: '#mvvm-app',
        data: {
            someStr: 'hello ',
            className: 'btn',
            htmlStr: '<span style="color: #f00;">red</span>',
            child: {
                someStr: 'World !',
                test:{
                    linshui:"Zhaoying"
                }
            }
        },

        methods: {
            clickBtn: function(e) {
                var randomStrArr = ['childOne', 'childTwo', 'childThree'];
                this.className = "linshui233"
                this.child.someStr = randomStrArr[parseInt(Math.random() * 3)];
            }
        }
    });

    vm.$watch('child.someStr', function() {
        console.log(arguments);
    });
    console.log(vm)
    
```

被之前那么一连串调用搞晕的我们可以直接来看一个动态图:当一个mvvm运行时都处理了什么:

![imgn](http://haoqiao.qiniudn.com/vue-source-active-1.gif)


我们对着demo一步步分析输出的内容

```

0.mvvm初始化
mvvm.js:12 observe data
   正在defineProtoperty:   someStr : "hello "
   正在defineProtoperty:   className : "btn"
   正在defineProtoperty:   htmlStr : "<span style=\"color: #f00;\">red</span>"
   正在defineProtoperty:   child : {"someStr":"World !","test":{"linshui":"Zhaoying"}}
   正在defineProtoperty:   someStr : "World !"
   正在defineProtoperty:   test : {"linshui":"Zhaoying"}
   正在defineProtoperty:   linshui : "Zhaoying"
 compile el data
   正在编译元素节点:
 <input type=​"text">​
   绑定model类型
   model内容更新
 调用Watcher
   添加新依赖
   id0变化，新增监听对象
 

```

第一步我们可以看到先`Observer`了data里的对象。
然后顺着demo写的顺序编译了第一个元素节点。

然后

```


    添加子依赖
   此时getVMVal中遍历后的val值为 
   "hello "
  remove attrv-model
   正在编译元素节点:
 <input type=​"text">​
   绑定model类型
   model内容更新
 调用Watcher
   添加新依赖
   id3变化，新增监听对象
 

   添加子依赖
   添加新依赖
   id4变化，新增监听对象
 

   添加子依赖
   此时getVMVal中遍历后的val值为 
   "World !"
  remove attrv-model
   正在编译元素节点:
 <p class=​"abc btn">​…​</p>​
绑定class类型
class内容更新
 调用Watcher
   添加新依赖
   id1变化，新增监听对象
  
   添加子依赖
   此时getVMVal中遍历后的val值为 
   "btn"
  remove attrv-class
  正在编译文本节点:
  "hello "
  绑定text类型
  text内容更新
 调用Watcher
   添加新依赖
   id0变化，新增监听对象
 
  添加子依赖
   此时getVMVal中遍历后的val值为 
   "hello "
   正在编译元素节点:
 <span>​World !​</span>​
compile.js:119   绑定text类型
compile.js:201   text内容更新
 调用Watcher
   添加新依赖
   id3变化，新增监听对象
 

   添加子依赖
   添加新依赖
   id4变化，新增监听对象
 

   添加子依赖
   此时getVMVal中遍历后的val值为 
   "World !"
  remove attrv-text
   发现存在子节点，准备遍历:
 <span>​World !​</span>​
   的子节点
 [text]
   发现存在子节点，准备遍历:
 <p class=​"abc btn">​…​</p>​
   的子节点
 [text, span, text]
   正在编译元素节点:
 <input type=​"text">​
   绑定model类型
   model内容更新
 调用Watcher
   添加新依赖
   id3变化，新增监听对象

```

这里要先说明一下为什么先编译子元素

```

   想象一下if为false的时候你先编译了父元素，然后，然后就没有了！！所以，要先编译子元素，然后编译父元素根据值来判断是否要保留Dom节点。还有就是指令本身也要在编译完别的指令才编译，否则你节点都没有了，别的指令还怎么编译？

```

最后所有循环下来基本上都是按照我们第一个原理图的顺序走的，Observer先监听所有传入的data里的对象，并维护一个Dep队列，其中每个dep[i]都有属于自己的id，这用于嵌套对象属性的监听。Watcher是用于每次get时加入Dep队列。它会比较一下属性值是否修改，以及是否是新属性，以及如果父元素改动会通知子元素。Compile用于处理传入的指令，然后编译的同时在对应的属性订阅器中添加Watcher。

## 结尾

事实上，还是挺复杂的如果再深究细节。能挖的很多，我觉得现阶段读框架源码我还是不一头全扎进去了。。。深深的感觉肚子里的墨水还很少，不过大概思路了解了，而且学到了很多东西这是这次阅读源码带来的收获。



### 参考文章

[剖析Vue原理&实现双向绑定MVVM](https://segmentfault.com/a/1190000006599500#articleHeader2)

[DMQ 实现mvvm](https://github.com/DMQ/mvvm)

[230行实现一个简单的mvvm](http://div.io/topic/1890)




