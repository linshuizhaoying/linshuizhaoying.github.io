---
layout: post
title: 前端开发——让算法"动"起来
category: 技术
tags: [学习,开发,前端,总结,数据结构,算法,source code]
keywords: 数据结构,算法,source code,前端
description: 
---

## 前言

上一篇介绍了比较简单数据结构和算法，但是很多情况下算法学习是比较枯燥的，但是非常庆幸的是我们作为前端开发可以自己找点乐子。比如，让算法"动"起来。

## 正文

当然在我们不清楚具体操作细节前我们可以先假设一下，我们能够用什么来实现。按照以前看过的排序动画我将其分为

```

1.Js操作Dom,再搭配简单的css
2.Canvas动画

```

之后在查资料的时候发现还有人用`d3`这个库来完成。

作为一个有(被)梦(坑)想(多)的前端，一开始就得考虑到如何实现套入多个算法，如果实现单步运行(可能的话还得有往回走的功能),如何实现动画速度的控制等等。

当然幻想那么多一下子实现是不现实的，我们得先找一个简单的例子来看看再一步步深入。

先来看下效果图

![imgn](http://haoqiao.qiniudn.com/sortanimation1.gif)

之后我们分析源码：


```
css:

#field {width:500px;height:510px;background:black;position:relative}
.bar {position:absolute;bottom:0;background:orange;border:1px solid brown;width:24px}

html:

<div id="field">
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
  <div class="bar"></div>
</div>

javascript:

!function(d){

  var bars = [].slice.call(d.querySelectorAll('.bar'));
  var arr = [8, 10, 3, 5, 6, 9, 2, 4, 7, 1];
  var state = [];

  var draw = function(){
    var bar, s;
    s = state.shift() || [];
    for(bar in bars){
      bars[bar].style.height = 25 * s[bar] + 'px';
      bars[bar].style.left = 25 * bar  + 'px';
    }
  }

  var sort = function(arr){
    for(var i = 0; i < arr.length; i++){
      for(var j = 0; j < arr.length - i - 1; j++){
        if(arr[j] > arr[j+1]){
          arr[j]   = arr[j] + arr[j+1];
          arr[j+1] = arr[j] - arr[j+1];
          arr[j]   = arr[j] - arr[j+1];
          			state.push(JSON.parse(JSON.stringify(arr)));
          			
          			
        }
      }
    }
  }
  
  sort(arr);
  setInterval(draw, 500);

}(document)


```

整个流程其实很清晰，但是其中部分代码让我疑惑了一段时间，经过google和问群里的朋友，终于解惑。我们先来理清这段代码的思路

```
首先这个动画是将大小宽度都定死了。以及排序数组的数量需要和html结构里bar的数量一致。

1.html的bar是长方体，它的宽是24px然后有个1px的border，因此在代码中动态改变left的时候需要设定为25.

2.js代码中用一个匿名立即函数包裹代码。
  bars = [].slice.call(d.querySelectorAll('.bar'));
这段将获取的nodelist转为成一个对象数组，这样方便对其中每个bar进行单独修改样式

3.设定一个state空数组来保存每一个状态,记住这才是动画的关键。

4.state.shift()临时像将数组模拟成队列，draw函数根据其第一个出列的内容来重新排列列表,在

setInterval(draw, 400)的配合下，就能形成一个动画排序。

5 sort函数和我们之前介绍的冒泡排序是一样的，只不过这里有一句

state.push(JSON.parse(JSON.stringify(arr)));

这句是核心，一看是乍看是不是很奇怪，为什么要JSON.stringify然后再JSON.parse。这里需要大家认真思考一下。

想想在哪里看过它？
深拷贝？
对，就是深拷贝。对于深拷贝不理解的我这里给出它的含义：

深拷贝是复制变量值，对于非基本类型的变量，则递归至基本类型变量后，再复制。

我这两天也整理过深拷贝，但是我还是一下子没理解为什么这里要这么写。

一开始我想偏差了，我一开始认为arr作为一个数字数组，对它进行深拷贝和用一个中间变量进行操作不是一样么，于是我加了这么几行代码

var temp = arr

state.push(temp);

然后 动画消失了，页面变成最后排好序的样子。

这时候群里有人提醒了我一句浅复制会修改原数组。我这才根据state.push反应过来。

在sort内部，每一个push都是为了保存交换后排序数组的状态，如果我用temp来代替它，那么state里面将全放着相同的最后排完序的状态。而JSON.parse(JSON.stringify(arr))对arr进行深复制，不会改动arr原数组，因此它就类似快照一样把每次排序的状态给push到state.然后配合setInterval一张一张的放映形成动画。


```

一个简单的排序动画其实里面也包含了不少有价值的内容。

回过头这么一看，这不是很容易套公(算)式(法)么

把我们之前学的插入排序拿来改改:

```

    function exchange(array, i, j) {
      var t = array[i];
      array[i] = array[j];
      array[j] = t;
    }




  var sort2 = function(numbers){
     for (var i = 0; i < numbers.length; i++) {
        /*
         * 当已排序部分的当前元素大于value，
         * 就将当前元素向后移一位，再将前一位与value比较
         */
     for (var j = i; j > 0 && numbers[j] < numbers[j - 1]; j--) {
          // If the array is already sorted, we never enter this inner loop!
          exchange(numbers, j, j - 1);
          state.push(JSON.parse(JSON.stringify(numbers)));
          console.log("此时数组:" + numbers)
        }
     }

   }
   
```

分分钟改变动画效果。

![imgn](http://haoqiao.qiniudn.com/sortanimation2.gif)

这样我们就完成目标中的一小步了。

然后之前考虑的单步运行和动画速度控制我们都可以改变相应参数来完成。

不过在继续深入之前我们先来看看如何用`canvans`来实现。

先来看效果

![imgn](http://haoqiao.qiniudn.com/sortanimation3.gif)

代码如下:

```

html:

<div id="restart">重新生成数据并排序</div>
<canvas id="canvas"><canvas>

css:

body {
  background-color: black;
  text-align: center;
}
#restart {
  color: white;
  font-family: monospace;
}	

javascript:

!function(){

  var canvas = document.getElementById('canvas'); 
  var data = [];
  canvas.width = window.innerWidth-30;
  canvas.height = window.innerHeight-35;
  CreateData(IntRandom(300, 100));
  Render();


  function Restart() {
    data = [];
    CreateData(IntRandom(300, 100));
  }

  function CreateData(val) {
    for(var i = 0; i <= val; i++) 
      data[i] = IntRandom(500, 10);
   
  }

  function BubbleSort() {
    var temp;
    for(var i = 0; i <= data.length-1; i++) {
      if(data[i] > data[i+1]) {
        temp = data[i];
        data[i] = data[i+1];
        data[i+1] = temp;
      }
    }
  }

  function Draw() {
    var posX = 0,
        posY = canvas.height;

    for(var i = 0; i <= data.length-1; i++) {
      c.fillStyle = RandomColor(i);
      c.fillRect(posX, canvas.height, 5, -data[i]);
      
      posY--;
      posX+=6;
    }
  }

  document.onclick = function() {
    Clear();
    Restart();
  };

  function Render() {
      requestAnimationFrame(Render);
      Clear();
      BubbleSort();
      Draw();
  }

  function Clear() {
    c.fillStyle = "black";
    c.fillRect(0, 0, canvas.width, canvas.height);
  }

  function RandomColor(i) {
    var n = Math.random() * 360;
    return "hsl("+parseInt(i)+", 100%, 50%)"
  }

function IntRandom(max, min) {
  return Math.floor(Math.random() * max + min);
}	

}()


```

这份代码其实是有缺陷的，不过没关系，在下面的分析中我们可以边看边改

```

1.首先是常规canvas操作，如果对canvas不上很熟悉的同学建议把高程相关部分刷一遍。
具体操作就是，获取整个浏览器屏幕长款，减去一部分作为画布的长宽，这是为了让生成的序列不会跟浏览器边缘贴合。
CreateData()这个方法就是随机生成一堆随机高度的长方形。

BubbleSort()就是常见的冒泡排序，但是在这里我们看到这只是一个冒泡算法，并没有像之前做一系列快照。(这里就涉及到了我们提到的缺陷)

draw()方法 从屏幕左侧坐标为0的点开始，这个posY其实毫无用处，因为我们的高度是根据之前随机生成一堆随机高度的长方形数组data来生成的。posX自加是为了保持间隔。

RandomColor() 这个方法根据高度来改变颜色。

之后是本代码的核心render()
之前我们看到操作dom的版本，动画效果是通过setInterval(draw, 500)来实现的，那么这里的动画效果哪里来？
我们可以看到这里用到了HTML5的API：

requestAnimationFrame

它的优势在于保证跟浏览器的绘制走，如果浏览设备绘制间隔是16.7ms，那就这个间隔绘制；如果浏览设备绘制间隔是10ms, 就10ms绘制。不会存在过度绘制的问题，动画不会掉帧。

这个从我们的效果图里也能看出动画的确很流畅。然而，这段代码是有问题的。问题在于它没有设置中止的标识，也就是它会不停刷新浏览器，时间久了将会卡住。而且细心的会发现之前我们看到的冒泡排序它只有一层循环。render()方法靠着循环调用硬生生把这些无序数组给堆成有序了。。。

而且太流畅的动画一气呵成，让我们无法仔细观察排序的经过，因此我们需要对其进行修改。


```

这里我们停下来思考一下，该如何改？

想想之前js操作dom版是怎么做的？我们同样可以套进去。
只需要把排序算法换成之前一样的。然后把render改成如下：

```

    var fps = 1; //每秒几帧

    var lastExecution = new Date().getTime();

    function Render() {
        if (state.length > 0) { //动画播放，没播完继续

            var now = new Date().getTime();

            if ((now - lastExecution) > (1000 / fps)) {
                Clear();
                Draw();
                lastExecution = new Date().getTime();
            }

            requestAnimationFrame(Render);

        }
    }

  

```

不过需要注意的是当你想重置requestAnimationFrame的时候，需要一开始就注明var stopId = requestAnimationFrame(Render);

然后配合cancelAnimationFrame(stopId)即可暂停继续。

最终效果如下:

![imgn](http://haoqiao.qiniudn.com/sortanimation4.gif)

有了以上基础你完全可以自己开始构建一个属于自己的排序动画。这篇就到这里。

## 结尾

所有代码和别的补充已经放在[github](https://github.com/linshuizhaoying/toss/tree/master/%E9%9D%A2%E8%AF%95/%E7%AE%97%E6%B3%95%E6%95%B4%E7%90%86)
而且会不断更新。有兴趣的可以去看看并动手敲一遍。



## 资料

[visualgo.net的排序动画](https://visualgo.net/sorting#)

[David Galles教授Canvas+JS实现排序动画](http://www.webhek.com/post/comparison-sort.html)


