---
layout: post
title: 捣鼓了一个Jquery拼图游戏插件
tags: [插件,私货,Jquery]
keywords: Jquery,代码插件,plugin,拼图,canvas,前端,学习总结
description: 
---

# 前言

最近捣鼓一个送人的生日作品，因此开发了一个拼图游戏与抽奖轮盘。基于canvas，部分代码是网上的思路。所有代码可以在github上查看[Puzzle Game Plugin](https://github.com/linshuizhaoying/puzzle-Game-Plugin)。

# 正文

一开始考虑写拼图游戏的时候就想着应该有类似的，于是Google一番发现的确有，但是发现并不适合自己。比如网上的版本大多数是需要键盘控制的，而我的需求是需要在手机上戳戳戳。然后网上的版本有些是需要你用一个空白小方块来取代其中一块。因此后来我想了想先写一个起手：

```
;(function($, window, document,undefined) {
    //定义构造函数
    var Puzzle = function(ele, opt) {
        this.$element = ele,
        this.defaults = {

        },
        this.options = $.extend({}, this.defaults, opt)
    }
    //定义方法
    Puzzle.prototype = {
        puzzleinit: function() {

        }
    }
    //在插件中使用对象
    $.fn.myPlugin = function(options) {
        //创建的实体
        var puzzle = new Puzzle(this, options);
        //调用其方法
        return puzzle.puzzleinit();
    }
})(jQuery, window, document);


$('#puzzle').myPlugin({

});

```

然后按照自己想法先列了几个功能：

```
1.默认定义一张世界图/其他图作为正面，一张人物照片作为背面

3.通过点击两张小图使它们交换位置

2.正面图分割小图并打乱

4.当小图的位置组合成正面图时，底部按钮变正常，点击正面图翻转显示背面图。下一个关卡跳出。

```

之后就是往里面填充内容。

提一下几个重要的方法，一个是打碎大图为小图：

```
         //重绘画布
            function drawTiles() {
                if(!solved){
                    //绘制之前先把画布擦一擦
                    context.clearRect( 0 , 0 , boardSize , boardSize );
                    for (var i = 0; i < tileCount; ++i) {
                        for (var j = 0; j < tileCount; ++j) {
                          var x = boardParts[i][j].x;
                          var y = boardParts[i][j].y;
             //第一个参数和其它的是相同的，都是一个图像或者另一个 canvas 的引用。
             //其它8个参数最好是参照右边的图解，前4个是定义图像源的切片位置和大小，
             //后4个则是定义切片的目标显示位置和大小。
                          context.drawImage(frontImg, x * tileSize, y * tileSize, tileSize, tileSize,
                                i * tileSize, j * tileSize, tileSize, tileSize);
                        }
                    }
                }
            }
```

交换拼图:

```

          //交换拼图
            function slideTile(toLoc, fromLoc) {
              if (!solved) {
                var temp = new Object;
                temp.x = boardParts[toLoc.x][toLoc.y].x;
                temp.y = boardParts[toLoc.x][toLoc.y].y;
                boardParts[toLoc.x][toLoc.y].x = boardParts[fromLoc.x][fromLoc.y].x;
                boardParts[toLoc.x][toLoc.y].y = boardParts[fromLoc.x][fromLoc.y].y;
                boardParts[fromLoc.x][fromLoc.y].x = temp.x;
                boardParts[fromLoc.x][fromLoc.y].y = temp.y;
                toLoc.x = fromLoc.x;
                toLoc.y = fromLoc.y;
                console.log(boardParts);
                checkSolved();
              }
            }

```

具体代码大家可以clone[Puzzle Game Plugin](https://github.com/linshuizhaoying/puzzle-Game-Plugin)。

## 结尾

整个插件花了大概两天写完。虽然简陋但是满足了我的使用需求也就不再继续。开始继续下一个项目。


