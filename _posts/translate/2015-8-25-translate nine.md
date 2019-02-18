---
layout: post
title: A Content-switching Component Built 3 Ways (三种方式制作内容切换页面组件）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://www.sitepoint.com/content-switching-component-built-three-ways/)

### 作者: Scott O`Hara

### 译者: 临水照影


## 正文之前

此文相当于基础巩固文。我只是好奇css的版本应该怎么写-0-


## 正文

不久前，我的一个朋友通过`<select>`元素制作了一个切换页面的UI组件。她的代码正常运行，但现在她想用用jqeury来制作，因此她向我求助能够帮她优化。

虽然不是完整的代码，我创建了一个她试图想要完成的例子：
    
    <div class="select-area">
      <select name="choose" id="choose" class="input-select">
        <option value="nul" selected>Make a Selection</option>
        <option value="opt1">Option 1</option>
        <option value="opt2">Option 2</option>
        <option value="opt3">Option 3</option>
      </select>
    </div>

    <section class="jqueryOptions opt1">
      <div class="content">
        <h2>Option 1 Content</h2>
        <p>
          ...
        </p>
      </div>
    </section>

    <section class="jqueryOptions opt2">
      <div class="content">
        <h2>Option 2 Content</h2>
        <p>
          ...
        </p>
      </div>
    </section>

    <section class="jqueryOptions opt3">
      <div class="content">
        <h2>Option 3 Content</h2>
        <p>
          ...
        </p>
      </div>
    </section>

在优化之后，这是我们用toggle来展示选择内容块状态的Jquery代码：
    
    $(function() {
      $('.jqueryOptions').hide();

      $('#choose').change(function() {
        $('.jqueryOptions').slideUp();
        $('.jqueryOptions').removeClass('current-opt');
        $("." + $(this).val()).slideDown();
        $("." + $(this).val()).addClass('current-opt');
      });
    });
    
    
## 这是怎么回事？
默认的，整个jquery函数查找样式名为`jqueryOptions`的内容块，然后隐藏它们。

当用户改变`select`输入框的选择选项时，这个函数先用Jquery的`slideUp()`将潜在打开内容块的关闭然后用`slideDown`将选择的选项指向的内容打开。

所以

    <option value="opt1">Option 1</option>

匹配
    
    <section class="options opt1">
      ...
    </section>

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/9xk9zv8q/)


## 为什么不离开Jquery？

Jquery的解决方案有个问题就是我们为这么一个小功能需要包含Jquery库（98kb）。我们可以做的更好

不改变原来的标记内容，让我们来用javascript和css来达到同样的效果。

首先我们需要想想该怎么样来完成这个效果：
    
    内容块需要被默认隐藏
    当被选择时，选中块显示
    我们在新的选项被选中时需要隐藏任何打开的内容块

在代码中可能有一些附加片段，但是这是我们需要牢记在心的三个要点。当然，默认隐藏内容块我们可以用css来设置。隐藏我们只需要完成剩下的两个。


## 创建纯javascript版本

开始，让我们创建一些让我们接下来工作更加轻松的变量
    
    var selectInput = document.getElementById('choose'),
    panels = document.querySelectorAll('.options'),
    currentSelect,
    i;

现在我们有了一个能更加方便访问输入框的`selectInput`变量。代表不同内容的面板panels，代表当前选择状态的currentSelect，和一个迭代变量。

接下来我们需要写些帮助我们关注从项目列表中取得子项的方法。

首先我们需要一个能够当新选项被选择时候隐藏其他内容的方法：
    
    function clearShow() {
      for ( i = 0; i < panels.length; i++ ) {
        panels[i].classList.remove('show');
      }
    }
    
`clearShow()` 这个方法就是做这些事情的。首先，从我们的` panels` 变量中取得内容并循环迭代删除` show` 这个样式。

` show` 这个样式让我们的内容在页面中可见。

现在我们需要让我们选择的面板显示：
    
    function addShow(showThis) {
      var el = document.getElementsByClassName(showThis);
      for ( i = 0; i < el.length; i++ ) {
         el[i].classList.add('show');
       }
    } 

`addShow`方法接受一个`showThis`参数然后添加`show`样式到这个节点。现在我们依旧需要一些方法将`showThis`的值传给`addShow（）`
    
    function vUpdate() {
      currentSelect = selectInput.value;

      clearShow();
      addShow(currentSelect);
    }

    selectInput.addEventListener('change', vUpdate);
    
