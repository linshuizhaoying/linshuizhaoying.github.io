---
layout: post
title: 前端日常恢复计划-Promise
category: 技术
tags: [学习,开发,前端,总结,2018,原生js开发,Source Code,Node,Promise]
keywords: 总结,2018,Promise,source code,前端
description: 
---

## Promise
在我的复习资料的分类中，promise的定义是这样的：

```

Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息

对象的状态不受外界影响。Promise对象代表一个异步操作，有三种状态：Pending（进行中）、Resolved（已完成，又称 Fulfilled）和Rejected（已失败）。

只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是Promise这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变

一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能：从Pending变为Resolved和从Pending变为Rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。就算改变已经发生了，你再对Promise对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的

Promise新建后就会立即执行

Promise是一个对象，充当异步操作和回调函数之间的中介。

     * var promise = new Promise(function(resolve, reject) {  
         // 异步操作的代码  

         if (/* 异步操作成功 */){  
           resolve(value);  
         } else {  
           reject(error);  
         }  
       });
* 每一个异步函数立刻返回一个Promise对象，每个对象指定回调函数，在异步任务完成后调用。
* Promise有三种状态:pending 未完成,resolved 已完成,rejected 失败。最终状态只有成功和失败。通过then方法添加两个回调函数处理resolved状态和rejected状态，一旦状态改变就调用。

     * po.then(function(value) {  
         // success  
       }, function(value) {  
         // failure  
       });

* 优点在于流程清晰，让回调函数变成规范链式写法，可以为多个异步操作指定同一个回调，可以为多个回调指定同一个错误处理方法。如果一个任务已经完成，再往上添加回调，会立刻执行。无须担心错过某个事件。

Promise的then方法返回一个新的Promise，而不是返回this

```

当然，理论知识是理论，我们需要写一点代码来实践。

```

function async(num) {
    let pm = new Promise((resolve, reject) => {
            let time = Math.ceil(Math.random() * 10); //生成1-10的随机数
            setTimeout(function() {
                    console.log("序号:" + num + "随机时间:" + time * 1000); 
                    resolve("序列" + num + "已完成");
            }, time * 1000)
    });
    return pm;
}

async(1).then(function(data){
  console.log(data)
  return async(2);

})
.then(function(data){
    console.log(data);
    return async(3);
})
.then(function(data){
    console.log(data);
});


```

输出结果：

```

序号:1随机时间:9000
序列1已完成
序号:2随机时间:3000
序列2已完成
序号:3随机时间:9000
序列3已完成

```

当我们想进行错误捕捉时我们可以修改代码如下:

```

function async(num) {
    let pm = new Promise((resolve, reject) => {
        let time = Math.ceil(Math.random() * 10); //生成1-10的随机数
        if (time < 5) {
            setTimeout(function() {
                console.log("序号:" + num + "随机时间:" + time * 1000);
                resolve("序列" + num + "已完成");
            }, time * 1000)
        } else {

            reject("序列" + num + "超时");
        }

    });
    return pm;
}

async(1).then(function(data) {
        console.log(data)
        return async(2);

    })
    .then(function(data) {
        console.log(data);
        return async(3);
    })
    .then(function(data) {
        console.log(data);
    })
    .catch(function(reason){
		    console.log('rejected');
		    console.log(reason);
		});


```

输出结果不唯一：

```

序号:1随机时间:1000
序列1已完成
rejected
序列2超时

```

如果在序列1的时候随机数大于5那么就停留在序列1，之后的都不会执行。

`Promise的all方法提供了并行执行异步操作的能力，并且在所有异步操作执行完后才执行回调。`

我们修改代码如下：

