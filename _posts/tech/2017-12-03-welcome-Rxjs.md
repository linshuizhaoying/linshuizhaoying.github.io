---
layout: post
title: 将RxJS融入React项目
tags: [学习,开发,前端,Node,React,RxJS,React-observable]
keywords: 学习,开发,前端,RxJS,React,React-observable
description: 
---

# 前言

最近准备毕设，技术选型的时候因为功能的一些需求准备将RxJs融入到项目中，考虑RxJs的时候因为之前的技术栈还犹豫了一下，查了一些资料以及粗略浏览了一些文档。感觉对于毕设项目RxJs的加入是有帮助的，因此打算系统的学习然后摘抄知识点以及实践一些demo做技术积累。


# RxJS技术积累

RxJs经过社区的努力学习资料还是很多的，官方中文文档就已经很不错，不过我们先从[30 天精通 RxJS](https://ithelp.ithome.com.tw/articles/10186103)初步感受一下RxJS.然后配合一些中文文档来补充知识点，最后再根据官方文档来校验整个知识体系。

##  Reactive Programming 的兴起

Reactive Programming 是RxJS 最重要的核心观念之一.

Vue.js的底层就是采用了Reactive Programming的观念来实作的，另外Vue官方也在11月推出了vue-rx。

Angular 2也全面引用了RxJS，不管是在http还是animation都用了RxJS的Observable！

Redux也在3.5版中加入了对Observable操作的支援.

## Observable 标准化

Observable将会被加入ECMAScript的标准

RxJS5 已经符合Observable 标准

## 异步的常见问题

* 竞态条件(Race Condition)

> 每当我们对同一个资源同时做多次的非同步存取时，就可能发生Race Condition 的问题。比如说我们发了一个Request 更新使用者资料，然后我们又立即发送另一个Request 取得使用者资料，这时第一个Request 和第二个Request 先后顺序就会影响到最终接收到的结果不同，这就是Race Condition。

* 记忆体泄漏(Memory Leak)

>做SPA (Single Page Application)网站时，我们是透过JavaScript来达到切换页面的内容，这时如果有对DOM注册监听事件，而没有在适当的时机点把监听的事件移除，就有可能造成Memory Leak。比如说在A页面监听body的scroll事件，但页面切换时，没有把scroll的监听事件移除。

* 复杂的状态(Complex State)

>有一支付费用户才能播放的影片，首先可能要先抓取这部影片的资讯，接着我们要在播放时去验证使用者是否有权限播放，而使用者也有可能再按下播放后又立即按了取消，而这些都是非同步执行，这时就会各种复杂的状态需要处理。


* 例外处理(Exception Handling)

>JavaScript 的try/catch 可以捕捉同步的例外，但非同步的程序就没这么容易，尤其当我们的异步行为很复杂时，这个问题就愈加明显。

## RxJS 基本介绍

`RxJS是一套由Observable sequences来组合异步行为和事件基础程序的Library`

RxJS 是`Functional Programming `跟`Reactive Programming` 的结合

> 把每个运算包成一个个不同的function，并用这些function 组合出我们要的结果，这就是最简单的Functional Programming


Functional Programming 强调没有Side Effect，也就是function 要保持纯粹，只做运算并返回一个值，没有其他额外的行为。

`Side Effect`

>Side Effect是指一个function做了跟本身运算返回值没有关系的事，比如说修改某个全域变数，或是修改传入参数的值，甚至是执行console.log都算是Side Effect。


前端常见的Side Effect:

* 发送http request
* 在画面输出值或是log
* 获得用户的input
* Query DOM 

`Reactive Programming`简单来说就是当变数或资源发生变动时，由变数或资源自动告诉我发生变动了

## Observable

### Observer Pattern（观察者模式）

Observer Pattern 其实很常遇到，许多API 的设计上都用了Observer Pattern，最简单的例子就是DOM 物件的事件监听:

```

function clickHandler(event) {
	console.log('user click!');
}

document.body.addEventListener('click', clickHandler)

```

观察者模式:我们可以对某件事注册监听，并在事件发生时，自动执行我们注册的监听者(listener)。

Es5版本:

```

function Producer() {
	
	// 这个 if 只是避免使用者不小心把 Producer 当做函数调用
	if(!(this instanceof Producer)) {
	  throw new Error('请用 new Producer()!');
	}
	
	this.listeners = [];
}

// 加入监听的方法
Producer.prototype.addListener = function(listener) {
	if(typeof listener === 'function') {
		this.listeners.push(listener)
	} else {
		throw new Error('listener 必须是 function')
	}
}

// 移除监听的方法
Producer.prototype.removeListener = function(listener) {
	this.listeners.splice(this.listeners.indexOf(listener), 1)
}

// 发送通知的方法
Producer.prototype.notify = function(message) {
	this.listeners.forEach(listener => {
		listener(message);
	})
}

```

es6 版本

```

class Producer {
	constructor() {
		this.listeners = [];
	}
	addListener(listener) {
		if(typeof listener === 'function') {
			this.listeners.push(listener)
		} else {
			throw new Error('listener 必须是 function')
		}
	}
	removeListener(listener) {
		this.listeners.splice(this.listeners.indexOf(listener), 1)
	}
	notify(message) {
		this.listeners.forEach(listener => {
			listener(message);
		})
	}
}

```

调用例子:

```

var egghead = new Producer(); 

function listener1(message) {
	console.log(message + 'from listener1');
}

function listener2(message) {
	console.log(message + 'from listener2');
}

egghead.addListener(listener1);egghead.addListener(listener2);

egghead.notify('A new course!!') 


```

输出:

a new course!! from listener1
a new course!! from listener2

### Iterator Pattern （迭代器模式）

>JavaScript 到了ES6 才有原生的Iterator

>在ECMAScript中Iterator最早其实是要采用类似Python的Iterator规范，就是Iterator在没有元素之后，执行next会直接抛出错误；但后来经过一段时间讨论后，决定采更functional的做法，改成在取得最后一个元素之后执行next永远都回传{ done: true, value: undefined }

```

var arr = [1, 2, 3];

var iterator = arr[Symbol.iterator]();

iterator.next();
// { value: 1, done: false }
iterator.next();
// { value: 2, done: false }
iterator.next();
// { value: 3, done: false }
iterator.next();
// { value: undefined, done: true }

```

简单实现:

```

es5:

function IteratorFromArray(arr) {
	if(!(this instanceof IteratorFromArray)) {
		throw new Error('请用 new IteratorFromArray()!');
	}
	this._array = arr;
	this._cursor = 0;	
}

IteratorFromArray.prototype.next = function() {
	return this._cursor < this._array.length ?
		{ value: this._array[this._cursor++], done: false } :
		{ done: true };
}

es6:

class IteratorFromArray {
	constructor(arr) {
		this._array = arr;
		this._cursor = 0;
	}
  
	next() {
		return this._cursor < this._array.length ?
		{ value: this._array[this._cursor++], done: false } :
		{ done: true };
	}
}

```

`优势`

1. Iterator的特性可以拿来做延迟运算(Lazy evaluation)，让我们能用它来处理大数组。
2. 第二因为iterator 本身是序列，所以可以第调用方法像map, filter... 等！

`延迟运算(Lazy evaluation)`

```

function* getNumbers(words) {
		for (let word of words) {
			if (/^[0-9]+$/.test(word)) {
			    yield parseInt(word, 10);
			}
		}
	}
	
	const iterator = getNumbers('30 天精通 RxJS (04)');
	
	iterator.next();
	// { value: 3, done: false }
	iterator.next();
	// { value: 0, done: false }
	iterator.next();
	// { value: 0, done: false }
	iterator.next();
	// { value: 4, done: false }
	iterator.next();
	// { value: undefined, done: true }

```

>把一个字串丢进getNumbersh函数时，并没有马上运算出字串中的所有数字，必须等到我们执行next()时，才会真的做运算，这就是所谓的延迟运算(evaluation strategy)

### Observable简介

Observer跟Iterator有个共通的特性，就是他们都是渐进式 (progressive)的取得资料，差别只在于Observer是生产者(Producer)推送资料(push )，而Iterator是消费者(Consumer)请求资料(pull)!

`Observable其实就是这两个Pattern思想的结合，Observable具备生产者推送资料的特性，同时能像序列，拥有序列处理资料的方法 (map, filter...)！`

RxJS说白了就是一个核心三个重点。

一个核心是Observable 再加上相关的Operators(map, filter...)，这个部份是最重要的，其他三个重点本质上也是围绕着这个核心在转，所以我们会花将近20 天的篇数讲这个部份的观念及使用案例。

另外三个重点分别是

* Observer
* Subject
* Schedulers

#### Observable 实践

`Observable 同时可以处理同步与异步的行为！`

```

同步操作

var observable = Rx.Observable
	.create(function(observer) {
		observer.next('Jerry'); 
		observer.next('Anna');
	})
	
// 订阅 observable	
observable.subscribe(function(value) {
	console.log(value);
})

> Jerry
> Anna

异步操作:

var observable = Rx.Observable
	.create(function(observer) {
		observer.next('Jerry'); // RxJS 4.x 以前的版本用 onNext
		observer.next('Anna');
		
		setTimeout(() => {
			observer.next('RxJS 30 days!');
		}, 30)
	})
	
console.log('start');
observable.subscribe(function(value) {
	console.log(value);
});
console.log('end');

>
start
Jerry
Anna
end
RxJS 30 days!

```

#### 观察者Observer

Observable 可以被订阅(subscribe)，或说可以被观察，而订阅Observable的又称为观察者(Observer)。
观察者是一个具有三个方法(method)的对象，每当Observable 发生事件时，便会呼叫观察者相对应的方法。

* next：每当Observable 发送出新的值，next 方法就会被呼叫。

* complete：在Observable 没有其他的资料可以取得时，complete 方法就会被呼叫，在complete 被呼叫之后，next 方法就不会再起作用。

* error：每当Observable 内发生错误时，error 方法就会被呼叫。

```

var observable = Rx.Observable
	.create(function(observer) {
			observer.next('Jerry');
			observer.next('Anna');
			observer.complete();
			observer.next('not work');
	})
	
// 定义一个观察者
var observer = {
	next: function(value) {
		console.log(value);
	},
	error: function(error) {
		console.log(error)
	},
	complete: function() {
		console.log('complete')
	}
}

//  订阅 observable	
observable.subscribe(observer)

>
Jerry
Anna
complete


// complete执行后，next就会自动失效，所以没有印出not work。

捕获错误实例：

var observable = Rx.Observable
  .create(function(observer) {
    try {
      observer.next('Jerry');
      observer.next('Anna');
      throw 'some exception';
    } catch(e) {
      observer.error(e)
    }
  });

var observer = {
	next: function(value) {
		console.log(value);
	},
	error: function(error) {
		console.log('Error: ', error)
	},
	complete: function() {
		console.log('complete')
	}
}

observable	
observable.subscribe(observer)

>
Jerry
Anna
Error:  some exception

```

`观察者可以是不完整的，他可以只具有一个next 方法`

`订阅一个Observable 就像是执行一个function`

#### Operator操作符

> Operators 就是一个个被附加到Observable 型别的函数，例如像是map, filter, contactAll... 等等

> 每个operator都会回传一个新的observable，而我们可以透过create的方法建立各种operator


Observable 有许多创建实例的方法，称为creation operator。下面我们列出RxJS 常用的creation operator:

```

create
of
from
fromEvent
fromPromise
never
empty
throw
interval
timer

```

当我们想要同步的传递几个值时，就可以用of这个operator来简洁的表达!

```

var source = Rx.Observable.of('Jerry', 'Anna');

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});

// Jerry
// Anna
// complete!

```

用from来接收任何可枚举的参数（Set, WeakSet, Iterator 等都可）

```

var arr = ['Jerry', 'Anna', 2016, 2017, '30 days'] 
var source = Rx.Observable.from(arr);

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});

// Jerry
// Anna
// 2016
// 2017
// 30 days
// complete!


var source = Rx.Observable.from('123');

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});
// 1
// 2
// 3
// complete!


var source = Rx.Observable
  .from(new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('Hello RxJS!');
    },3000)
  }))
  
source.subscribe({
    next: function(value) {
    	console.log(value)
    },
    complete: function() {
    	console.log('complete!');
    },
    error: function(error) {
    console.log(error)
    }
});

// Hello RxJS!
// complete!

```

可以用Event建立Observable，通过`fromEvent`的方法


```

var source = Rx.Observable.fromEvent(document.body, 'click');

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});

```


fromEvent的第一个参数要传入DOM ，第二个参数传入要监听的事件名称。上面的代码会针对body 的click 事件做监听，每当点击body 就会印出event。


> 获取 DOM 的常用方法：
> document.getElementById()
> document.querySelector()
> document.getElementsByTagName()
> document.getElementsByClassName()

Event来建立Observable实例还有另一个方法fromEventPattern，这个方法是给类事件使用

>所谓的类事件就是指其行为跟事件相像，同时具有注册监听及移除监听两种行为，就像DOM Event有addEventListener及removeEventListener一样

```

class Producer {
	constructor() {
		this.listeners = [];
	}
	addListener(listener) {
		if(typeof listener === 'function') {
			this.listeners.push(listener)
		} else {
			throw new Error('listener 必須是 function')
		}
	}
	removeListener(listener) {
		this.listeners.splice(this.listeners.indexOf(listener), 1)
	}
	notify(message) {
		this.listeners.forEach(listener => {
			listener(message);
		})
	}
}

var egghead = new Producer(); 


var source = Rx.Observable
    .fromEventPattern(
        (handler) => egghead.addListener(handler), 
        (handler) => egghead.removeListener(handler)
    );
  
source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
})

egghead.notify('Hello! Can you hear me?');

```

`fromPromise`

创建由 promise 转换而来的 observable，并发出 promise 的结果。

```

// 基于输入来决定是 resolve 还是 reject 的示例 promise
const myPromise = (willReject) => {
    return new Promise((resolve, reject) => {
          if(willReject){
            reject('Rejected!');
        }
        resolve('Resolved!');
    })
}
// 先发出 true，然后是 false
const source = Rx.Observable.of(true, false);
const example = source
    .mergeMap(val => Rx.Observable
        // 将 promise 转换成 observable
        .fromPromise(myPromise(val))
        // 捕获并优雅地处理 reject 的结果
        .catch(error => Rx.Observable.of(`Error: ${error}`))
    )
// 输出: 'Error: Rejected!', 'Resolved!'
const subscribe = example.subscribe(val => console.log(val));


```

`throw`

>在订阅上发出错误

```

// 在订阅上使用指定值来发出错误
const source = Rx.Observable.throw('This is an error!');
// 输出: 'Error: This is an error!'
const subscribe = source.subscribe({
  next: val => console.log(val),
  complete: () => console.log('Complete!'),
  error: val => console.log(`Error: ${val}`)
});

```



`empty`

empty会给我们一个空的observable，如果我们订阅这个observable会发生什么事呢？它会立即送出complete的讯息！

```

var source = Rx.Observable.empty();

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});
// complete!

```

`never`

>never 会给我们一个无穷的observable，如果我们订阅它又会发生什么事呢？...什么事都不会发生，它就是一个一直存在但却什么都不做的observable。

```

var source = Rx.Observable.never();

source.subscribe({
    next: function(value) {
        console.log(value)
    },
    complete: function() {
        console.log('complete!');
    },
    error: function(error) {
        console.log(error)
    }
});

```

`catch`

>catch(project : function): Observable

```

// 发出错误
const source = Rx.Observable.throw('This is an error!');
// 优雅地处理错误，并返回带有错误信息的 observable
const example = source.catch(val => Rx.Observable.of(`I caught: ${val}`));
// 输出: 'I caught: This is an error'
const subscribe = example.subscribe(val => console.log(val));


// 捕获拒绝的 promise

// 创建立即拒绝的 Promise
const myBadPromise = () => new Promise((resolve, reject) => reject('Rejected!'));
// 1秒后发出单个值
const source = Rx.Observable.timer(1000);
// 捕获拒绝的 promise，并返回包含错误信息的 observable
const example = source.flatMap(() => Rx.Observable
                                       .fromPromise(myBadPromise())
                                       .catch(error => Rx.Observable.of(`Bad Promise: ${error}`))
                                    );
// 输出: 'Bad Promise: Rejected'
const subscribe = example.subscribe(val => console.log(val));



```

`interval`

>interval有一个参数必须是数值(Number)，这的数值代表发出讯号的间隔时间(ms)。

```

var source = Rx.Observable.interval(1000);

source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});
// 0
// 1
// 2
// ...

```

`timer`

>有两个参数时，第一个参数代表要发出第一个值的等待时间(ms)，第二个参数代表第一次之后发送值的间隔时间

```

var source = Rx.Observable.timer(1000, 5000);

source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});
// 0
// 1
// 2 ...

var source = Rx.Observable.timer(1000);

source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});
// 0
// complete!
```

timer也可以只接收一个参数,会等N秒后输出结果同时通知结束。


`unsubscribe`

>订阅observable后，会回传一个subscription，拥有释放资源的unsubscribe方法

```

var source = Rx.Observable.timer(1000, 1000);

// 取得 subscription
var subscription = source.subscribe({
	next: function(value) {
		console.log(value)
	},
	complete: function() {
		console.log('complete!');
	},
	error: function(error) {
    console.log('Throw Error: ' + error)
	}
});

setTimeout(() => {
    subscription.unsubscribe()
}, 5000);
// 0
// 1
// 2
// 3
// 4


```


`map`

>传入一个callback function，这个callback function 会带入每次发送出来的元素


```

var source = Rx.Observable.interval(1000);
var newest = source.map(x => x + 2); 

newest.subscribe(console.log);

// 2
// 3
// 4
// 5..

```

`mapTo`

>mapTo 可以把传进来的值改成一个固定的值

```

var source = Rx.Observable.interval(1000);
var newest = source.mapTo(2); 

newest.subscribe(console.log);
// 2
// 2
// 2
// 2..

```

`filter`

>要传入一个callback function，这个function 会传入每个被送出的元素，并且回传一个boolean 值，如果为true 的话就会保留，如果为false 就会被滤掉

```

var source = Rx.Observable.interval(1000);
var newest = source.filter(x => x % 2 === 0); 

newest.subscribe(console.log);
// 0
// 2
// 4
// 6..

// 基于属性过滤对象

// 发出 ({name: 'Joe', age: 31}, {name: 'Bob', age:25})
const source = Rx.Observable.from([{name: 'Joe', age: 31}, {name: 'Bob', age:25}]);
// 过滤掉年龄小于30岁的人
const example = source.filter(person => person.age >= 30);
// 输出: "Over 30: Joe"
const subscribe = example.subscribe(val => console.log(`Over 30: ${val.name}`));

```

`sample`

>当提供的 observable 发出时从源 observable 中取样。
>sample(sampler: Observable): Observable

```

// 每1秒发出值
const source = Rx.Observable.interval(1000);
// 每2秒对源 observable 最新发出的值进行取样
const example = source.sample(Rx.Observable.interval(2000));
// 输出: 2..4..6..8..
const subscribe = example.subscribe(val => console.log(val));

// 当 interval 发出时对源 observable 取样

const source = Rx.Observable.zip(
  // 发出 'Joe', 'Frank' and 'Bob' in sequence
  Rx.Observable.from(['Joe', 'Frank', 'Bob']),
  // 每2秒发出值
  Rx.Observable.interval(2000)
);
// 每2.5秒对源 observable 最新发出的值进行取样
const example = source.sample(Rx.Observable.interval(2500));
// 输出: ["Joe", 0]...["Frank", 1]...........
const subscribe = example.subscribe(val => console.log(val));

```

`take`

>取前几个元素后就结束

```

var source = Rx.Observable.interval(1000);
var example = source.take(3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// complete

```

`takeUntil`
>发出值，直到提供的 observable 发出值，它便完成。
>takeUntil(notifier: Observable): Observable

```

// 每1秒发出值
const source = Rx.Observable.interval(1000);
// 5秒后发出值
const timer = Rx.Observable.timer(5000);
// 当5秒后 timer 发出值时， source 则完成
const example = source.takeUntil(timer);
// 输出: 0,1,2,3
const subscribe = example.subscribe(val => console.log(val));

// 取前5个偶数

// 每1秒发出值
const source = Rx.Observable.interval(1000);
// 是偶数吗？
const isEven = val => val % 2 === 0;
// 只允许是偶数的值
const evenSource = source.filter(isEven);
// 保存运行中的偶数数量
const evenNumberCount = evenSource
    .scan((acc, _) => acc + 1, 0);
// 不发出直到发出过5个偶数
const fiveEvenNumbers = evenNumberCount.filter(val => val > 5);

const example = evenSource
    // 还给出当前偶数的数量以用于显示
  .withLatestFrom(evenNumberCount)
    .map(([val, count]) => `Even number (${count}) : ${val}`)
    // 当发出了5个偶数时，source 则完成
  .takeUntil(fiveEvenNumbers);
/*
    Even number (1) : 0,
    Even number (2) : 2
    Even number (3) : 4
    Even number (4) : 6
    Even number (5) : 8
*/
const subscribe = example.subscribe(val => console.log(val));

```

`takeWhile`

>发出值，直到提供的表达式结果为 false 
>takeWhile(predicate: function(value, index): boolean): Observable

```

// 使用限定条件取值

// 发出 1,2,3,4,5
const source = Rx.Observable.of(1,2,3,4,5);
// allow values until value from source is greater than 4, then complete
// 允许值发出直到 source 中的值大于4，然后便完成
const example = source.takeWhile(val => val <= 4);
// 输出: 1,2,3,4
const subscribe = example.subscribe(val => console.log(val));

// takeWhile() 和 filter() 的区别

// 发出 3, 3, 3, 9, 1, 4, 5, 8, 96, 3, 66, 3, 3, 3
const source = Rx.Observable.of(3, 3, 3, 9, 1, 4, 5, 8, 96, 3, 66, 3, 3, 3);

// 允许值通过直到源发出的值不等于3，然后完成
// 输出: [3, 3, 3]
source
 .takeWhile(it => it === 3 )
 .subscribe(val => console.log('takeWhile', val));

// 输出: [3, 3, 3, 3, 3, 3, 3]
source
 .filter(it => it === 3)
 .subscribe(val => console.log('filter', 3));


```


`first`

> 会取observable 送出的第1 个元素之后就直接结束

```

var source = Rx.Observable.interval(1000);
var example = source.first();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

// 0
// complete

```

`takeUntil`

>他可以在某件事情发生时，让一个observable 直接发送完成(complete)

```

var source = Rx.Observable.interval(1000);
var click = Rx.Observable.fromEvent(document.body, 'click');
var example = source.takeUntil(click);     
   
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// 3
// complete (点击body

```

`concatAll`

>有时我们的Observable里的元素又是一个observable，就像是二维阵列，阵列里面的元素是阵列，这时我们就可以用concatAll把它摊平成一维阵列，大家也可以直接把concatAll想成把所有元素concat起来

```

var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.of(1,2,3));

var example = source.concatAll();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

`concatAll`是同步的，一个任务接一个任务执行,也就是等到当前处理的结束后才会再处理下一个.

```

var obs1 = Rx.Observable.interval(1000).take(5);
var obs2 = Rx.Observable.interval(500).take(2);
var obs3 = Rx.Observable.interval(2000).take(1);

var source = Rx.Observable.of(obs1, obs2, obs3);

var example = source.concatAll();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// 3
// 4
// 0
// 1
// 0
// complete

```

[拉拽效果](https://jsfiddle.net/s6323859/1ahzh7a7/2/)

![imgn](http://haoqiao.qiniudn.com/WechatIMG50.jpeg)


```
// 取得body 的原因是因为移动(mousemove)跟左键放掉(mouseup)都应该是在整个body 监听。

const dragDOM = document.getElementById('drag');
const body = document.body;

const mouseDown = Rx.Observable.fromEvent(dragDOM, 'mousedown');
const mouseUp = Rx.Observable.fromEvent(body, 'mouseup');
const mouseMove = Rx.Observable.fromEvent(body, 'mousemove');

// 当mouseDown 时，转成mouseMove 的事件
// mouseMove 要在mouseUp 后结束

mouseDown
	.map(event => mouseMove.takeUntil(mouseUp))
  .concatAll()
  .map(event => ({ x: event.clientX, y: event.clientY }))
  .subscribe(pos => {
  	dragDOM.style.left = pos.x + 'px';
    dragDOM.style.top = pos.y + 'px';
  })

```


`forkJoin`

>当所有 observables 完成时，发出每个 observable 的最新值。

>forkJoin(...args, selector : function): Observable

```

// 发起可变数量的请求

const myPromise = val => new Promise(resolve => setTimeout(() => resolve(`Promise Resolved: ${val}`), 5000))

/*
  当所有 observables 完成是，将每个 observable 
  的最新值作为数组发出
*/
const example = Rx.Observable.forkJoin(
  // 立即发出 'Hello'
  Rx.Observable.of('Hello'),
  // 1秒后发出 'World'
  Rx.Observable.of('World').delay(1000),
  // 1秒后发出0
  Rx.Observable.interval(1000).take(1),
  // 以1秒的时间间隔发出0和1
  Rx.Observable.interval(1000).take(2),
  // 5秒后解析 'Promise Resolved' 的 promise
  myPromise('RESULT')
);
//输出: ["Hello", "World", 0, 1, "Promise Resolved: RESULT"]
const subscribe = example.subscribe(val => console.log(val));

// 发起5个请求
const queue = Rx.Observable.of([1,2,3,4,5]);
// 发出包含所有5个结果的数组
const exampleTwo = queue
  .mergeMap(q => Rx.Observable.forkJoin(...q.map(myPromise)));
/*
  输出:
  [
   "Promise Resolved: 1", 
   "Promise Resolved: 2", 
   "Promise Resolved: 3", 
   "Promise Resolved: 4",    
   "Promise Resolved: 5"
  ]
*/
const subscribeTwo = exampleTwo.subscribe(val => console.log(val));


```



`skip`

>略过前几个元素

```

var source = Rx.Observable.interval(1000);
var example = source.skip(3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 3
// 4
// 5...

```

`takeLast`

>倒过来取最后几个

```

var source = Rx.Observable.interval(1000).take(6);
var example = source.takeLast(2);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 4
// 5
// complete

```

`last` = `takeLast(1)`

>取得最后一个元素

```

var source = Rx.Observable.interval(1000).take(6);
var example = source.last();

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 5
// complete

```

`concat`

>按照顺序，前一个 observable 完成了再订阅下一个 observable 并发出值.concat 可以把多个observable 实例合并成一个

>concat(observables: ...*): Observable

```

var source = Rx.Observable.interval(1000).take(3);
var source2 = Rx.Observable.of(3)
var source3 = Rx.Observable.of(4,5,6)
var example = source.concat(source2, source3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// 3
// 4
// 5
// 6
// complete

```


`startWith`

>startWith可以在observable的一开始加入要发送的元素

>startWith(an: Values): Observable



```

var source = Rx.Observable.interval(1000);
var example = source.startWith(0);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 0
// 1
// 2
// 3...


//  startWith 用作 scan 的初始值

// 发出 ('World!', 'Goodbye', 'World!')
const source = Rx.Observable.of('World!', 'Goodbye', 'World!');
// 以 'Hello' 开头，后面接当前字符串
const example = source
  .startWith('Hello')
  .scan((acc, curr) => `${acc} ${curr}`);
/*
  输出:
  "Hello"
  "Hello World!"
  "Hello World! Goodbye"
  "Hello World! Goodbye World!"
*/
const subscribe = example.subscribe(val => console.log(val));


//  使用多个值进行 startWith

// 每1秒发出值
const source = Rx.Observable.interval(1000);
// 以 -3, -2, -1 开始
const example = source.startWith(-3, -2, -1);
// 输出: -3, -2, -1, 0, 1, 2....
const subscribe = example.subscribe(val => console.log(val));

```

`merge`

>将多个 observables 转换成单个 observable 。
>merge(input: Observable): Observable

```

var source = Rx.Observable.interval(500).take(3);
var source2 = Rx.Observable.interval(300).take(6);
var example = source.merge(source2);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 0
// 1
// 2
// 1
// 3
// 2
// 4
// 5
// complete

merge把多个observable同时处理，这跟concat一次处理一个observable是完全不一样的

merge之后的example在时间序上同时在跑source与source2，当两件事情同时发生时，会同步送出资料(被merge的在后面)，当两个observable都结束时才会真的结束。

```

`combineLatest`

>当任意 observable 发出值时，发出每个 observable 的最新值。

>combineLatest(observables: ...Observable, project: function): Observable

```

var source = Rx.Observable.interval(500).take(3);
var newest = Rx.Observable.interval(300).take(6);

var example = source.combineLatest(newest, (x, y) => x + y);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// 3
// 4
// 5
// 6
// 7
// complete

newest送出了0，但此时source并没有送出过任何值，所以不会执行callback
source送出了0，此时newest最后一次送出的值为0，把这两个数传入callback得到0。
newest送出了1，此时source最后一次送出的值为0，把这两个数传入callback得到1。
newest送出了2，此时source最后一次送出的值为0，把这两个数传入callback得到2。
source送出了1，此时newest最后一次送出的值为2，把这两个数传入callback得到3。
newest送出了3，此时source最后一次送出的值为1，把这两个数传入callback得到4。
source送出了2，此时newest最后一次送出的值为3，把这两个数传入callback得到5。
source 结束，但newest 还没结束，所以example 还不会结束。
newest送出了4，此时source最后一次送出的值为2，把这两个数传入callback得到6。
newest送出了5，此时source最后一次送出的值为2，把这两个数传入callback得到7。
newest 结束，因为source 也结束了，所以example 结束。

例子2:

// timerOne 在1秒时发出第一个值，然后每4秒发送一次
const timerOne = Rx.Observable.timer(1000, 4000);
// timerTwo 在2秒时发出第一个值，然后每4秒发送一次
const timerTwo = Rx.Observable.timer(2000, 4000)
// timerThree 在3秒时发出第一个值，然后每4秒发送一次
const timerThree = Rx.Observable.timer(3000, 4000)

// 当一个 timer 发出值时，将每个 timer 的最新值作为一个数组发出
const combined = Rx.Observable
.combineLatest(
    timerOne,
    timerTwo,
    timerThree
);

const subscribe = combined.subscribe(latestValues => {
  // 从 timerValOne、timerValTwo 和 timerValThree 中获取最新发出的值
    const [timerValOne, timerValTwo, timerValThree] = latestValues;
  /*
      示例:
    timerOne first tick: 'Timer One Latest: 1, Timer Two Latest:0, Timer Three Latest: 0
    timerTwo first tick: 'Timer One Latest: 1, Timer Two Latest:1, Timer Three Latest: 0
    timerThree first tick: 'Timer One Latest: 1, Timer Two Latest:1, Timer Three Latest: 1
  */
  console.log(
    `Timer One Latest: ${timerValOne}, 
     Timer Two Latest: ${timerValTwo}, 
     Timer Three Latest: ${timerValThree}`
   );
});


使用 projection 函数的 combineLatest:

// timerOne 在1秒时发出第一个值，然后每4秒发送一次
const timerOne = Rx.Observable.timer(1000, 4000);
// timerTwo 在2秒时发出第一个值，然后每4秒发送一次
const timerTwo = Rx.Observable.timer(2000, 4000)
// timerThree 在3秒时发出第一个值，然后每4秒发送一次
const timerThree = Rx.Observable.timer(3000, 4000)

// combineLatest 还接收一个可选的 projection 函数
const combinedProject = Rx.Observable
.combineLatest(
    timerOne,
    timerTwo,
    timerThree,
    (one, two, three) => {
      return `Timer One (Proj) Latest: ${one}, 
              Timer Two (Proj) Latest: ${two}, 
              Timer Three (Proj) Latest: ${three}`
    }
);
// 输出值
const subscribe = combinedProject.subscribe(latestValuesProject => console.log(latestValuesProject));


组合2个按钮的事件:

<div>
  <button id='red'>Red</button>
  <button id='black'>Black</button>
</div>
<div id="redTotal"></div>
<div id="blackTotal"></div>
<div id="total"></div>

// 用来设置 HTML 的辅助函数
const setHtml = id => val => document.getElementById(id).innerHTML = val;

const addOneClick$ = id => Rx.Observable
    .fromEvent(document.getElementById(id), 'click')
    // 将每次点击映射成1
    .mapTo(1)
    .startWith(0)
    // 追踪运行中的总数
    .scan((acc, curr) => acc + curr)
    // 为适当的元素设置 HTML
    .do(setHtml(`${id}Total`))


const combineTotal$ = Rx.Observable
  .combineLatest(
    addOneClick$('red'),
    addOneClick$('black')
  )
  .map(([val1, val2]) => val1 + val2)
  .subscribe(setHtml('total'));


```



`combineAll`

> combineAll(project: function): Observable

>当源 observable 完成时，对收集的 observables 使用 combineLatest

```
// 映射成内部的 interval observable


//每秒发出值，并只取前2个
const source = Rx.Observable.interval(1000).take(2);
//将 source 发出的每个值映射成取前5个值的 interval observable 
const example = source.map(val => Rx.Observable.interval(1000).map(i => `Result (${val}): ${i}`).take(5));
/*
  soure 中的2个值会被映射成2个(内部的) interval observables，
  这2个内部 observables 每秒使用 combineLatest 策略来 combineAll，
  每当任意一个内部 observable 发出值，就会发出每个内部 observable 的最新值。
*/
const combined = example.combineAll();
/*
  输出:
  ["Result (0): 0", "Result (1): 0"]
  ["Result (0): 1", "Result (1): 0"]
  ["Result (0): 1", "Result (1): 1"]
  ["Result (0): 2", "Result (1): 1"]
  ["Result (0): 2", "Result (1): 2"]
  ["Result (0): 3", "Result (1): 2"]
  ["Result (0): 3", "Result (1): 3"]
  ["Result (0): 4", "Result (1): 3"]
  ["Result (0): 4", "Result (1): 4"]
*/
const subscribe = combined.subscribe(val => console.log(val));


```


`zip`

>zip 会取每个observable 相同顺位的元素并传入callback，也就是说每个observable 的第n 个元素会一起被传入callback 

>zip 操作符会订阅所有内部 observables，然后等待每个发出一个值。一旦发生这种情况，将发出具有相应索引的所有值。这会持续进行，直到至少一个内部 observable 完成。

>zip(observables: *): Observable


```

// 以交替的时间间隔对多个 observables 进行 zip

const sourceOne = Rx.Observable.of('Hello');
const sourceTwo = Rx.Observable.of('World!');
const sourceThree = Rx.Observable.of('Goodbye');
const sourceFour = Rx.Observable.of('World!');
// 一直等到所有 observables 都发出一个值，才将所有值作为数组发出
const example = Rx.Observable
  .zip(
    sourceOne,
    sourceTwo.delay(1000),
    sourceThree.delay(2000),
    sourceFour.delay(3000)
  );
// 输出: ["Hello", "World!", "Goodbye", "World!"]
const subscribe = example.subscribe(val => console.log(val));

// 当一个 observable 完成时进行 zip

// 每1秒发出值
const interval = Rx.Observable.interval(1000);
// 当一个 observable 完成时，便不会再发出更多的值了
const example = Rx.Observable
  .zip(
    interval,
    interval.take(2)
  );
// 输出: [0,0]...[1,1]
const subscribe = example.subscribe(val => console.log(val));



var source = Rx.Observable.interval(500).take(3);
var newest = Rx.Observable.interval(300).take(6);

var example = source.zip(newest, (x, y) => x + y);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 2
// 4
// complete

newest送出了第一个值0，但此时source并没有送出第一个值，所以不会执行callback。
source送出了第一个值0，newest之前送出的第一个值为0，把这两个数传入callback得到0。
newest送出了第二个值1，但此时source并没有送出第二个值，所以不会执行callback。
newest送出了第三个值2，但此时source并没有送出第三个值，所以不会执行callback。
source送出了第二个值1，newest之前送出的第二个值为1，把这两个数传入callback得到2。
newest送出了第四个值3，但此时source并没有送出第四个值，所以不会执行callback。
source送出了第三个值2，newest之前送出的第三个值为2，把这两个数传入callback得到4。
source 结束example 就直接结束，因为source 跟newest 不会再有对应顺位的值


```


`withLatestFrom`

>withLatestFrom 运作方式跟combineLatest 有点像，只是他有主从的关系，只有在主要的observable 送出新的值时，才会执行callback，附随的observable 只是在背景下运作。

>withLatestFrom(other: Observable, project: Function): Observable

[完成拖拽Demo](https://jsfiddle.net/s6323859/ochbtpk5/3/)


![imgn](http://haoqiao.qiniudn.com/WechatIMG52.jpeg)

```

// 发出频率更快的第二个 source 的最新值,主要的停止，附属的也会停止

// 每5秒发出值
const source = Rx.Observable.interval(5000).take(2);
// 每1秒发出值
const secondSource = Rx.Observable.interval(1000);
const example = source
  .withLatestFrom(secondSource)
  .map(([first, second]) => {
    return `First Source (5s): ${first} Second Source (1s): ${second}`;
  });
/*
  输出:
  "First Source (5s): 0 Second Source (1s): 4"
  "First Source (5s): 1 Second Source (1s): 9"

*/
const subscribe = example.subscribe(val => console.log(val));


```

`defaultIfEmpty`

>如果在完成前没有发出任何通知，那么发出给定的值
>defaultIfEmpty(defaultValue: any): Observable

```

// 没有值的 Observable.of 的默认值

const empty = Rx.Observable.of();
// 当源 observable 为空时，发出 'Observable.of() Empty!'，否则发出源的任意值
const exampleOne = empty.defaultIfEmpty('Observable.of() Empty!');
// 输出: 'Observable.of() Empty!'
const subscribe = exampleOne.subscribe(val => console.log(val));

//  Observable.empty 的默认值

// 空的 observable 
const empty = Rx.Observable.empty();
// 当源 observable 为空时，发出 'Observable.empty()!'，否则发出源的任意值
const example = empty.defaultIfEmpty('Observable.empty()!');
// 输出: 'Observable.empty()!'
const subscribe = example.subscribe(val => console.log(val));


```

`every`

>如果完成时所有的值都能通过断言，那么发出 true，否则发出 false 。
>every(predicate: function, thisArg: any): Observable

```

// 一些值不符合条件

// 发出5个值
const source = Rx.Observable.of(1,2,3,4,5);
const example = source
  // 每个值都是偶数吗？
  .every(val => val % 2 === 0)
// 输出: false
const subscribe = example.subscribe(val => console.log(val));


```

`scan`

>scan 其实就是Observable 版本的reduce

```

var source = Rx.Observable.from('hello')
             .zip(Rx.Observable.interval(600), (x, y) => x);

var example = source.scan((origin, next) => origin + next, '');

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// h
// he
// hel
// hell
// hello
// complete

```

scan 常用在状态的计算处理，最简单的就是对一个数字的加减，我们可以绑定一个button 的click 事件，并用map 把click event 转成1，之后送处scan 计算值再做显示

```

const addButton = document.getElementById('addButton');
const minusButton = document.getElementById('minusButton');
const state = document.getElementById('state');

const addClick = Rx.Observable.fromEvent(addButton, 'click').mapTo(1);
const minusClick = Rx.Observable.fromEvent(minusButton, 'click').mapTo(-1);

const numberState = Rx.Observable.empty()
  .startWith(0)
  .merge(addClick, minusClick)
  .scan((origin, next) => origin + next, 0)
  
numberState
  .subscribe({
    next: (value) => { state.innerHTML = value;},
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
  });

```

`buffer`

* buffer
* bufferCount
* bufferTime
* bufferToggle
* bufferWhen

```

var source = Rx.Observable.interval(300);
var source2 = Rx.Observable.interval(1000);
var example = source.buffer(source2);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// [0,1,2]
// [3,4,5]
// [6,7,8]...

buffer 要传入一个observable(source2)，它会把原本的observable (source)送出的元素缓存在阵列中，等到传入的observable(source2) 送出元素时，就会触发把缓存的元素送出。

等价于 

var source = Rx.Observable.interval(300);
var example = source.bufferCount(3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

`buffer来做某个事件的过滤`

```

只有在500毫秒内连点两下，才能成功印出'success'

const button = document.getElementById('demo');
const click = Rx.Observable.fromEvent(button, 'click')
const example = click
                .bufferTime(500)
                .filter(arr => arr.length >= 2);

example.subscribe({
    next: (value) => { console.log('success'); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

`delay`

>delay 可以延迟observable 一开始发送元素的时间点

```

var source = Rx.Observable.interval(300).take(5);

var example = source.delay(500);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// 3
// 4


var source = Rx.Observable.interval(300).take(5);

var example = source.delay(new Date(new Date().getTime() + 1000));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

`delayWhen`

>delayWhen 的作用跟delay 很像，最大的差别是delayWhen 可以影响每个元素,而且需要传一个callback 并回传一个observable

```

var source = Rx.Observable.interval(300).take(5);

var example = source
              .delayWhen(
                  x => Rx.Observable.empty().delay(100 * x * x)
              );

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[图片跟随例子](https://jsbin.com/hayixa/2/edit?html,css,js,output)

![imgn](http://haoqiao.qiniudn.com/rxdelay.gif)

`debounce`

>debounce运作的方式是每次收到元素，他会先把元素cache住并等待一段时间，如果这段时间内已经没有收到任何元素，则把元素送出；如果这段时间内又收到新的元素，则会把原本cache住的元素释放掉并重新计时，不断反覆。

```

var source = Rx.Observable.interval(300).take(5);
var example = source.debounceTime(1000);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 4
// complete

我们每300 毫秒就会送出一个数值，但我们的debounceTime 是1000 毫秒，也就是说每次debounce 收到元素还等不到1000 毫秒，就会收到下一个新元素，然后重新等待1000 毫秒，如此重复直到第五个元素送出时，observable 结束(complete)了，debounce 就直接送出元素。

```

debounceTime优化输入

```

const searchInput = document.getElementById('searchInput');
const theRequestValue = document.getElementById('theRequestValue');

Rx.Observable.fromEvent(searchInput, 'input')
  .debounceTime(300)
  .map(e => e.target.value)
  .subscribe((value) => {
    theRequestValue.textContent = value;
  })

```

`throttle`

>跟debounce 的不同是throttle 会先开送出元素，等到有元素被送出就会沉默一段时间，等到时间过了又会开放发送元素。

```

var source = Rx.Observable.interval(300).take(5);
var example = source.throttleTime(1000);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 4
// complete

```

> throttle比较像是控制行为的最高频率，也就是说如果我们设定1000毫秒，那该事件频率的最大值就是每秒触发一次不会再更快，debounce则比较像是必须等待的时间，要等到一定的时间过了才会收到元素。

`throttle更适合用在连续性行为，比如说UI动画的运算过程，因为UI动画是连续的，像我们之前在做拖拉时，就可以加上throttleTime(12)让mousemove event不要发送的太快，避免画面更新的速度跟不上样式的切换速度。`

`distinct` 

>把相同值的内容滤掉只留一个

> 有两种去缓存方式

```

var source = Rx.Observable.from(['a', 'b', 'c', 'a', 'b'])
            .zip(Rx.Observable.interval(300), (x, y) => x);
var example = source.distinct()

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// a
// b
// c
// complete


```

可以传入一个selector callback function，这个callback function 会传入一个接收到的元素，并回传我们真正希望比对的值

```

var source = Rx.Observable.from([{ value: 'a'}, { value: 'b' }, { value: 'c' }, { value: 'a' }, { value: 'c' }])
            .zip(Rx.Observable.interval(300), (x, y) => x);
var example = source.distinct((x) => {
    return x.value
});

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// {value: "a"}
// {value: "b"}
// {value: "c"}
// complete

```

`distinctUntilChanged`

>只有当当前值与之前最后一个值不同时才将其发出。

>distinctUntilChanged(compare: function): Observable

```
//  使用基础值进行 distinctUntilChanged

//only output distinct values, based on the last emitted value
// 基于最新发出的值进行比较，只输出不同的值
const myArrayWithDuplicatesInARow = Rx.Observable
  .from([1,1,2,2,3,1,2,3]);

const distinctSub = myArrayWithDuplicatesInARow
    .distinctUntilChanged()
      // 输出 : 1,2,3,1,2,3
    .subscribe(val => console.log('DISTINCT SUB:', val));

const nonDistinctSub = myArrayWithDuplicatesInARow
    // 输出 : 1,1,2,2,3,1,2,3
    .subscribe(val => console.log('NON DISTINCT SUB:', val));

// 使用对象进行 distinctUntilChanged

const sampleObject = {name: 'Test'};
// 对象必须有相同的引用
const myArrayWithDuplicateObjects = Rx.Observable.from([sampleObject, sampleObject, sampleObject]);
// 基于最新发出的值进行比较，只输出不同的对象
const nonDistinctObjects = myArrayWithDuplicateObjects
  .distinctUntilChanged()
  // 输出: 'DISTINCT OBJECTS: {name: 'Test'}
  .subscribe(val => console.log('DISTINCT OBJECTS:', val));


```



`catch`

> RxJS 中的 catch 可以回传一个observable 来送出新的值, catch 可以回传一个新的Observable、Promise、Array 或任何Iterable 的物件，来传送之后的元素。

```

var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .catch(error => Rx.Observable.of('h'));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});                    


// 遇到错误后，让observable 结束

var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .catch(error => Rx.Observable.empty());

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});                    