`vUpdate()`方法在任何选择输入框被更新的时候执行。所以，当`vUpdate()`方法运行时，它执行以下操作：
    
    获取selectInput的值然后储存到currentSelect的变量
    执行clearShow方法
    执行addShow方法
    
这是这个[Demo](http://jsfiddle.net/linshuizhaoying/9xk9zv8q/1/)

在我们继续进行之前，如果你选择了一个选项然后刷新整个页面，这个值可能会留在input但是面板不会显示。

我们可以用下面的代码来修补：
    
    if (selectInput.value !== 'nul') {
      currentSelect = selectInput.value;
      addShow(currentSelect);
    }
 
最后，如果你想要支持ie9或者更低版本，我们不能用`classList()`，可以用下面的代替。
    
    function addClass(elm, newClass) {
        elm.className += ' ' + newClass;
    }

    function removeClass(elm, deleteClass) {
      elm.className = elm.className.replace(new RegExp("\\b" + deleteClass + "\\b", 'g'), '    ').trim();
      /* the RegExp here makes sure that only
     the class we want to delete, and not
     any other potentially matching strings 
     get deleted.
      */
    }

## 能只用CSS来实现吗？

我做了很多组件，我喜欢尝试用纯css来实现它们。然而Jquery和javascript可以基于同样的布局，但是css需要修改。

与之前的版本相比最大也是最重要的就是`select`需要被完全代替。

我们将用`box/radio button`的hack来重建`select`

这是我们的新`select`元素：
    
    <input type='checkbox' class="invis" id="open_close" />
    <input type='radio' name='opts' class="invis" id="opt1" />
    <input type='radio' name='opts' class="invis" id="opt2" />
    <input type='radio' name='opts' class="invis" id="opt3" />

    <header class="header-base">
      <div class="content">
        <p>
          Choose an Option
        </p>

    <div class="select-area">
      <label for="open_close" class="input-select">
        ...
      </label>

      <ul class="select-options">
        <li>
          <label for="opt1">Option 1</label>
        </li>
        <li>
          <label for="opt2">Option 2</label>
        </li>
        <li>
          <label for="opt3">Option 3</label>
        </li>
      </ul>
    </div>

      </div>
    </header>

ul无序列表将作为我们新的`select`元素

`opt1`、`opt2`、`opt3`标签通过相应的ID改变`radio buttons`的选择状态，在CSS中，根据哪个`radio buttons`被选择，对应的面板将被显示：
    
    #opt1:checked ~ main .opt1,
    #opt2:checked ~ main .opt2,
    #opt3:checked ~ main .opt3 {
      display: block;
      height: 100%;
      overflow: visible;
      visibility: visible;
    }

这里用到了很多css细节，你看在[这里](http://www.sitepoint.com/you-can-do-that-with-css/)或者[这里](http://www.scottohara.me/article/morph-button-updated.html)了解更多信息。

我们想要页面在选项被按下时能快速更新。但是这里有一个问题，`select`元素和内容块都基于相同的`radio buttons`，点击`select`会让内容消失。不仅如此，如果`select`选项被打开，我们点击其中一个。选项会消失，内容会显示。

为了解决这个UX缺陷，我增加了
    
    <input type="checkbox" class="invis" id="open_close" />。
    
现在`select`元素和当前内容可以在同时被打开。但这里有一个问题就是点击选项时，选择不关机，除非再点一次。但我觉得这是一个更好的体验，为了帮助提高选择元素需要被再次点击关闭的注意力，我改变了点击时候显示向下箭头为X：
    
    #open_close:checked ~ .header-base .select-area .select-options {
      opacity: 1;
      visibility: visible;
    }

    #open_close:checked ~ .header-base .select-area:after {
      border: none;
      content: 'X';
      top: -24px;
    }

另一个我想重写`select`的特性是当一个新的选项被选择，主`select`元素的内容更改为对应选项。用`:before`能帮助我们实现这一点。
    
    .input-select:before {
      content: 'Make a Selection';
    }

    #opt1:checked ~ .header-base .input-select:before {
      content: 'Option 1';
    }

    #opt2:checked ~ .header-base .input-select:before {
      content: 'Option 2';
    }

    #opt3:checked ~ .header-base .input-select:before {
      content: 'Option 3';
    }


这是[纯css Demo](http://jsfiddle.net/linshuizhaoying/9xk9zv8q/2/)

## 总结

对于你选择哪一个方案都要注意权衡各项优点和缺点，致力于提供最佳的用户体验。

## 译者

javascript和jquery内容平平，css这块的确学到了，然后就是作者在css那块提供的学习资料很值得学习。










