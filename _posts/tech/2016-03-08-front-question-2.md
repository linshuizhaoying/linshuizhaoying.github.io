---
layout: post
title: 前端面试题错题集(二)
tags: [学习,私货,前端,面试]
keywords: Javascript,代码,CSS3,面试题,前端,学习总结
description: 
---

## 前言

今天是把面试题里的js部分总结一下。然后应该是收收心把es6给学了。。。

## 正文

```
用javascript实现用户登录验证的代码。

这个我一开始想到的是验证邮箱啊，判断是否空啊，然后密码要星号啊这类。
但是没想到一点。就是一开始显示的时候应该自动选择焦点到第一个输入内容。因此觉得还是有必要记录以下的。


<script language=javascript>
function checkSubmit()
{
    if ((document.form1.name.value)==”)
    {
        window.alert (‘姓名必须填写’);
        document.form1.name.select();
        document.form1.name.focus();
        return false;
    }
    else
        return true;
}
</script>
<form name=”form1″ onsubmit=”javascript:return checkSubmit()”>
<input type=”text” name=”name”>
</form>

```


```

请给Array本地对象增加一个原型方法，它用于删除数组条目中重复的条目(可能有多个)，返回值是一个包含被删除的重复条目的新数组。

一开始我自己是这么写的：
Array.prototype.getSingle = function(){
    var temp  = old;
    var newArray = [];
    function check(item){
       var flag = true;
      for(var i  = 0 ; i < newArray.length;i++){
            if(item == newArray[i]){
                flag = false;
           }
      }
     return flag;
    }
    for(var i = 0; i<= old.length;i++){
      if(check(temp[i])){
         newArray.push(temp[i]);
     }
   }
    return newArray;
}

很惭愧，写的时候发现自己忘了怎么从外部原型拿到数据了。。觉得用this好像不太多。。。然后就先用old来代替了。

看了下答案：
Array.prototype.distinct = function() {
    var ret = [];
    for (var i = 0; i < this.length; i++)
    {
        for (var j = i+1; j < this.length;) {   
            if (this[i] === this[j]) {
                ret.push(this.splice(j, 1)[0]);
            } else {
                j++;
            }
        }
     }
     return ret;
}
//for test
alert(['a','b','c','d','b','a','e'].distinct());

感觉还是答案写的简练。大致思路还是类似的就是循环判断。

```


```
请填充代码，使mySort()能使传入的参数按照从小到大的顺序显示出来。
function mySort() {
    var tags = new Array();//使用数组作为参数存储容器
    请补充你的代码
    return tags;//返回已经排序的数组
}
 
var result = mySort(50,11,16,32,24,99,57,100);/传入参数个数不确定
console.info(result);//显示结果

这个感觉应该考的就是冒泡。。。然而因为原生有sort的缘故我很久没自己写排序算法了。。。造成了这一题我跪了。。。

这题一共看到两种解法，一种就是原生sort传比较方法:
function mySort() {
    var tags = new Array();
    for(var i = 0;i < arguments.length;i++) {
        tags.push(arguments[i]);
    }
    tags.sort(function(compare1,compare2) {
        return compare1- compare2;
    });
    return tags;
}
 
var result = mySort(50,11,16,32,24,99,57,100);
console.info(result);

还有一种就是自己写冒泡排序
function mySort() {
    var tags = new Array();
    tags = Array.prototype.slice.call(arguments);//对参数进行分割
    for(var i=0; i<tags.length; i++) {遍历
        for(var j=i+1; j<tags.length; j++) {
            if(tags[i] > tags[j]) { //从小到大排序
                var temp = tags[i];
                tags[i] = tags[j];
                tags[j] = temp;
            }
        }
    }
    return tags;
}

```

```
输出对象中值大于2的key的数组

这个我一开始想到的是用for in 来遍历然后判断。后来看了下答案。

输出对象中值大于2的key的数组
var data = {a: 1, b: 2, c: 3, d: 4};
Object.keys(data).filter(function(x) { return 1 ;})
期待输出：[“c”,”d”]

我都快忘了还有Keys这个属性了。。。

Object.keys(data)返回的是data可被枚举的属性，为 ["a", "b", "c", "d"]，对这个数组中的每一个元素用filter方法过滤，data["x"]等价于data.x,就是表示data对象的x属性的值,注意这里的data["x"]中的x是带引号的

```