```

`catch 的callback 能接收第二个参数，这个参数会接收当前的observalbe，我们可以回传当前的observable 来做到重新执行,通常会用在断线重连的情境`

```

var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .catch((error, obs) => obs);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});                    

```

`retry`

>retry会放在即时同步的重新连接,可以无限重复，也可传递n，重复n次结束,retry 只有在例外发生时才触发

```

var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .retry(1);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
}); 
// a
// b
// c
// d
// a
// b
// c
// d
// Error: TypeError: x.toUpperCase is not a function

```

`retryWhen`

>可以把例外发生的元素放到一个observable中，让我们可以直接操作这个observable，并等到这个observable操作完后再重新订阅一次原本的observable。

`通常会把retryWhen 拿来做错误通知或是例外收集`

```

var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source
                .map(x => x.toUpperCase())
                .retryWhen(
                errorObs => errorObs.map(err => fetch('...')));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
}); 

```

`repeat`

> 一直重复订阅

```

var source = Rx.Observable.from(['a','b','c'])
            .zip(Rx.Observable.interval(500), (x,y) => x);

var example = source.repeat(1);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
 
// a
// b
// c
// a
// b
// c
// complete

```

`错误提示例子`

> 即时同步断线时，利用catch 返回一个新的observable，这个observable 会先送出错误讯息并且把原本的observable 延迟5 秒再做合并，虽然这只是一个模仿，但它清楚的展示了RxJS 在做错误处理时的灵活性。

```