```

function async(num) {
    let pm = new Promise((resolve, reject) => {
        let time = Math.ceil(Math.random() * 10); //生成1-10的随机数
        if (time < 9) {
            setTimeout(function() {
                console.log("序号:" + num + "随机时间:" + time * 1000);
                resolve("序列" + num + "已完成");
            }, time * 1000)
        } else {

            reject("序列" + num + "超时");
        }

    });
    return pm;
}

Promise
.all([async(1), async(2), async(3)])
.then(function(results){
    console.log(results);
});

```

在不出错的情况下，输出结果如下:

```

序号:1随机时间:3000
序号:3随机时间:3000
序号:2随机时间:7000
(3) ["序列1已完成", "序列2已完成", "序列3已完成"]

```

如果将`all`改为`race`，那么哪个最先完成会先输出`resolve`结果.其余的不会。

```

序号:3随机时间:3000
序号:2随机时间:2000
序号:1随机时间:3000
序号:2随机时间:2000
序列2已完成  // resolve结果
序号:3随机时间:2000
序号:1随机时间:3000

```

以上是`Promise`的最大众的用法。


## Question

### catch捕获位置

在执行catch测试的时候突然有个小问题，就是catch最终捕获的位置是哪里。

经过查找资料和测试

```

var d = new Date();
// 一秒后进入rejected状态
var promise1 = new Promise(function(resolve, reject) {
    setTimeout(reject, 1000, 'reject from promise1');
});
// 只绑定了onFulfilled回调
var promise2 = promise1.then(result => {
    console.log('promise1.then(resolve):', result);
});
// 绑定了onFulfilled和onRejected。（这里为了演示，正常情况下，建议使用catch处理rejected状态）
promise2.then(
    result => console.log('result:', result, new Date() - d),
    error => console.log('error:', error, new Date() - d)
);

//error: reject from promise1 1004

```

`如果当前的promise实例没有绑定回调函数，或者绑定的不是函数，那么当前实例就会把其状态以及不可变值或者不可变原因传递给当前实例调用.then方法返回的新promise实例。
在上述例子中就表现为，promise1把它的不可变原因以及rejected状态传递给了promise2，所以promise2的onRejected回调方法就把promise1中reject的内容打印出来了。`


### 函数return 形式的闭包的promise写法

```

function doSth() {
    return new Promise(function(resolve, reject) {
        //做点什么异步的事情
        //结束的时候调用 resolve，比如：
        setTimeout(function(){
            resolve(); //这里才是真的返回
        },1000)
    })
}

```

### 如何自己实现一个promise

