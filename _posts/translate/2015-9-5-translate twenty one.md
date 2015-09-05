---
layout: post
title:  Emoji Toggles! (表情符号切换）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](https://css-tricks.com/emoji-toggles/)

### 作者: Chris Coyier 


### 译者: 临水照影


## 正文之前

暑期的翻译计划可能以这篇作为结尾，以后如果看到优秀的文章，可能会在周末或者有空的时候翻译。但未来可能更注重自己对一些技术实践进行总结。可能不会再有类似效果教程翻译这种，取而代之的是应该是多个效果实践后组成一篇文章。翻译之旅亦是我的学习之旅。它的结束表明了我新的学习之路的开启。

## 正文

你知道`<label>`配合`for`属性可以匹配单选框，它可以切换选择和不选中。而且，用[CheckBox hack](https://css-tricks.com/the-checkbox-hack/) ，你可以做出很多有趣的效果。

下面的效果就是切换选择时显示不同的emoji。

![imgn](http://7s1say.com1.z0.glb.clouddn.com//emoji1.gif)

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/gjdao84r/)

这是改进版

![imgn](http://7s1say.com1.z0.glb.clouddn.com//%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202015-09-05%2010.15.34%20AM.png)

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/gjdao84r/1/)


输入本身是隐藏的，但是将绝对定位放在前面，这样点击任何地方都可以切换复选框。

以下代码为核心：
    
    .emoji-toggle {
      position: relative;
      .well { // the label
        cursor: pointer;
      }
      .toggle { // the checkbox
        appearance: none;
        background: transparent;
        position: absolute;
        width: 100%;
        height: 100%;
        cursor: pointer;
        z-index: 100; 

        // "off"
        ~.emoji:before { 
          content: "emoji unicode here";
          position: absolute;
          left: 0;
          top: -15px;
          font-size: 40px;
          z-index: 1;
          transition: 0.2s;
        }

        // "on"
        &:checked {
          ~.emoji:before {
            content: "different emoji unicode here";
            left: 100%;
            margin-left: -1em;
          }
        }

      }
    }

## Sass 版本

为了使用不同的版本，它需要改变4个伪元素。所以用sass做一个`mixin`可以减轻工作量。
    
    @mixin emojiType($leftEmoji, $rightEmoji, $leftLabel, $rightLabel) {
     .toggle {
       ~.emoji:before {
         content: $leftEmoji;
       }
       &:checked {
         ~.emoji:before {
           content: $rightEmoji;
         }
       }
        ~label {
         &:before {
           content: $leftLabel;
         }
         &:after {
           content: $rightLabel;
         }
       }
     }
   }

   // Usage
   .emoji-happy {
     @include emojiType(
       "\01F604", "\01F620", "Happy", "Mad"
     );
   }

## Emoj表情

[你可以在这里找到相关的unicode代码](http://apps.timwhitlock.info/emoji/tables/unicode)

## 结尾

注意emoji表情在不同浏览器/平台会有差异。