const title = document.getElementById('title');

var source = Rx.Observable.from(['a','b','c','d',2])
            .zip(Rx.Observable.interval(500), (x,y) => x)
            .map(x => x.toUpperCase()); 
            

var example = source.catch(
                (error, obs) => Rx.Observable.empty()
                               .startWith('连线错误，五秒后重新连接')
                               .concat(obs.delay(5000))
                 );

example.subscribe({
    next: (value) => { title.innerText = value },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
}); 

```

`concatAll`

>处理完前一个observable 才会在处理下一个observable


```
// observable是无限的永远不会完成(complete)，就导致他永远不会处理第二个送出的observable

var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000));

var example = source.concatAll();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

// observable 变成有限的，只会送出三个元素

var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000).take(3));

var example = source.concatAll();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});


// 使用 promise 来进行 concatAll

// 创建并解析一个基础的 promise
const samplePromise = val => new Promise(resolve => resolve(val));
// 每2秒发出值
const source = Rx.Observable.interval(2000);

const example = source
  .map(val => samplePromise(val))
  // 合并解析过的 promise 的值
  .concatAll();
// 输出: 'Example with Promise 0', 'Example with Promise 1'...
const subscribe = example.subscribe(val => console.log('Example with Promise:', val));

```

`switch`

>会在新的observable送出后直接处理新的observable不管前一个observable是否完成，每当有新的observable送出就会直接把旧的observable退订(unsubscribe)，永远只处理最新的observable


```

