---
layout: post
title: 打造属于自己的在线前端调试工具
category: 技术
tags: [js,设计模式,jsfiddle,Realtime Playground,折腾,积累,Vue.js,Vue,Webpack]
keywords: 前端,资料,学习,javascript
description: 
---

# 前言

很久很久以前,当第一次接触codepen的时候,整个人被惊艳到。后来接触了jsfiddle，实乃在线调试的最佳利器。国内类似应用还有runjs。但是这些都需要联网。当某些时候只想快速的写点demo测试的时候，不得不自己搭配一个环境载入一些东东，然后才开始写demo例子。而且最近几天又重新阅读了[张鑫旭大神博客](www.zhangxinxu.com/wordpress/)，里面直接在文章内部显示的例子让人觉得非常人性化。因此，打造一个自己的在线前端调试工具就添加到了todo列表(事实上优先级最高的todo列表里面还有一个每天晚上花时间重构的应用QAQ)。然后显而易见这是一个”大工程“，尤其对于最近比较拖延还作死想到一出写一出的作者君而言。因此这篇前言是写在还未动工之前。。。

# 正文

在知乎上也有看到问[jsfiddle](https://jsfiddle.net/)这类在线web调试器的原理，然而其实这在google中搜索可以发现很早以前就有人贴出了原理。而且还是买一赠多的方式。

这里贴出参考资料:


[How to Write Your Own JSFiddle (In 15 Minutes or Less)](https://websanova.com/blog/jquery/how-to-write-your-own-jsfiddle-in-15-minutes-or-less)


[Building Your Own HTML, CSS, JS Realtime Playground](http://codetheory.in/building-your-own-html-css-js-realtime-playground/)

然后在动手之前想了想，如此明显的单页面应用，怎么能不用大vue来实现呢？毕竟后续还需要增加代码格式化，自动补全什么的功能，因此妥妥的上vue。

然后我们来整理一下思路:

```
1.首先不考虑加载外部js
2.三个textarea用来作为html,css,js输入，一个iframe作为最终输出。
3.将三个textarea里面的内容分别以传入iframe里
4.iframe重新渲染

```

乱七八糟的讲了一大堆,来直接看我们第一个初级版本的代码：

```

<style>
textarea{
  width: 200px;
  height: 100px;
}
</style>

<template>

  <section>
    <div id="html">
      <h3>HTML</h3>
      <textarea name="html"></textarea>
    </div>
    <div id="css">
      <h3>CSS</h3>
      <textarea name="css"></textarea>
    </div>
    <div id="js">
      <h3>JavaScript</h3>
      <textarea name="js"></textarea>
    </div>
  </section>
  
  <!-- Sandboxing -->
  <section id="output">
    <iframe></iframe>
  </section>
  <button v-on:click="render()">Run</button>

</template>

<script>

module.exports = {
  data:{
  
  },
  ready:function(){

  },
  methods: {
   render:function(){
      (function() {
        
        // Base template
        var base_tpl =
            "<!doctype html>\n" +
            "<html>\n\t" +
            "<head>\n\t\t" +
            "<meta charset=\"utf-8\">\n\t\t" +
            "<title>Test</title>\n\n\t\t\n\t" +
            "</head>\n\t" +
            "<body>\n\t\n\t" +
            "</body>\n" +
            "</html>";
        
        var sourceCode = function() {
          var html = $('#html textarea').val(),
              css = $('#css textarea').val(),
              js = $('#js textarea').val(),
              src = '';
          
          // HTML
          src = base_tpl.replace('</body>', html + '</body>');
          
          // CSS
          css = '<style>' + css + '</style>';
          src = src.replace('</head>', css + '</head>');
          
          // Javascript
          js = '<script>' + js + '<\/script>';
          src = src.replace('</body>', js + '</body>');
          
          return src;
        };
        
        var render = function() {
          var source = sourceCode();
          
          var iframe = document.querySelector('#output iframe'),
              iframe_doc = iframe.contentDocument;
          
          iframe_doc.open();
          iframe_doc.write(source);
          iframe_doc.close();
        };
        render();
      }());
   }
  },
  components: {

  }
}
</script>

```


测试代码:

```
html:

<div class="test">123xxx</div>

css:

.test{
color:red;
}

js:

var deleteLink = document.querySelectorAll('.test');

for (var i = 0; i < deleteLink.length; i++) {
    deleteLink[i].addEventListener('click', function(event) {
      alert("ok")
  });
}

```

# 结尾


来贴张成果图:

![imgn](http://img.haoqiao.me/jsfiddleone.png)

所以最基础的版本我们已经有了，剩下的就是布局调整，添加新功能，增强体验等等操作了。欲知后事如何,我们下回分解。


