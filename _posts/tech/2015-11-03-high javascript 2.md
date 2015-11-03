---
layout: post
title: 高性能javascript 读书笔记2
category: 技术
tags: [js,读书笔记,学习整理]
keywords: 前端,资料,学习,javascript
description: 
---

## 前言 
继续昨天的整理。

## 倒序循环的性能提高优化
一般来说，按照以前的认知，倒序和正序似乎是一样的。
但是我们看下面这几段代码:

```
for(var i = items.length;i--;){
  process(items[i]);
}

var j = i.items.length;
while(j--){
  process(items[i]);
}


var k  = items.length - 1;
do{
  process(items[i]);
}while(k--);


```

事实上这里的倒序并不像我们以前的那种写法，现在每个控制条件只是简单的和0比较，控制条件与true值比较时，任何非零数会自动转为true,而零值等同于false.`实际上，控制条件已经从两次比较(迭代次数少于总数？它是否是true？)减少到一次比较（它是true么）`

这个写法真的是简单粗暴啊。

## 利用Memoization来避免重复计算

这个只是提供一个思路,并记录一下大概写法。因为书中是对递归进行优化。其实这个在很多地方都能用到。我自己觉得这个对个人做项目的时候有很大启发。我们先来看下代码：

```
function momoize(func,cache){
  cache = cache ||{};
  
  var shell = function(arg{
    if(!cache.hasOwnProperty(arg)){
      cache[arg] = func(arg);
    }
    return cache[arg];
  };
  
  return shell;
}
```

## 记录代码运行时间

记录下来以后用来测试性能

```
var start = +new Date(),
    stop;

someLongProcess();

stop +=new Date();

console.log(stop - start);

```

## 允许你在程序中提取一个包含代码的字符串，然后动态执行它
一开始我以为只有eval和前几天刚学到的new Function

么想到

`setTimeout()和setInterval()也可以。`

直接来看例子

```
var n1 =1,n2 =2;

result = eval("n1 + n2");

sum = new Function("arg1","arg2","return arg1 + arg 2");

setTimeout("sum = n1 + n2",100);

setInterval("sum = n1 + n2",100)


```

##结尾
今天就到这里。