var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000));

var example = source.switch();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

`mergeAll`

>收集并订阅所有的 observables 

>把二维的observable 转成一维的，并且能够同时处理所有的observable,所有的observable 是并行(Parallel)处理的，也就是说mergeAll 不会像switch 一样退订(unsubscribe)原先的observable 而是并行处理多个observable

>mergeAll(concurrent: number): Observable

```

// 使用 promise 来进行 concatAll

const myPromise = val => new Promise(resolve => setTimeout(() => resolve(`Result: ${val}`), 2000))
// 发出 1,2,3
const source = Rx.Observable.of(1,2,3);

const example = source
  // 将每个值映射成 promise
  .map(val => myPromise(val))
  // 发出 source 的结果
  .mergeAll();

/*
  输出:
  "Result: 1"
  "Result: 2"
  "Result: 3"
*/
const subscribe = example.subscribe(val => console.log(val));


```

```

var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000));

var example = source.mergeAll();
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

mergeAll 可以传入一个数值，这个数值代表他可以同时处理的observable 数量

```
// 前面两个observabel 可以被并行处理，但第三个observable 必须等到第一个observable 结束后，才会开始。

var click = Rx.Observable.fromEvent(document.body, 'click');
var source = click.map(e => Rx.Observable.interval(1000).take(3));

