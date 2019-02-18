---
layout: post
title: Thinking Outside the DOM (Dom之外的思考，数据采集和校验）
category: 翻译
tags: [国外文章翻译]
keywords: 学习，资料，程序，前端
description: 
---

## 前言

### 原文来源:[这里](http://www.sitepoint.com/thinking-outside-dom-composed-validators-data-collection/)

### 作者: Christian Johansen

### 译者: 临水照影


## 正文之前
这篇文章好像有姊妹篇，不过这篇更能引起我的兴趣。


## 正文

在第一篇文章中，我们讨论了大多数javascript库的普遍问题：紧密耦合的代码。（那篇文章点[这里](http://www.sitepoint.com/thinking-outside-dom-concepts-setup/)）然后，我向你介绍了分离观念的好处。为了证明观念，我们开始开发了一个不仅限于表单的表单验证系统，它甚至可以在DOM之外的地方工作。

这本章我们将介绍如何编写验证器，怎样采集数据和如何报错，最后我会给你一个包含本文例子的Github的仓库链接。

### 编写校验器

在上一篇文章我们介绍了验证单个字段的验证器，用一个规则一对一的验证字段是可以工作的，但是当有大量例子的时候需要更深层次的考虑。你可以通过一个很长的正则表达式来验证一个Email地址，但是这只能告诉你的用户Email是否被接受。一个更好的方法是对email的各个部分进行验证。

它可能长这样：

    var rules = [
      pattern('email', /@/, 'Your email is missing an @'),
      pattern('email', /^\S+@/, 'Please enter the username in your email address',
      // ...
    ];

当这个开始工作，它可能为一个邮箱地址而产生多个错误信息。这要求我们手动重复判断每个部分是否有意义。即使我们现在还没讨论关于错误信息的渲染，拥有一个关于多个验证最后只显示第一个错误信息抽象的概念是很重要的。事实证明，这是`&&`操作符的语义。下面这个验证器将把多个验证方法作为参数，然后依次应用直到找到失败的那项。
    
    function and() {
      var rules = arguments;

      return function (data) {
        var result, l = rules.length;

        for (var i = 0; i < l; ++i) {
          result = rules[i](data);
          if (result) {
            return result;
          }
        }
      };
    }


现在我们可以用一种方法来表现我们的邮箱验证而且它只会显示一个错误信息提示。

    var rules = [and(
      pattern('email', /@/, 'Your email is missing an @'),
      pattern('email', /^\S+@/, 'Please enter the username in your email address',
      // ...
    )];

这个可以被编写成一个独立的验证。
    
    function email(id, messages) {
      return and(
        pattern('email', /@/, messages.missingAt),
        pattern('email', /^\S+@/, messages.missingUser)
        // ...
      );
    }

当我们想要只有在某些条件符合时检测某些情况，为了解决这个问题，我们需要介绍`when`方法

    function when(pred, rule) {
      return function (data) {
        if (pred(data)) {
          return rule(data);
        }
      };
    }

正如你所见，`when`是一个验证，就像`required`。你可以传入一个验证数据的方法，如果它返回`true`，我们执行了验证，否则`when`认为数据是正确的。

我们还需要一个匹配模式：

    function matches(id, re) {
      return function (data) {
        return re.test(data[id]);
      };
    }

除了它不是一个验证，它已经很接近我们的模式验证。另外值得一提的是这些方法都很简约，而且它们一起应用在实际情况比单个应用更加高效。通过这最后一个拼图，我们已经可以获得一个完整的验证器

    function email(id, messages) {
      return and(
        pattern(id, /@/, messages.missingAt),
        pattern(id, /^\S+@/, messages.missingUser),
        pattern(id, /@\S+$/, messages.missingDomain),
        pattern(id, /@\S+\.\S+$/, messages.missingTLD),
        when(matches(id, /@hotmail\.[^\.]+$/),
          pattern(id, /@hotmail\.com$/, messages.almostHotmail)
        ),
        when(matches(id, /@gmail\.[^\.]+$/),
          pattern(id, /@gmail\.com$/, messages.almostGmail)
        )
      );
    }

它可以这么应用：
   
    email('email', {
     missingAt: 'Missing @',
     missingUser: 'You need something in front of the @',
     missingDomain: 'You need something after the @',
     missingTLD: 'Did you forget .com or something similar?',
     almostHotmail: 'Did you mean hotmail<strong>.com</strong>?',
     almostGmail: 'Did you mean gmail<strong>.com</strong>?'
    });


这是Demo [Click Here](http://jsfiddle.net/linshuizhaoying/ytL0hm9n/)


### 提取数据

现在我们可以验证数据，为了解决我们的表单验证初始化我们需要从表单中将数据提取出来。首先，我们将

    <form action="/doit" novalidate>
      <label for="email">
        Email
        <input type="email" name="email" id="email" value="christian@cjohansen.no">
      </label>
      <label for="password">
        Password
        <input type="password" name="password" id="password">
      </label>
      <label class="faded hide-lt-pad">
        <input type="checkbox" name="remember" value="1" checked>
        Remember me
      </label>
      <button type="submit">Login</button>
    </form>


转变为
    
    {
      email: 'christian@cjohansen.no',
      password: '',
      remember: '1'
    }

在测试中这一步相当简单，但是它将请求访问DOM元素。接下来的测试例子长这样：


    describe('extractData', function () {
      it('fetches data out of a form', function () {
        var form = document.createElement('form');
        var input = document.createElement('input');
        input.type = 'text';
        input.name = 'phoneNumber';
        input.value = '+47 998 87 766';
        form.appendChild(input);

        assert.deepEqual(extractData(form), {'phoneNumber': '+47 998 87 766'});
      });
    });

我们来抽象它

    it('fetches data out of a form', function () {
      var form = document.createElement('form');
      addElement(
        form,
        'input',
        {type: 'text', name: 'phoneNumber', value: '+47 998 87 766'}
      );

      assert.deepEqual(extractData(form), {'phoneNumber': '+47 998 87 766'});
    });

我们将从表单中选择所有的输入来获取数据，比如`input`，`select`，`textarea`。然后提取`name`属性和当前的值。当然，还需要处理一些特殊值，比如复选框和单选按钮。这个主函数长这样:

    function extractData(form) {
      return getInputs(form).reduce(function (data, el) {
        var val = getValue[el.tagName.toLowerCase()](el);
        if (val) { data[el.name] = val.trim(); }
        return data;
      }, {});
    };

在这个片段中，`extractData`函数依靠 getInputs()函数。这是为了得到一组表单里的DOM元素作为参数。



## 错误报告
    
为了显示错误，我们需要设计一个函数来接受一个表单和一组错误信息。然而，这里有个挑战需要解决：为了避免在DOM里复制错误，这个方法需要保持一个状态，让它自己明白哪些错误已经渲染了。它还需要让每个错误在表单中都能在新的被渲染时销毁。
哪种方案更合适取决于你自己的选择。

我不会深入渲染的细节，下面是一个简单的例子

    function renderErrors(form, errors) {
      removeErrors(form);
      errors.forEach(function (error) {
        renderError(form, error);
      });
    }

为了渲染错误，我们需要找到对应的input，然后在它的右侧插入元素。我们只渲染第一个错误提醒。这是一个非常基础的渲染策略但是效果很好：
    
    
    function renderError(form, error) {
      var input = form.querySelector("[name=" + error.id + "]");
      var el = document.createElement("div");
      el.className = "error js-validation-error";
      el.innerHTML = error.messages[0];
      input.parentNode.insertBefore(el, input);
    }

在这段代码中，设定了两个class，前者只是为了style，后者作为一个内部的判断，在下面函数里删除

    function removeErrors(form) {
      var errors = form.querySelectorAll(".js-validation-error");

      for (var i = 0, l = errors.length; i < l; ++i) {
        errors[i].parentNode.removeChild(errors[i]);
      }
    }

[Demo Click Here](http://jsfiddle.net/linshuizhaoying/z8vmc68e/)


## 将它们整合在一起

我们现在需要以下功能：从DOM中读取，验证数据，渲染验证结果。我们需要高级的接口将它们绑在一起：

     validateForm(myForm, [
       required("login", "Please choose a login"),
       email("email", i18n.validation.emailFormat),
       confirmation("password", "password-confirmation", "Passwords don't match")
     ], {
       success: function (e) {
         alert("Congratulations, it's all correct!");
       }
     });

作为渲染，这种高等级接口可以很简单也可以很复杂。在代码中，validateForm()函数直到用户提交第一次submit才会进行验证。如果有错误，它会进入一种“智能验证模式”，错误如果被修复错误信息将会很快被移除。


完整的代码看[这里](http://jsfiddle.net/linshuizhaoying/yxwvvxny/)

这是该项目的[github地址](https://github.com/sitepoint-editors/validate)

##译者


最后直接给出的项目有点复杂，建议还是自己重新写一个比较好。





