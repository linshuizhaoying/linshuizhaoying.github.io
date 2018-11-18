---
layout: post
title: 百度前端学院之个人总结
tags: [学习,私货,前端,环境]
keywords: Javascript,代码,前端,学习总结
description: 
---

## 前言

最后一个任务写到开头路由之后猛然发现和自己想写的一个项目有点冲突，又因为没有多大动力去继续就暂停了，然后准备开始写小结，当然写小结什么的也要有创意，这次我是分散开来，当然主体还是从时间顺序来。

## 正文

首先我们来看现在的所有[任务](http://ife.baidu.com/task/all)

![imgn](http://img.haoqiao.me/ife1.png)

![imgn](http://img.haoqiao.me/ife2.png)

![imgn](http://img.haoqiao.me/ife3.png)

这么多任务我也不细细展开，我就从之前看到的来讲。主要还是Js这块，不讲css是因为这块还是要多看别人的写法然后自己去写。

因为很多任务都是个人根据自己喜欢去挑着写，这次总结我大概主要把其它自己没写的看下别人的完成。

首先是`javascript与树`，这个任务要求完成一个类似如图效果的树状结构。

![imgn](http://7xrp04.com1.z0.glb.clouddn.com/task_2_25_1.jpg)

需求如下：

```
		节点的折叠与展开
		允许增加节点与删除节点
		按照内容进行节点查找，并且把找到的节点进行特殊样式呈现，如果找到的节点处于被父节点折叠隐藏的状态，则需要做对应的展开

```

先不看别人的代码自己想想这种情况要怎么处理，首先节点折叠展开肯定是样式的改变，增加节点和删除节点很简单就是简单的dom节点操作。根据内容查找节点需要遍历。然后判断。基本思路有了看别人的代码，我就挑评价比较高的代码来看-0-，

[原地址](http://ife.baidu.com/review/detail?workId=5702)

[在线demo地址](https://leegent.github.io/ife2016/task25/task25.html)

```

/**
 * Created by mystery on 2016/3/29.
 * 感谢Steel Team，他们的代码给了我很大的启发
 */

// ==========================================封装TreeNode================================================
function TreeNode(obj) {
    this.parent = obj.parent;
    this.childs = obj.childs || [];
    this.data = obj.data || "";
    this.selfElement = obj.selfElement; // 访问对应的DOM结点
    this.selfElement.TreeNode = this;  // 对应的DOM结点访问回来
}
// 原型模式封装公共操作
TreeNode.prototype = {
    constructor: TreeNode,
    // 解耦样式操作，四个参数表示是否改变箭头、可见性、改为高亮、改为普通，后两个参数可省略
    render: function (arrow, visibility, toHighlight, deHighlight) {
        if (arguments.length < 3) {
            toHighlight = false;
            deHighlight = false;
        }
        if (arrow) {
            if (this.isLeaf()) { // 是个叶节点，设为空箭头
                this.selfElement.getElementsByClassName("arrow")[0].className = "arrow empty-arrow";
            }
            else if (this.isFolded()) { // 折叠状态，设为右箭头
                this.selfElement.getElementsByClassName("arrow")[0].className = "arrow right-arrow";
            }
            else { // 展开状态，设为下箭头
                this.selfElement.getElementsByClassName("arrow")[0].className = "arrow down-arrow";
            }
        }
        if (visibility) { // 改变可见性
            if (this.selfElement.className.indexOf("nodebody-visible") == -1) { // 本不可见，改为可见
                this.selfElement.className = this.selfElement.className.replace("hidden", "visible");
            }
            else { //改为不可见
                this.selfElement.className = this.selfElement.className.replace("visible", "hidden");
            }
        }
        if (toHighlight) { // 设为高亮
            this.selfElement.getElementsByClassName("node-title")[0].className = "node-title node-title-highlight";
        }
        if (deHighlight) { // 取消高亮
            this.selfElement.getElementsByClassName("node-title")[0].className = "node-title";
        }
    },
    // 删除结点，DOM会自动递归删除子节点，TreeNode递归手动删除子节点
    deleteNode: function () {
        var i;
        // 递归删除子节点
        if(!this.isLeaf()){
            for(i=0;i<this.childs.length;i++){
                this.childs[i].deleteNode();
            }
        }
        this.parent.selfElement.removeChild(this.selfElement);// 移除对应的DOM结点
        for (i = 0; i < this.parent.childs.length; i++) { // 从父节点删除该孩子
            if (this.parent.childs[i] == this) {
                this.parent.childs.splice(i, 1);
                break;
            }
        }
        // 调整父结点箭头样式
        this.parent.render(true, false);
    },
    // 增加子节点
    addChild: function (text) {
        if (text == null) return this;
        if (text.trim() == "") {
            alert("节点内容不能为空！");
            return this;
        }
        // 先增加子节点，再渲染自身样式
        // 若当前节点关闭，则将其展开
        if(!this.isLeaf() && this.isFolded()){
            this.toggleFold();
        }
        // 创建新的DOM结点并附加
        var newNode = document.createElement("div");
        newNode.className = "nodebody-visible";
        var newHeader = document.createElement("label");
        newHeader.className = "node-header";
        var newSymbol = document.createElement("div");
        newSymbol.className = "arrow empty-arrow";
        var newTitle = document.createElement("span");
        newTitle.className = "node-title";
        newTitle.innerHTML = text;
        var space = document.createElement("span");
        space.innerHTML = "&nbsp;&nbsp;";
        var newDelete = document.createElement("img");
        newDelete.className = "deleteIcon";
        newDelete.src = "images/delete.png";
        var newAdd = document.createElement("img");
        newAdd.className = "addIcon";
        newAdd.src = "images/add.png";
        newHeader.appendChild(newSymbol);
        newHeader.appendChild(newTitle);
        newHeader.appendChild(space);
        newHeader.appendChild(newAdd);
        newHeader.appendChild(newDelete);
        newNode.appendChild(newHeader);
        this.selfElement.appendChild(newNode);
        // 创建对应的TreeNode对象并添加到子节点队列
        this.childs.push(new TreeNode({parent: this, childs: [], data: text, selfElement: newNode}));
        // 渲染自身样式
        this.render(true, false);
        return this; // 返回自身，以便链式操作
    },
    // 展开、收拢结点
    toggleFold: function () {
        if (this.isLeaf()) return this; // 叶节点，无需操作
        // 改变所有子节点的可见状态
        for (var i=0;i<this.childs.length;i++)
            this.childs[i].render(false, true);
        // 渲染本节点的箭头
        this.render(true, false);
        return this; // 返回自身，以便链式操作
    },
    // 判断是否为叶结点
    isLeaf: function(){
        return this.childs.length == 0;
    },
    // 判断结点是否处于折叠状态
    isFolded: function(){
        if(this.isLeaf()) return false; // 叶结点返回false
        if(this.childs[0].selfElement.className == "nodebody-visible") return false;
        return true;
    }
};
//=======================================以上是封装TreeNode的代码=============================================

//=============================================事件绑定区===============================================
// 创建根节点对应的TreeNode对象
var root = new TreeNode({parent: null, childs: [], data: "前端攻城狮", selfElement: document.getElementsByClassName("nodebody-visible")[0]});
// 为root绑定事件代理，处理所有节点的点击事件
addEvent(root.selfElement, "click", function (e) {
    var target = e.target || e.srcElement;
    var domNode = target;
    while (domNode.className.indexOf("nodebody") == -1) domNode = domNode.parentNode; // 找到类名含有nodebody前缀的DOM结点
    selectedNode = domNode.TreeNode; // 获取DOM对象对应的TreeNode对象
    // 如果点在节点文字或箭头上
    if (target.className.indexOf("node-title") != -1 || target.className.indexOf("arrow") != -1) {
        selectedNode.toggleFold(); // 触发toggle操作
    }
    else if (target.className == "addIcon") { // 点在加号上
        selectedNode.addChild(prompt("请输入子结点的内容："));
    }
    else if (target.className == "deleteIcon") { // 点在减号上
        selectedNode.deleteNode();
    }
});

// 给root绑定广度优先搜索函数，无需访问DOM，返回一个搜索结果队列
root.search = function (query) {
    var resultList = [];
    // 广度优先搜索
    var queue = []; // 辅助队列，顺序存储待访问结点
    var current = this;
    // 当前结点入队
    queue.push(current);
    while (queue.length > 0) {
        // 从“待访问”队列取出队首结点访问，并将其所有子节点入队
        current = queue.shift();
        // 还原当前结点颜色
        current.render(false, false, false, true);
        // 读取当前结点data
        if (current.data == query) resultList.push(current); //找到了
        // 将当前结点的所有孩子节点入“待访问”队
        for (var i=0;i<current.childs.length;i++) {
            queue.push(current.childs[i]);
        }
    }
    return resultList;
};
// 搜索并显示结果
addEvent(document.getElementById("search"), "click", function () {
    var text = document.getElementById("searchText").value.trim();
    if(text == "") {
        document.getElementById("result").innerHTML = "请输入查询内容！";
        return;
    }
    // 执行搜索
    var resultList = root.search(text);
    // 处理搜索结果
    if (resultList.length == 0) {
        document.getElementById("result").innerHTML = "没有查询到符合条件的结点！";
    }
    else {
        document.getElementById("result").innerHTML = "查询到" + resultList.length + "个符合条件的结点";
        // 将所有结果结点沿途展开，结果结点加粗红色展示
        var pathNode;
        for (var x = 0;x<resultList.length;x++) {
            pathNode = resultList[x];
            pathNode.render(false, false, true, false);
            while (pathNode.parent != null) {
                if (pathNode.selfElement.className == "nodebody-hidden") pathNode.parent.toggleFold(); // 若是收拢状态，则展开
                pathNode = pathNode.parent;
            }
        }
    }
});
// 清除搜索结果
addEvent(document.getElementById("clear"), "click", function () {
    document.getElementById("searchText").value = "";
    root.search(null); // 清除高亮样式
    document.getElementById("result").innerHTML = "";
});
//==================================================================================================

//=======================================Demo展示区==================================================
//动态生成Demo树
root.addChild("技术").addChild("IT公司").addChild("谈笑风生");
root.childs[0].addChild("HTML5").addChild("CSS3").addChild("JavaScript").addChild("PHP").addChild("Node.JS").toggleFold();
root.childs[0].childs[4].addChild("JavaScript").toggleFold();
root.childs[1].addChild("百度").addChild("腾讯").addChild("大众点评").toggleFold();
root.childs[2].addChild("身经百战").addChild("学习一个").addChild("吟两句诗").toggleFold();
root.childs[2].childs[2].addChild("苟利国家生死以").toggleFold();
//初始化查询Demo值
document.getElementById("searchText").value = "JavaScript";
//==================================================================================================

// 跨浏览器兼容的工具函数
function addEvent(element, type, handler) {
    if (element.addEventListener) {
        element.addEventListener(type, handler);
    }
    else if (element.attachEvent) {
        element.attachEvent("on" + type, handler);
    }
    else {
        element["on" + type] = handler;
    }
}

```

对照之前的想法来看，首先之前想的删除节点的确是操作dom，但是还需要考虑到如果父节点里面还有节点需要递归操作删除。

比较有意思的搜索是采用广度优先搜索函数,我们来看定义

```

广度优先遍历（Breadth-First Traversat）：从图中某个顶点v出发，在访问了v之后依次访问v的各个未曾访问过的邻接点，然后分别从这些邻接点出发依次访问它们的邻接点，并使“先被访问的顶点的邻接点”先于“后被访问的顶点的邻接点”被访问，直至图中所有已被访问的顶点的邻接点都被访问到。若此时图中尚有顶点未被访问，则另选图中一个未曾被访问的顶点作起始点，重复上述过程，直至图中所有顶点都被访问到为止。

```

![imgn](http://img.haoqiao.me/ife-bfs.png)

这是图的算法.从以上代码中可以看到作者对代码进行封装的程度很高.

接下来是`听指令的小方块`这个任务,这个任务很有意思,更有意思的有大神将其写的远远超过预期。

任务是实现一个小方块可以根据命令移动。如图:

![imgn](http://7xrp04.com1.z0.glb.clouddn.com/task_2_36_1.jpg)

我们来看锟斤拷的代码，这在当时被很多人惊叹，因为不仅完成了指定任务，更重要的是加了一个绘图功能。

[demo地址](http://qiuxiang.github.io/boxbot/)

我们先来看基础的移动模块


```
var BOTTOM = 0
var LEFT = 90
var TOP = 180
var RIGHT = 270

/**
 * @constructor
 * @param {string} selector
 */
var BoxbotBot = function (selector) {
  this.element = document.querySelector(selector)
  this.init()
}

BoxbotBot.prototype.init = function () {
  this.element.style.left = this.element.clientWidth + 'px'
  this.element.style.top = this.element.clientHeight + 'px'
  this.element.style.transform = 'rotate(0deg)'
}

/**
 * 转换方向
 *
 * @param {int} direction
 */
BoxbotBot.prototype.turn = function (direction) {
  var ROTATE_MAP = {
    0: {0:0, 90: 90, 180: 180, 270: -90},
    90: {90: 0, 180: 90, 270: 180, 0: -90},
    180: {180: 0, 270: 90, 0: 180, 90: -90},
    270: {270: 0, 0: 90, 90: 180, 180: -90}
  }
  this.element.style.transform = 'rotate(' +
    (this.getCurrentAngle() + ROTATE_MAP[this.getCurrentDirection()][direction]) + 'deg)'
}

/**
 * 获取当前旋转角度
 *
 * @return {int}
 */
BoxbotBot.prototype.getCurrentAngle = function () {
  var match = this.element.style.transform.match(/rotate\((.*)deg\)/)
  if (match) {
    return parseInt(match[1])
  } else {
    return 0
  }
}

/**
 * 旋转
 *
 * @param {int} angle
 */
BoxbotBot.prototype.rotate = function (angle) {
  this.element.style.transform = 'rotate(' + (this.getCurrentAngle() + angle) + 'deg)'
}

/**
 * 获取当前方向
 *
 * @return {int}
 */
BoxbotBot.prototype.getCurrentDirection = function () {
  var angle = this.getCurrentAngle() % 360
  return angle >= 0 ? angle : angle + 360
}

/**
 * 获取指定方向上偏移的位置
 *
 * @param {int} direction
 * @param offset
 * @returns {[int]}
 */
BoxbotBot.prototype.getOffsetPosition = function (direction, offset) {
  var position = {0: [0, 1], 90: [-1, 0], 180: [0, -1], 270: [1, 0]}[direction]
  return [position[0] * offset, position[1] * offset]
}

/**
 * 获取当前位置便宜量
 * 
 * @param {string} direction left|top
 * @returns {int}
 */
BoxbotBot.prototype.getCurrentOffset = function (direction) {
  var offset = this.element.style[direction]
  if (offset) {
    return parseInt(offset.replace('px', ''))
  } else {
    return 0
  }
}

/**
 * 获取当前位置
 *
 * @returns {[int]}
 */
BoxbotBot.prototype.getCurrentPosition = function () {
  return [
    Math.round(this.getCurrentOffset('left') / this.element.clientWidth),
    Math.round(this.getCurrentOffset('top') / this.element.clientHeight)]
}

/**
 * 以当前位置为基准，获取指定方向上的位置
 *
 * @param {int|null} [direction] 如果为 null 则取当前方向
 * @param {int} [offset=0]
 * @returns {[int]}
 */
BoxbotBot.prototype.getPosition = function (direction, offset) {
  if (direction == null) {
    direction = this.getCurrentDirection()
  }
  offset = offset || 0
  var offsetPosition = this.getOffsetPosition(direction, offset)
  var currentPosition = this.getCurrentPosition()
  return [currentPosition[0] + offsetPosition[0], currentPosition[1] + offsetPosition[1]]
}

/**
 * 跳到指定位置
 *
 * @param {[int]} position
 * @param {boolean} [turn] 是否旋转方向
 */
BoxbotBot.prototype.goto = function (position, turn) {
  if (turn) {
    var currentPosition = this.getCurrentPosition()
    var distance = [position[0] - currentPosition[0], position[1] - currentPosition[1]]
    if (distance[0] < 0) {
      this.turn(LEFT)
    } else if (distance[0] > 0) {
      this.turn(RIGHT)
    } else if (distance[1] < 0) {
      this.turn(TOP)
    } else if (distance[1] > 0) {
      this.turn(BOTTOM)
    }
  }
  this.element.style.left = position[0] * this.element.clientWidth + 'px'
  this.element.style.top = position[1] * this.element.clientHeight + 'px'
}

/**
 * 朝指定方向移动
 *
 * @param {int} direction
 * @param {int} step
 */
BoxbotBot.prototype.move = function (direction, step) {
  this.goto(this.getPosition(direction, step))
}


```

可以看出编码习惯非常好，每段代码之前都有说明。

对于旋转角度，他的写法很独特，像我之前写的话是解析输入命令然后再旋转，他是直接正则style里面的`transform: rotate(270deg);` 然后再旋转。

尤其值得学习的是类似下面这段

```

/**
 * 获取当前位置便宜量
 * 
 * @param {string} direction left|top
 * @returns {int}
 */
BoxbotBot.prototype.getCurrentOffset = function (direction) {
  var offset = this.element.style[direction]
  if (offset) {
    return parseInt(offset.replace('px', ''))
  } else {
    return 0
  }
}


```

从这段里我们可以看出优良代码风格给我们会带来很大遍历，尤其是用原型模式封装，调用起来很方便。

```

/**
 * 递归实现的深度优先搜索算法
 *
 * @param path
 * @param target
 * @param visited
 * @returns {[[int]]}
 */
BoxbotFinder.prototype.dfs = function (path, target, visited) {
  if (!visited) {
    visited = {}
    path = [path]
  }

  var current = path[path.length - 1]
  if (current[0] == target[0] && current[1] == target[1]) {
    path.shift()
    return path
  }

  var offsets = this.createOffsets([target[0] - current[0], target[1] - current[1]])
  for (var i = 0; i < offsets.length; i += 1) {
    var next = [offsets[i].x + current[0], offsets[i].y + current[1]]
    var positionKey = next[0] + '-' + next[1]

    if (this.isAvailable(next) && !visited[positionKey]) {
      path.push(next)
      visited[positionKey] = next

      var result = this.dfs(path, target, visited)
      if (result) {
        return result
      }

      path.pop()
    }
  }
}

/**
 * 广度优先搜索
 *
 * @param {[int]} from
 * @param {[int]} to
 */
BoxbotFinder.prototype.bfs = function (from, to) {
  var offsets = [[0, 1], [-1, 0], [0, -1], [1, 0]];
  var queue = [new FinderNode(null, from)]

  while (true) {
    var current = queue.shift()

    if (current.position[0] == to[0] && current.position[1] == to[1]) {
      return current.getPath()
    }

    for (var i = 0; i < offsets.length; i += 1) {
      var position = [current.position[0] + offsets[i][0], current.position[1] + offsets[i][1]]

      if (this.isAvailable(position) && !current.isParent(position)) {
        var node = new FinderNode(current, position)

        current.children.push(node)
        queue.push(node)
      }
    }
  }
}


```

而且他将寻路算法独立出来，对于新手而言非常友好，不需要考虑上下文。

## 结尾

其实还有很多很棒的写法，尤其是有es6的各种版本写法，感觉都值得学习，更多的是官方给了不少学习资料，大家都可以去仔细看看。关于百度前端技术学院可能就先告一段落。要开始做自己之前想做的一些东西了。