var example = source.mergeAll(2);
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

`pairwise`

>将前一个值和当前值作为数组发出

>签名: pairwise(): Observable<Array>

```

var interval = Rx.Observable.interval(1000);

// 返回: [0,1], [1,2], [2,3], [3,4], [4,5]
interval.pairwise()
    .take(5)
    .subscribe(console.log);

```


`race`

>使用首先发出值的 observable 
>签名: race(): Observable

```

// 接收第一个发出值的 observable
const example = Rx.Observable.race(
  // 每1.5秒发出值
  Rx.Observable.interval(1500),
  // 每1秒发出值
  Rx.Observable.interval(1000).mapTo('1s won!'),
  // 每2秒发出值
  Rx.Observable.interval(2000),
  // 每2.5秒发出值
  Rx.Observable.interval(2500)
);
//输出: "1s won!"..."1s won!"...etc
const subscribe = example.subscribe(val => console.log(val));


```


`concatMap`

>concatMap 其实就是map 加上concatAll 的简化写法

> 将值映射成内部 observable，并按顺序订阅和发出。
> concatMap(project: function, resultSelector: function): Observable

```

var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .map(e => Rx.Observable.interval(1000).take(3))
                .concatAll();
                
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});


to:


var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .concatMap(
                    e => Rx.Observable.interval(100).take(3)
                );
                
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

// concatMap 也会先处理前一个送出的observable 再处理下一个observable


function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.concatMap(
                    e => Rx.Observable.from(getPostData()));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

//  映射成内部 observable

// 发出 'Hello' 和 'Goodbye'
const source = Rx.Observable.of('Hello', 'Goodbye');
// 将 source 的值映射成内部 observable，当一个完成发出结果后再继续下一个
const example = source.concatMap(val => Rx.Observable.of(`${val} World!`));
// 输出: 'Example One: 'Hello World', Example One: 'Goodbye World'
const subscribe = example
  .subscribe(val => console.log('Example One:', val));


// 映射成 promise

// 发出 'Hello' 和 'Goodbye'
const source = Rx.Observable.of('Hello', 'Goodbye');
// 使用 promise 的示例
const examplePromise = val => new Promise(resolve => resolve(`${val} World!`));
// 将 source 的值映射成内部 observable，当一个完成发出结果后再继续下一个
const example = source.concatMap(val => examplePromise(val))
// 输出: 'Example w/ Promise: 'Hello World', Example w/ Promise: 'Goodbye World'
const subscribe = example.subscribe(val => console.log('Example w/ Promise:', val));



```


