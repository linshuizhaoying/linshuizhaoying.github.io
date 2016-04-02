---
layout: post
title: 百度前端学院之听指令的小方块（一）
tags: [学习,私货,前端,环境]
keywords: Javascript,代码,听指令的小方块,工厂,前端,学习总结
description: 
---

## 前言
这是做的第二个系列任务，只有空完成了前两项=-=，因此先记录一下。

## 正文
任务描述
		如图 ![imgn](http://7xrp04.com1.z0.glb.clouddn.com/task_2_33_1.jpg)
		，实现一个类似棋盘的格子空间，每个格子用两个数字可以定位，一个红正方形的DOM在这个空间内，正方形中的蓝色边表示这是他的正面，有一个input输入框
		在输入框中允许输入如下指令，按下按钮后，使得正方形做相应动作
		GO：向蓝色边所面向的方向前进一格（一格等同于正方形的边长）
		TUN LEF：向左转（逆时针旋转90度）
		TUN RIG：向右转（顺时针旋转90度）
		TUN BAC：向右转（旋转180度）
		移动不能超出格子空间

直接来看代码：

```
css:
*, *:before, *:after {
	-webkit-box-sizing:border-box;
	-moz-box-sizing:border-box;
	box-sizing:border-box;
}
.container {
	position:relative;
}
table {
	border-spacing:0;
	border-collapse:collapse;
	background-color:transparent;
}
tbody {
	top:0;
	left:0;
}

td{
	width:50px;
	height:50px;
	border-right:1px solid #aaa;
	border-bottom:1px solid #aaa;
	font-size:20px;
	font-weight:bold;
	text-align:center;
}

td:first-child {
	border-right:2px solid #333;
	border-bottom:0;
}
td:last-child {
	border-right:2px solid #333;
}
tr:first-child td {
	border-right:0;
	border-bottom:2px solid #333;
}
tr:last-child td {
	border-bottom:2px solid #333;
}
tr:first-child td:first-child {
	border-bottom:0;
}
tr:last-child td:first-child {
	border-bottom:0;
}

.square {
	position:absolute;
	top:300px;
	left:300px;
	width:50px;
	height:50px;
	background-color:#f00;
	transition:0.3s;
	-webkit-transition:0.3s;
}
.square-face {
	position:absolute;
	top:0;
	left:0;
	width:50px;
	height:15px;
	background-color:#00f;
}
.control {
	margin-top:15px;
	margin-left:45px;
}
.control input {
	margin:5px;
}



html:

    <div class="container">
        <table>
            <tbody id="boxContent">

            </tbody>
        </table>
        <div id="square" class="square">
            <div class="square-face"></div>
        </div>
        <div id="control" class="control">
            <input id="command" type="text">
            <input id="run" type="button" value="执行">
            <br>
            <input id="go" type="button" value="GO">
            <input id="left" type="button" value="Left">
            <input id="right" type="button" value="Right">
            <input id="back" type="button" value="Back">
        </div>
    </div>

js:
	    window.onload = function() {
	        createBoxContainer(); //创建面板
	        // 获取HTLM元素
	        square.draw(); //初始化，置于0，0

	        document.getElementById("run").onclick = function() {
	            var operate = document.getElementById("command").value; //获取输入框的值
	            switch (operate.toUpperCase()) {
                    case 'GO':
                        square.go();
                        break;
                    case 'TUN LEF':
                        square.changeFace('left');
                        break;
                    case 'TUN RIG':
                        square.changeFace('right');
                        break;
                    case 'TUN BAC':
						            square.changeFace('left');
						            square.changeFace('left');
                        break;
                }
	        }
	        document.getElementById("go").onclick = function() {
	            square.go();
	        }
	        document.getElementById("left").onclick = function() {
	            square.changeFace('left');
	        }
	        document.getElementById("right").onclick = function() {
	            square.changeFace('right');
	        }
	        document.getElementById("back").onclick = function() {

	            square.changeFace('left');
	            square.changeFace('left');

	        }
	    }

	    function createBoxContainer() {
	        var bg = document.getElementById("boxContent");
	        var tr = [];
	        for (var i = 0; i < 11; i++) {
	            tr[i] = document.createElement("tr"); // 创建11行tr
	            bg.appendChild(tr[i]);
	            var td = [];
	            for (var j = 0; j < 11; j++) {
	                td[j] = document.createElement("td"); // 创建11列td
	                if (i === 0 && j > 0) {
	                    td[j].innerHTML = j; // 标注列数
	                }
	                if (i > 0 && j === 0) {
	                    td[j].innerHTML = i; // 标注行数
	                }
	                tr[i].appendChild(td[j]);
	            }
	        }
	    }

	    var square = {
	        config: {
	            posx: 0, //定义方块的x坐标
	            posy: 0, //定义方块的y坐标
	            face: 'up' //蓝色条朝向，
	        },

	        changeFace: function(face) {
	            var that = this;
	            var turn = {
	                ["left"]: [-1],
	                ["right"]: [
	                    1
	                ]
	            }
	            var postion = {
	                [0]: [
	                    'up'
	                ],
	                [1]: [
	                    'right'
	                ],
	                [2]: [
	                    'down'
	                ],
	                [3]: [
	                    'left'
	                ],
	            };
	            var faceRotate = {
	                ["up"]: [
	                    0
	                ],
	                ["down"]: [
	                    2
	                ],
	                ["left"]: [
	                    3
	                ],
	                ["right"]: [
	                    1
	                ],
	            };


	            var square = document.getElementById("square");
	            var r = (parseInt(faceRotate[that.config.face]) + parseInt(turn[face]) + 4) % 4;
	            that.config.face = postion[r];
	            console.log(r);
	            square.style.transform = square.style.webkitTransform = square.style.msTransform = "rotate(" + (r * 90) + "deg)";
	        },
	        go: function() {
	            var that = this;
	            console.log(that.config.face);
	            if (that.config.face == 'up' && that.config.posy > 0) {
	                that.config.posy--; // square上移
	            } else if (that.config.face == 'right' && that.config.posx < 9) {
	                that.config.posx++; // square右移
	            } else if (that.config.face == 'down' && that.config.posy < 9) {
	                that.config.posy++; // square下移
	            } else if (that.config.face == 'left' && that.config.posx > 0) {
	                that.config.posx--; // square左移
	            } else {
	                return false;
	            }
	            that.draw();
	        },
	        draw: function() {
	            var that = this;
	            var square = document.getElementById("square");
	            square.style.top = that.config.posy * 50 + 50 + "px";
	            square.style.left = that.config.posx * 50 + 50 + "px";

	            console.log(that.config.posx + 'x');
	            console.log(that.config.posy + 'y');
	        }
	    }


```

## 结尾

这个代码比较简单就不仔细说明了。