这里推荐[史上最易读懂的 Promise 实现](https://zhuanlan.zhihu.com/p/21834559)这篇文章。跟着作者一步一步来就行了。

```

try {
  module.exports = Promise
} catch (e) {}

function Promise(executor) {
  var self = this

  self.status = 'pending' // Promise当前的状态
  self.onResolvedCallback = []// Promise resolve时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面
  self.onRejectedCallback = []// Promise reject时的回调函数集，因为在Promise结束之前有可能有多个回调添加到它上面

  function resolve(value) {
    if (value instanceof Promise) {
      return value.then(resolve, reject)
    }
    setTimeout(function() { // 异步执行所有的回调函数
      if (self.status === 'pending') {
        self.status = 'resolved'
        self.data = value
        for (var i = 0; i < self.onResolvedCallback.length; i++) {
          self.onResolvedCallback[i](value)
        }
      }
    })
  }

  function reject(reason) {
    setTimeout(function() { // 异步执行所有的回调函数
      if (self.status === 'pending') {
        self.status = 'rejected'
        self.data = reason
        for (var i = 0; i < self.onRejectedCallback.length; i++) {
          self.onRejectedCallback[i](reason)
        }
      }
    })
  }

  try {
    executor(resolve, reject)
  } catch (reason) {
    reject(reason)
  }
}


/*
resolvePromise函数即为根据x的值来决定promise2的状态的函数
也即标准中的[Promise Resolution Procedure](https://promisesaplus.com/#point-47)
x为`promise2 = promise1.then(onResolved, onRejected)`里`onResolved/onRejected`的返回值
`resolve`和`reject`实际上是`promise2`的`executor`的两个实参，因为很难挂在其它的地方，所以一并传进来。
相信各位一定可以对照标准把标准转换成代码，这里就只标出代码在标准中对应的位置，只在必要的地方做一些解释
*/

function resolvePromise(promise2, x, resolve, reject) {
  var then
  var thenCalledOrThrow = false

  if (promise2 === x) {
    return reject(new TypeError('Chaining cycle detected for promise!'))
  }

  if (x instanceof Promise) {
  / 如果x的状态还没有确定，那么它是有可能被一个thenable决定最终状态和值的
    // 所以这里需要做一下处理，而不能一概的以为它会被一个“正常”的值resolve
    
    if (x.status === 'pending') { //because x could resolved by a Promise Object
      x.then(function(v) {
        resolvePromise(promise2, v, resolve, reject)
      }, reject)
    } else { //but if it is resolved, it will never resolved by a Promise Object but a static value;
      x.then(resolve, reject)
    }
    return
  }

  if ((x !== null) && ((typeof x === 'object') || (typeof x === 'function'))) {
    try {
    // 2.3.3.1 因为x.then有可能是一个getter，这种情况下多次读取就有可能产生副作用
      // 即要判断它的类型，又要调用它，这就是两次读取
      
      then = x.then //because x.then could be a getter
      if (typeof then === 'function') {
        then.call(x, function rs(y) {
          if (thenCalledOrThrow) return
          thenCalledOrThrow = true
          return resolvePromise(promise2, y, resolve, reject)
        }, function rj(r) {
          if (thenCalledOrThrow) return
          thenCalledOrThrow = true
          return reject(r)
        })
      } else {
        resolve(x)
      }
    } catch (e) {
      if (thenCalledOrThrow) return
      thenCalledOrThrow = true
      return reject(e)
    }
  } else {
    resolve(x)
  }
}

Promise.prototype.then = function(onResolved, onRejected) {
  var self = this
  var promise2
  onResolved = typeof onResolved === 'function' ? onResolved : function(v) {
    return v
  }
  onRejected = typeof onRejected === 'function' ? onRejected : function(r) {
    throw r
  }

  if (self.status === 'resolved') {
    return promise2 = new Promise(function(resolve, reject) {
      setTimeout(function() { // 异步执行onResolved
        try {
          var x = onResolved(self.data)
          resolvePromise(promise2, x, resolve, reject)
        } catch (reason) {
          reject(reason)
        }
      })
    })
  }

  if (self.status === 'rejected') {
    return promise2 = new Promise(function(resolve, reject) {
      setTimeout(function() { // 异步执行onRejected
        try {
          var x = onRejected(self.data)
          resolvePromise(promise2, x, resolve, reject)
        } catch (reason) {
          reject(reason)
        }
      })
    })
  }

  if (self.status === 'pending') {
    // 这里之所以没有异步执行，是因为这些函数必然会被resolve或reject调用，而resolve或reject函数里的内容已是异步执行，构造函数里的定义
    return promise2 = new Promise(function(resolve, reject) {
      self.onResolvedCallback.push(function(value) {
        try {
          var x = onResolved(value)
          resolvePromise(promise2, x, resolve, reject)
        } catch (r) {
          reject(r)
        }
      })

      self.onRejectedCallback.push(function(reason) {
          try {
            var x = onRejected(reason)
            resolvePromise(promise2, x, resolve, reject)
          } catch (r) {
            reject(r)
          }
        })
    })
  }
}

Promise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected)
}

Promise.deferred = Promise.defer = function() {
  var dfd = {}
  dfd.promise = new Promise(function(resolve, reject) {
    dfd.resolve = resolve
    dfd.reject = reject
  })
  return dfd
}

```


## 总结

现阶段比较浮躁，太深入的也就没去仔细看了...大底项目中如果要去用以上是足够的。