concatMap 还有第二个参数是一个selector callback，这个callback 会传入四个参数，分别是

* 外部observable 送出的元素
* 内部observable 送出的元素
* 外部observable 送出元素的index
* 内部observable 送出元素的index

```

function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.concatMap(
                e => Rx.Observable.from(getPostData()), 
                (e, res, eIndex, resIndex) => res.title);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```


`switchMap`

>switchMap 其实就是map 加上switch 简化的写法,switchMap 会在下一个observable 被送出后直接退订前一个未处理完的observable，适合自动完成(auto complete)

```

var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .map(e => Rx.Observable.interval(1000).take(3))
                .switch();
                
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

to:

var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .switchMap(
                    e => Rx.Observable.interval(100).take(3)
                );
                
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

// 发送了多个request 但最后真正印出来的log 只会有一个

function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.switchMap(
                    e => Rx.Observable.from(getPostData()));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

`mergeMap`

>mergeMap 其实就是map 加上mergeAll 简化的写法

```

var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .map(e => Rx.Observable.interval(1000).take(3))
                .mergeAll();
                
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});


to:

var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source
                .mergeMap(
                    e => Rx.Observable.interval(100).take(3)
                );
                
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

// mergeMap 可以并行处理多个observable，以这个例子来说当我们快速点按两下，元素发送的时间点是有机会重叠的

