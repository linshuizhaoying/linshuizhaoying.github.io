---
layout: post
title: 点击页面其他区域导航条隐藏
category: 技术
tags: [CFF讨论,VUE.JS,弹框,遮罩]
keywords: 前端,资料,学习,vue
description: 
---

## 起因

上次参加的CFF会议之后有个会后微信群，今天里面小伙伴出题：

```
问题描述,点击顶部的导航icon(三横杠),显示导航内容.再点击消失.现在交互设计要求点击页面其他区域.导航条需要隐藏起来,怎么实现?

```
如图：

![imgn](http://img.haoqiao.me/2e34c3da924e0e3e8ae6daa9b71ee30e.png)

于是一下子炸出了不少潜水鱼，于是我凑了个热闹，提出用`透明遮罩`来解决。

群里各种各样的方案:

```
1.遍历父节点判断
2.平时常见的灰色遮盖层换成透明的(同理)
3.弹出导航的时候 在 document 加个点击事件 判断下一次的点击事件的 目标 event.target 如果目标不是导航 就把导航藏起来 
4.=-=这里不给看
```

在提出了了透明遮罩来解决之后，小伙伴提出了质疑：

```
这个不可取.用户点击头部的链接需要点击两次
```

然后学识浅薄的我被问住了。这时候有小伙伴给出了解决方案:

```
header{z9}wrapper{z8}。wrapper的存在并不影响header的事件触发啊？如果点其他区域要求有响应（比如跳转），那菜单也就没有收起的必要了把？
```

于是我就先放到了任务列表，今晚回来实践了一下。

### 透明遮罩

试验环境是我自己之前搭建的基于vue.js的自动化流程。因此是基于vue的-0-，毕竟我试验完了可以以后用到项目里.

不过思路是相同的,具体代码改一改相信都能用上。

自己画一个菜单挺麻烦，于是codepen扒之。

先把index.vue搭起来:

```
<style lang="sass">
/* Important styles */
#toggle {
  display: block;
  width: 28px;
  height: 30px;
  margin: 30px auto 10px;
}

#toggle span:after,
#toggle span:before {
  content: "";
  position: absolute;
  left: 0;
  top: -9px;
}
#toggle span:after{
  top: 9px;
}
#toggle span {
  position: relative;
  display: block;
}

#toggle span,
#toggle span:after,
#toggle span:before {
  width: 100%;
  height: 5px;
  background-color: #888;
  transition: all 0.3s;
  backface-visibility: hidden;
  border-radius: 2px;
}

/* on activation */
#toggle.on span {
  background-color: transparent;
}
#toggle.on span:before {
  transform: rotate(45deg) translate(5px, 5px);
}
#toggle.on span:after {
  transform: rotate(-45deg) translate(7px, -8px);
}
#toggle.on + #menu {
  opacity: 1;
  visibility: visible;
}

/* menu appearance*/
#menu {
  position: relative;
  color: #999;
  width: 200px;
  padding: 10px;
  margin: auto;
  font-family: "Segoe UI", Candara, "Bitstream Vera Sans", "DejaVu Sans", "Bitstream Vera Sans", "Trebuchet MS", Verdana, "Verdana Ref", sans-serif;
  text-align: center;
  border-radius: 4px;
  background: white;
  box-shadow: 0 1px 8px rgba(0,0,0,0.05);
  /* just for this demo */
  opacity: 0;
  visibility: hidden;
  transition: opacity .4s;
  z-index: 20;
}
#menu:after {
  position: absolute;
  top: -15px;
  left: 95px;
  content: "";
  display: block;
  border-left: 15px solid transparent;
  border-right: 15px solid transparent;
  border-bottom: 20px solid white;
}
ul, li, li a {
  list-style: none;
  display: block;
  margin: 0;
  padding: 0;
}
li a {
  padding: 5px;
  color: #888;
  text-decoration: none;
  transition: all .2s;
}
li a:hover,
li a:focus {
  background: #1ABC9C;
  color: #fff;
}


/* demo styles */
body { margin-top: 3em; background: #eee; color: #555; font-family: "Open Sans", "Segoe UI", Helvetica, Arial, sans-serif; }
p, p a { font-size: 12px;text-align: center; color: #888; }

</style>

<template>

	<a href="#menu" id="toggle"><span></span></a>

	<div id="menu">
	  <ul>
	    <li><a href="#home">Home</a></li>
	    <li><a href="#about">About</a></li>
	    <li><a href="#contact">Contact</a></li>
	  </ul>
	</div>

    <overlay v-if="showlayer" show="{{@ showlayer }}">
		</overlay>

</template>

<script>

	module.exports = {
	  ready:function(){
	  	var that = this;
			var theToggle = document.getElementById('toggle');

			// based on Todd Motto functions
			// http://toddmotto.com/labs/reusable-js/

			// hasClass
			function hasClass(elem, className) {
				return new RegExp(' ' + className + ' ').test(' ' + elem.className + ' ');
			}
			// addClass
			function addClass(elem, className) {
			    if (!hasClass(elem, className)) {
			    	elem.className += ' ' + className;
			    }
			}
			// removeClass
			function removeClass(elem, className) {
				var newClass = ' ' + elem.className.replace( /[\t\r\n]/g, ' ') + ' ';
				if (hasClass(elem, className)) {
			        while (newClass.indexOf(' ' + className + ' ') >= 0 ) {
			            newClass = newClass.replace(' ' + className + ' ', ' ');
			        }
			        elem.className = newClass.replace(/^\s+|\s+$/g, '');
			    }
			}
			// toggleClass
			function toggleClass(elem, className) {
				var newClass = ' ' + elem.className.replace( /[\t\r\n]/g, " " ) + ' ';
			    if (hasClass(elem, className)) {
			        while (newClass.indexOf(" " + className + " ") >= 0 ) {
			            newClass = newClass.replace( " " + className + " " , " " );
			        }
			        elem.className = newClass.replace(/^\s+|\s+$/g, '');
			    } else {
			        elem.className += ' ' + className;
			    }
			}

			theToggle.onclick = function() {
				 that.showlayer = true;
			   toggleClass(this, 'on');
			   return false;
			}
	  },
			
    data: function(){
      return {
        showlayer: false
      }
        
    },
		components: {
      'overlay': require('../../components/project/overlay.vue'),
		}
	}
</script>

```

然后把之前写的overlay.vue拿来改一改用：

```
<template>
  <div class="overlay">
    <div class="overlay-mask" v-on="click: show=false"></div>
    <div class="overlay-inner" v-on="click: show=false">
      <content></content>
    </div>
  </div>
</template>

<script>
  module.exports = {
    props: ['show'],
    ready:function(){

      // hasClass
      function hasClass(elem, className) {
        return new RegExp(' ' + className + ' ').test(' ' + elem.className + ' ');
      }
      // addClass
      function addClass(elem, className) {
          if (!hasClass(elem, className)) {
            elem.className += ' ' + className;
          }
      }
      // removeClass
      function removeClass(elem, className) {
        var newClass = ' ' + elem.className.replace( /[\t\r\n]/g, ' ') + ' ';
        if (hasClass(elem, className)) {
              while (newClass.indexOf(' ' + className + ' ') >= 0 ) {
                  newClass = newClass.replace(' ' + className + ' ', ' ');
              }
              elem.className = newClass.replace(/^\s+|\s+$/g, '');
          }
      }
      
      function toggleClass(elem, className) {
        var newClass = ' ' + elem.className.replace( /[\t\r\n]/g, " " ) + ' ';
          if (hasClass(elem, className)) {
              while (newClass.indexOf(" " + className + " ") >= 0 ) {
                  newClass = newClass.replace( " " + className + " " , " " );
              }
              elem.className = newClass.replace(/^\s+|\s+$/g, '');
          } else {
              elem.className += ' ' + className;
          }
      }

      var that = this;
      var theToggle = document.getElementById('toggle');
      $( ".overlay-inner" ).on( "click",function( event ) {
        toggleClass(theToggle, 'on');
      });

    }
  };
</script>

<style>
  .overlay {
    position: fixed;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    z-index: 9;
    height: 100%;
    width: 100%;
    background-color: white;
    opacity: 0.1;
    overflow-y: scroll;
    -webkit-overflow-scrolling: touch;
  }
  .overlay-mask {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    z-index: 10;
    cursor: pointer;
  }
  .overlay-mask:after {
    content: '×';
    position: absolute;
    top: 16px;
    right: 18px;
    color: #ccc;
    font: 500 24px/1 "Helvetica Neue", "Arial", sans-serif;
  }
  .overlay-inner {
    position: absolute;
    top: 0;
    left: 50%;
    width: 860px;
    height: 100%;
    margin-left: -430px;
    z-index: 11;
  }
</style>

```

只要把overlay的z-index小于#menu的大于最底层.你会发现有几个函数我重复了,在实际应用中抽出来就好了。

### 弹出导航的时候 在 document 加个点击事件 判断下一次的点击事件的 目标 event.target 如果目标不是导航 就把导航藏起来 

于是去stackoverflow直接找到例子。

```
$(document).on('click', function(e) {
   if (e.target.id == 'div1') {
       alert('Div Clicked !!');
   } else {
       $('#div1').hide();
   }

})
```

### 第四种方法

这种看起来挺高大上的，也是提问者自己给出的方案:

```
可以用一个支持focus伪类的元素坐钩子。初始element+nav hidden,获取焦点后element:focus+nav show.失去焦点 自动隐藏掉。

剩下就是要在html５标签中寻找支持focus伪类的元素。

需要写个脚本遍历一下html５的元素，哪些支持focus伪类。

```

这种具体没接触过，不过据说a标签是支持的。

### 延生

还有很多思路，比如有人提出用call来解决,如果有其它思路可以一起分享。


