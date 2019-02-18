---
layout: post
title: javascript 和 数据结构 (1)
category: 技术
tags: [JS，基础,javascript,数据结构,前端总结,基础]
keywords: 前端,资料,学习
description: 
---

## 前言
昨天粗略扫了一遍<<数据结构与算法javascript描述>>,感觉里面的每一章都有很多值得学习的内容。因此特意看书的时候做了书签。今天按照书签顺序来结合自己理解深入学习。

### 数组

首先是比较巧妙的末尾添加元素

```
var element = [1,2,3,4]
element[element.length] = 5; //[1,2,3,4,5]

```

这里标记一下pop()从数组末尾删除元素。shift()删除数组第一个元素。

splice()可以任意添加和删除元素。提供以下参数

```
1.起始索引(你希望开始添加元素的地方)
2.需要删除元素的个数(添加元素时设为0)
3.想要添加进数组的元素
```

sort()默认能对字符串数组排序。比如

```
var strings = ["a","xxx","v","b"]
strings.sort();//["a", "b", "v", "xxx"]
```

如果要对数字排序，需要传入判断函数。

```
function compare(n,m){
  return n-m;
}
var num = [1,4,67,2,4,8,2];
num.sort(compare);//[1, 2, 2, 4, 4, 8, 67]

```

#### 迭代器
forEach()

```
function square(n){
  console.log(n*n);
}

var num = [1,3,5,7,9];
num.forEach(square);// 1,9,25,49,81
```

every() //以前居然没听过有这个方法，不可思议

```
function check(n){
  return n % 2 == 0;
}

var num = [1,2,4,6,7];
var even = num.every(check);
if(even){
  console.log("全都能被2整除");
}else{
  console.log("不是全都能被2整除");
}
//不是全都能被2整除

```
这个方法用到合适的地方感觉能减少不少代码量。

some() //又是一个用都没用过的方法-0-

```
function check(n){
  return n % 2 == 0;
}

var num = [1,2,4,6,7];
var some = num.some(check);
if(some){
  console.log("只要一个元素能被2整除我就返回");
}else{
  console.log("只要没有元素能被2整除我就返回");
}

// 只要一个元素能被2整除我就返回

```
reduce() //话说我没用过的方法好多啊

```
function add (n,m){
  return n + m;
}

var num = [1,3,5,7,9];
var sum =  num.reduce(add);
console.log(sum);//25
```
也可用来拼接字符串。

map() 返回一个新数组，该数组的元素为对原有元素应用某个函数得到结果。

```
function curve(n){
  return n + 2;
}

var num = [1,2,3,4,5,6];
var newnum = num.map(curve);
console.log(newnum);//[3, 4, 5, 6, 7, 8]
```

也可用来拼接字符串

```
function first(word){
  return word[0];
}

var world = ["father","mother","i","love","you"]
var neww = world.map(first);
console.log(neww);//["f", "m", "i", "l", "y"]
console.log(neww.join(""));//fmily

```
还有一个filter,顾名思义，用来过滤

```
function check(n){
  return n % 2 == 0;
}

var num = [1,2,4,6,7];
var filter = num.filter(check);
console.log(filter);//[2, 4, 6]

```

## 最后

留给自己一个小作业,给定年份,输出所有月份,按星期格式输出。用上以上的所有的函数,可自行扩展功能。(提升,可以用二维数组,功能比如说可以是给某一天添加事件，给某几天统一添加事件)

这个可能要等蛮久我才会有时间去写,先添加到omnifocus里待办。