function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.mergeMap(
                    e => Rx.Observable.from(getPostData()));

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

mergeMap 也能传入第二个参数selector callback，这个selector callback 跟concatMap 第二个参数也是完全一样的，但mergeMap 的重点是我们可以传入第三个参数，来限制并行处理的数量


```
// 限制同时发送的request 数量

function getPostData() {
    return fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
}
var source = Rx.Observable.fromEvent(document.body, 'click');

var example = source.mergeMap(
                e => Rx.Observable.from(getPostData()), 
                (e, res, eIndex, resIndex) => res.title, 3);

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});

```

[Auto Complete 实例](https://jsfiddle.net/qsaop1qq/3/)


```

co3st url = 'https://zh.wikipedia.org/w/api.php?action=opensearch&format=json&limit=5&origin=*';

const getSuggestList = (keyword) => fetch(url + '&search=' + keyword, { method: 'GET', mode: 'cors' })
                                    .then(res => res.json())

const searchInput = document.getElementById('search');
const suggestList = document.getElementById('suggest-list');

const keyword = Rx.Observable.fromEvent(searchInput, 'input');
const selectItem = Rx.Observable.fromEvent(suggestList, 'click');

const render = (suggestArr = []) => suggestList.innerHTML = suggestArr.length > 0 ? suggestArr.map(item => '<li>'+ item +'</li>').join('') : '<li>'+ '无查询结果' +'</li>'

keyword
  .debounceTime(100)
  .switchMap(
    e => getSuggestList(e.target.value),
    (e, res) => res[1]
  )
  .subscribe(list => render(list))

selectItem
  .filter(e => e.target.matches('li'))
  .map(e => e.target.innerText)
  .subscribe(text => { 
      searchInput.value = text !== '无查询结果' ? value : 'No Result';
      render();
  })

```

![imgn](http://haoqiao.qiniudn.com/rxautocomplete1.gif)

`window系列operators`

* window
* windowCount
* windowTime
* windowToggle
* windowWhen

>window 是会把元素拆分出来放到新的observable 


```

// window 要传入一个observable，每当这个observable 送出元素时，就会把正在处理的observable 所送出的元素放到新的observable 中并送出

var click = Rx.Observable.fromEvent(document, 'click');
var source = Rx.Observable.interval(1000);
var example = source.window(click);

example
  .switch()
  .subscribe(console.log);
// 0
// 1
// 2
// 3
// 4
// 5 ...

```

`windowToggle`

>windowToggle 不像window 只能控制内部observable 的结束，windowToggle 可以传入两个参数，第一个是开始的observable，第二个是一个callback 可以回传一个结束的observable

```

var source = Rx.Observable.interval(1000);
var mouseDown = Rx.Observable.fromEvent(document, 'mousedown');
var mouseUp = Rx.Observable.fromEvent(document, 'mouseup');

var example = source
  .windowToggle(mouseDown, () => mouseUp)
  .switch();
  
example.subscribe(console.log);

```

`groupBy`

>把相同条件的元素拆分成一个Observable

```

var source = Rx.Observable.interval(300).take(5);

var example = source
              .groupBy(x => x % 2);
              
example.subscribe(console.log);

// GroupObservable { key: 0, ...}
// GroupObservable { key: 1, ...}

```

```

var people = [
    {name: 'Anna', score: 100, subject: 'English'},
    {name: 'Anna', score: 90, subject: 'Math'},
    {name: 'Anna', score: 96, subject: 'Chinese' }, 
    {name: 'Jerry', score: 80, subject: 'English'},
    {name: 'Jerry', score: 100, subject: 'Math'},
    {name: 'Jerry', score: 90, subject: 'Chinese' }, 
];
var source = Rx.Observable.from(people)
						   .zip(
						     Rx.Observable.interval(300), 
						     (x, y) => x);

var example = source
  .groupBy(person => person.name)
  .map(group => group.reduce((acc, curr) => ({ 
	    name: curr.name,
	    score: curr.score + acc.score 
	})))
	.mergeAll();
	
example.subscribe(console.log);
// { name: "Anna", score: 286 }
// { name: 'Jerry', score: 270 }

```

#### 深入Operator

**延迟运算**

> 延迟运算:所有Observable 一定会等到订阅后才开始对元素做运算，如果没有订阅就不会有运算的行为

```

var source = Rx.Observable.from([1,2,3,4,5]);
var example = source.map(x => x + 1);

// Observable 还没有被订阅，所以不会真的对元素做运算

```

**渐进式取值**

普通函数是返回所有结果再进行下一轮操作

![imgn](http://haoqiao.qiniudn.com/l0HlPZeB9OvFu7QwE.gif)

>operators 都必须完整的运算出每个元素的返回值并组成一个结果返回

```

var source = [1,2,3];
var example = source
              .filter(x => x % 2 === 0) // 返回一个结果
              .map(x => x + 1) // 返回一个结果

// source.filter(...)就会返回一整个新阵列，再接下一个operator又会再返回一个新的阵列

```

`每一次的operator 的运算都会建立一个新的阵列，并在每个元素都运算完后返回这个新阵列`


虽然Observable 的operator 也都会回传一个新的observable，但因为元素是渐进式取得的关系，所以每次的运算是一个元素运算到底，而不是运算完全部的元素再返回

```

var source = Rx.Observable.from([1,2,3]);
var example = source
              .filter(x => x % 2 === 0)
              .map(x => x + 1)

example.subscribe(console.log);

内部执行逻辑：

送出1到filter被过滤掉
送出2到filter在被送到map转成3，送到observer console.log印出
送出3到filter被过滤掉


```

`每个元素送出后就是运算到底，在这个过程中不会等待其他的元素运算。这就是渐进式取值的特性`

![imgn](http://haoqiao.qiniudn.com/3o6ZtqrBfUyHvMDQ2c.gif)

## Subject

```

// observable 是可以多次订阅的,每次的订阅都建立了一个新的执行。

var source = Rx.Observable.interval(1000).take(3);

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

source.subscribe(observerA);
source.subscribe(observerB);

// "A next: 0"
// "B next: 0"
// "A next: 1"
// "B next: 1"
// "A next: 2"
// "A complete!"
// "B next: 2"
// "B complete!"


```

`有些情况下我们会希望第二次订阅source 不会从头开始接收元素，而是从第一次订阅到当前处理的元素开始发送，我们把这种处理方式称为组播(multicast)`

**手动建立subject**

```

// 我们用subject 订阅source 并把observerA 加到subject 中，一秒后再把observerB 加到subject，这时就可以看到observerB 是直接收1 开始，这就是组播(multicast)的行为

var source = Rx.Observable.interval(1000).take(3);

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subject = {
    observers: [],
    addObserver: function(observer) {
        this.observers.push(observer)
    },
    next: function(value) {
        this.observers.forEach(o => o.next(value))    
    },
    error: function(error){
        this.observers.forEach(o => o.error(error))
    },
    complete: function() {
        this.observers.forEach(o => o.complete())
    }
}

subject.addObserver(observerA)

source.subscribe(subject);

setTimeout(() => {
    subject.addObserver(observerB);
}, 1000);

// "A next: 0"
// "A next: 1"
// "B next: 1"
// "A next: 2"
// "B next: 2"
// "A complete!"
// "B complete!"


等价于：

var source = Rx.Observable.interval(1000).take(3);

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subject = new Rx.Subject()

subject.subscribe(observerA)

source.subscribe(subject);

setTimeout(() => {
    subject.subscribe(observerB);
}, 1000);

// "A next: 0"
// "A next: 1"
// "B next: 1"
// "A next: 2"
// "B next: 2"
// "A complete!"
// "B complete!"

```

#### Subject简介

`Subject 可以拿去订阅Observable(source) 代表他是一个Observer，同时Subject 又可以被Observer(observerA, observerB) 订阅，代表他是一个Observable。`

> Subject 同时是Observable 又是Observer

> Subject 会对内部的observers 清单进行组播(multicast)

#### Subject应用

> Subject 在内部管理一份observer 的清单，并在接收到值时遍历这份清单并送出值

```

var subject = new Rx.Subject();

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

subject.subscribe(observerA);
subject.subscribe(observerB);

subject.next(1);
// "A next: 1"
// "B next: 1"
subject.next(2);
// "A next: 2"
// "B next: 2"

```

这里我们可以直接用subject 的next 方法传送值，所有订阅的observer 就会接收到，又因为Subject 本身是Observable，所以这样的使用方式很适合用在某些无法直接使用Observable 的前端框架中，例如在React 想对DOM 的事件做监听

```

class MyButton extends React.Component {
    constructor(props) {
        super(props);
        this.state = { count: 0 };
        this.subject = new Rx.Subject();
        
        this.subject
            .mapTo(1)
            .scan((origin, next) => origin + next)
            .subscribe(x => {
                this.setState({ count: x })
            })
    }
    render() {
        return <button onClick={event => this.subject.next(event)}>{this.state.count}</button>
    }
}

```

`BehaviorSubject`

> BehaviorSubject 是用来呈现当前的值，而不是单纯的发送事件。BehaviorSubject 会记住最新一次发送的元素，并把该元素当作目前的值，在使用上BehaviorSubject 建构式需要传入一个参数来代表起始的状态

```

// BehaviorSubject 在建立时就需要给定一个状态，并在之后任何一次订阅，就会先送出最新的状态。其实这种行为就是一种状态的表达而非单存的事件，就像是年龄跟生日一样，年龄是一种状态而生日就是事件；所以当我们想要用一个stream 来表达年龄时，就应该用BehaviorSubject 。

var subject = new Rx.BehaviorSubject(0); // 0
var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

subject.subscribe(observerA);
// "A next: 0"
subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
    subject.subscribe(observerB); 
    // "B next: 3"
},3000)

```

`ReplaySubject`

> 在新订阅时重新发送最后的几个元素，这时我们就可以用ReplaySubject

```

var subject = new Rx.ReplaySubject(2); // 重复发送最后俩个元素
var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

subject.subscribe(observerA);
subject.next(1);
// "A next: 1"
subject.next(2);
// "A next: 2"
subject.next(3);
// "A next: 3"

setTimeout(() => {
    subject.subscribe(observerB);
    // "B next: 2"
    // "B next: 3"
},3000)

```

`AsyncSubject`

> 在subject结束后送出最后一个值

```

var subject = new Rx.AsyncSubject();
var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

subject.subscribe(observerA);
subject.next(1);
subject.next(2);
subject.next(3);
subject.complete();
// "A next: 3"
// "A complete!"

setTimeout(() => {
    subject.subscribe(observerB);
    // "B next: 3"
    // "B complete!"
},3000)

```

#### Observable and Subject

`multicast`

>multicast 可以用来挂载subject 并回传一个可连结(connectable)的observable

```

var source = Rx.Observable.interval(1000)
             .take(3)
             .multicast(new Rx.Subject());

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

source.subscribe(observerA); // subject.subscribe(observerA)

source.connect(); // source.subscribe(subject)

setTimeout(() => {
    source.subscribe(observerB); // subject.subscribe(observerB)
}, 1000);

```

`必须真的等到执行connect()后才会真的用subject订阅source，并开始送出元素，如果没有执行connect()observable是不会真正执行的。`

```


var source = Rx.Observable.interval(1000)
             .do(x => console.log('send: ' + x))
             .multicast(new Rx.Subject()); // 無限的 observable 

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subscriptionA = source.subscribe(observerA);

var realSubscription = source.connect();

var subscriptionB;
setTimeout(() => {
    subscriptionB = source.subscribe(observerB);
}, 1000);

setTimeout(() => {
    subscriptionA.unsubscribe();
    subscriptionB.unsubscribe(); 
    // 虽然A,B退订，但是时间流还是继续执行
}, 5000);

setTimeout(() => {
    realSubscription.unsubscribe();
    // 这里才会真正的退订
}, 7000);

```

`refCount`

>建立一个只要有订阅就会自动connect 的observable

```

var source = Rx.Observable.interval(1000)
             .do(x => console.log('send: ' + x))
             .multicast(new Rx.Subject())
             .refCount();

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subscriptionA = source.subscribe(observerA); // 当source 一被observerA 订阅时(订阅数从0 变成1)，就会立即执行并发送元素


var subscriptionB;
setTimeout(() => {
    subscriptionB = source.subscribe(observerB);

}, 1000);

setTimeout(() => {
    subscriptionA.unsubscribe(); // 订阅减一    subscriptionB.unsubscribe(); // 订阅为0，停止发送
}, 5000);

```

`publish`

> 等价于 multicast(new Rx.Subject())

```

var source = Rx.Observable.interval(1000)
             .publish() 
             .refCount();
             
// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.Subject()) 
//             .refCount();

var source = Rx.Observable.interval(1000)
             .publishReplay(1) 
             .refCount();
             
// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.ReplaySubject(1)) 
//             .refCount();


var source = Rx.Observable.interval(1000)
             .publishBehavior(0) 
             .refCount();
             
// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.BehaviorSubject(0)) 
//             .refCount();

var source = Rx.Observable.interval(1000)
             .publishLast() 
             .refCount();
             
// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.AsyncSubject(1)) 
//             .refCount();

```


`share`

> 等价于  publish + refCount 


```

var source = Rx.Observable.interval(1000)
             .share();
             
// var source = Rx.Observable.interval(1000)
//             .publish() 
//             .refCount();

// var source = Rx.Observable.interval(1000)
//             .multicast(new Rx.Subject()) 
//             .refCount();

```

## Scheduler

### Scheduler简介

Scheduler 控制一个observable 的订阅什么时候开始，以及发送元素什么时候送达，主要由以下三个元素所组成

```

Scheduler 是一个对象结构。它知道如何根据优先级或其他标准来储存并执行任务。
Scheduler 是一个执行环境。它意味着任务何时何地被执行，比如像是立即执行、在回调(callback)中执行、setTimeout 中执行、animation frame 中执行
Scheduler是一个虚拟时钟。它透过now()这个方法提供了时间的概念，我们可以让任务在特定的时间点被执行。

```

```

// Scheduler 会影响Observable 开始执行及元素送达的时机

var observable = Rx.Observable.create(function (observer) {
    observer.next(1);
    observer.next(2);
    observer.next(3);
    observer.complete();
});

console.log('before subscribe');
observable.observeOn(Rx.Scheduler.async) // 设为 async
.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
console.log('after subscribe');

```


# 项目中的RxJs
 
在项目中RxJs可以通过库的形式引用，也可以引用结合了框架的组合。

通过之前的学习，对RxJs有了一定的了解。对我而言RxJS最好的应用场景就是复杂的UI交互。

而且在学习RxJS的资料中，很多典型的Demo都是:

* 拖拽交互
* Auto Complete
* 等等

利用RxJS能把我们以前需要写很多判断，很多逻辑的UI交互都简化了，通过它自带的一套Stream的用法，可以利用更少的代码完成以前的复杂的工作量，提供了开发效率。

RxJS同时能应用在组件状态管理中，可以参考[Reactive 视角审视 React 组件](https://www.zybuluo.com/bornkiller/note/840550) 

在React中，内部通过`setState`管理状态。状态的变更可以依赖RxJS流,在需要的Response中`setState`即可。

其他方案可以自行根据项目需求加入，需要就引入，不需要就不要加，不要为RxJS而RxJS.

还要注意的是RxJS的操作符非常强大，但是数量极多，因此一开始开发入门的时候先设计好逻辑再去查文档。

官方的[example](https://github.com/Reactive-Extensions/RxJS/tree/master/examples)有很多例子可以参考应用。


# 认识一下 redux-observable

> redux-observable，则是通过创建epics中间件，为每一个dispatch添加相应的附加效果。相较于thunk中间件，使用redux-observable来处理异步action，有以下两个优点：

不需要修改reducer，我们的reducer可以依然保持简单的纯函数形态。
epics中间件会将action封装成Observable对象，可以使用RxJs的相应api来控制异步流程。

比起`redux-thunk`,`redux-observable`能够强有力的支持更为复杂的异步逻辑。用更少的代码来实现需求。


# 将 redux-observable 融入项目

如果有之前项目有`redux-thunk`基础的，只需要考虑将负责的异步逻辑提取出来即可。

> 使用redux-thunk 处理简单异步逻辑，然后使用 redux-observable 处理复杂情况

因为	`redux-observable`也只是一个中间件，它提供了RxJs对异步逻辑的强大支撑，因此它和原有的 `react全家桶`并不会起到冲突，

关于更多细节可以去[官方中文文档](redux-observable-cn.js.org)查看


# 总结

通过几天的学习，对RxJS有了一定的了解，之后就是将其应用到实际项目中。

# 资料

> 学习操作符的时候可以对照弹珠图

[Rx Observables 的交互弹珠图 ](http://rxmarbles.com/)

> [Learn RxJS 的中文版](https://rxjs-cn.github.io/learn-rxjs-operators)

> [redux-observable中文文档](https://redux-observable-cn.js.org/docs/basics/Epics.html)