```
请列举js异步编程的方法

这里我一开始想到的是Promise 异步的话好像还有ajax，不过我知道考点是什么。后来看了答案是这样的：

回调函数，这是异步编程最基本的方法。
事件监听，另一种思路是采用事件驱动模式。任务的执行不取决于代码的顺序，而取决于某个事件是否发生。
发布/订阅，上一节的"事件"，完全可以理解成"信号"。
Promises对象，Promises 对象是CommonJS 工作组提出的一种规范，目的是为异步编程提供统一接口。


$.ajax, 异步加载请求，加载完成后执行回调。
Promise类。
promise = new Promise( function(resolve, reject) {
   // heavy function
    resolve('ok')
});
promise.then( function(data1,data2) {
     //here heavy function has been done
     //your code here
})
自定义回调
var heavyFunc = function(callback) {
    // heavy function here
    // a lot of time
    callback()
}
var callback = function() {
    // execute when heavy function is done
    // your code here
}
heavyFunc(callback)


```


```
验证邮箱
这题一看就是看考正则的。。。但我正则的功力在之前一个项目之后已经好久没使用，退化的只能看别人的正则啧啧称赞了。。。

var checkEmail  = function(email){
var preg = 
        "(^[a-zA-Z]|^[\\w-_\\.]*[a-zA-Z0-9])@(\\w+\\.)+\\w+$"，,
    pregObj  =new RegExp(preg);
    return pregObj.test(email);
}

```


```
请编写一段JavaScript脚本生成下面这段DOM结构。要求：使用标准的DOM方法或属性。
这个也是考基础的。我大概知道怎么写的，但是一下子让自己用最原始方法。

<div id=”example”>  
    <p class=”slogan”>淘！你喜欢</p>
</div>


 window.onload = function() {
            var div = document.createElement('div');
            div.id = "example";
            var p = document.createElement('p');
            p.className = "slogan";
            p.innerHTML = '淘！你喜欢';
            div.appendChild(p);
            document.body.appendChild(div);
        }

```

```
请编写一个通用的事件注册函数（请看下面的代码）。
这个很简单的。高程里的更详细。我就做下记录

function addEvent(element, type, handler)
{
    // 在此输入你的代码，实现预定功能
 
    if (element.addEventListener)
    {
        element.addEventListener(type, handler, false);
    }
    else if (element.attachEvent)     //for IE
    {
        element.attachEvnet(“on” + type, handler);
    }
    else
    {
        element[“on” + type] = handler;
    }
}



```

```

请编写一个JavaScript函数 parseQueryString，它的用途是把URL参数解析为一个对象，如：
var url = “http://www.taobao.com/index.php?key0=0&key1=1&key2=2.....”
var obj = parseQueryString(url);
alert(obj.key0)  // 输出0

平时看到这种我一般都是忽略的。。。因为实际项目中没怎么用过这种方式。不过看到不少面试题都要考。。估摸着也是考正则.

答案有两种方法:
第一种：
function parseQueryString ( name ){
  name = name.replace(/[\[]/,"\\\[").replace(/[\]]/,"\\\]");
  var regexS = "[\\?&]"+name+"=([^&#]*)";
  var regex = new RegExp( regexS );
  var results = regex.exec( window.location.href );
  if( results == null )
    return "";
  else
    return results[1];
}

第二种：
var url = "http://www.taobao.com/index.php?key0=0&key1=1&key2=2.............";
var obj = parseQueryString(url);
alert(obj.key2);
       
function parseQueryString(argu){
 
  var str = argu.split('?')[1];
  var result = {};
  var temp = str.split('&');
  for(var i=0; i<temp.length; i++)
  {
     var temp2 = temp[i].split('=');
     result[temp2[0]] = temp2[1];
  }
  return result;
}



```

## 结尾
  这部分面试题我觉得还是自己对有些不太重视导致自己知道大概思路却写的不怎么完整。因此真正去准备的时候应该把这些忽略的东西好好理顺一下。

