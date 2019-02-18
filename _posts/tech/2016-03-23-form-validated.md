---
layout: post
title: 百度前端学院之表单验证
tags: [学习,私货,前端,环境]
keywords: Javascript,代码,表单验证,前端,学习总结
description: 
---

## 前言

很久没写博客了，最近在和小伙伴一起玩百度前端学院的东西，上周是页面基础就略去不说，这周的js任务非常棒，深入浅出，感觉写完的确能够学到不少姿势。还能看到不少惊艳的作品。联系到实际情况我挑了表单验证这一块的内容。

## 正文

首先看下[任务](http://ife.baidu.com/task/detail?taskId=30)

![imgn](http://7xrp04.com1.z0.glb.clouddn.com/task_2_30_1.jpg)

```

表单获得焦点时，下方显示表单填写规则
表单失去焦点时校验表单内容
校验结果正确时，表单边框显示绿色，并在下方显示验证通过的描述文字
校验结果错误时，表单边框显示红色，并在下方显示验证错误的描述文字
点击提交按钮时，对页面中所有输入进行校验，校验结果显示方式同上。若所有表单校验通过，弹窗显示“提交成功”，否则显示“提交失败”

```

然后看一下完成的效果

![imgn](http://img.haoqiao.me/active80.gif)

一开始为了完成功能写的匆忙，导致代码非常丑陋0-0.后来用心简化了一下，但是看了下别人用ES6写的，感觉还是自愧不如。不过都是为了学习,在这里我贴出我的代码.

```
<!doctype html>
<html>

<head>
    <meta charset="utf-8">
    <title>表单验证</title>
    <style>
    input {
        width: 200px;
        height: 16px;
        padding: 6px 12px;
        font-size: 14px;
        line-height: 1.42857143;
        color: #555;
        background-color: #fff;
        background-image: none;
        border: 1px solid #ccc;
        border-radius: 4px;
        /*        outline: none;*/
        box-shadow: inset 0 1px 1px rgba(0, 0, 0, .075);
    }
    
    .btn {
        display: inline-block;
        padding: 6px 12px;
        margin-bottom: 0;
        font-size: 14px;
        font-weight: 400;
        line-height: 1.42857143;
        text-align: center;
        white-space: nowrap;
        vertical-align: middle;
        -ms-touch-action: manipulation;
        touch-action: manipulation;
        cursor: pointer;
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
        background-image: none;
        border: 1px solid transparent;
        border-radius: 4px;
    }
    
    .btn-primary {
        color: #fff;
        background-color: #337ab7;
        border-color: #2e6da4;
    }
    
    .validate_message {
        font-size: 16px;
        color: #B4B4B4;
    }
    
    .bsuccess {
        border-color: #5cb85c!important;
    }
    
    .bdanger {
        border-color: #d9534f!important;
    }
    
    .success {
        color: #5cb85c!important;
    }
    
    .danger {
        color: #d9534f!important;
    }
    </style>
</head>

<body>

    <body>
        <form action="">
            <div class="form">
                名称&ensp;&ensp;&ensp;&ensp;&ensp;
                <input type="text" id="name">
                <p id="nameMessage" class="validate_message"></p>
                密码&ensp;&ensp;&ensp;&ensp;&ensp;
                <input type="password" id="password">
                <p id="passwordMessage" class="validate_message"></p>
                密码确认&ensp;
                <input type="password" id="verifypass">
                <p id="verifypassMessage" class="validate_message"></p>
                邮箱&ensp;&ensp;&ensp;&ensp;&ensp;
                <input type="text" id="email">
                <p id="emailMessage" class="validate_message"></p>
                手机&ensp;&ensp;&ensp;&ensp;&ensp;
                <input type="text" id="phone">
                <p id="phoneMessage" class="validate_message"></p>
            </div>
            <div id="verify" class="btn btn-primary">提交</div>
        </form>
    </body>
    <script>
    //对每个控件绑定获得焦点事件

    document.querySelector("#name").addEventListener('focus', function() {
        ruleMessage('nameMessage', 'name');
    }, false);
    document.querySelector("#password").addEventListener('focus', function() {
        ruleMessage('passwordMessage', 'password');
    }, false);
    document.querySelector("#verifypass").addEventListener('focus', function() {
        ruleMessage('verifypassMessage', 'verifypass');
    }, false);
    document.querySelector("#email").addEventListener('focus', function() {
        ruleMessage('emailMessage', 'email');
    }, false);
    document.querySelector("#phone").addEventListener('focus', function() {
        ruleMessage('phoneMessage', 'phone');
    }, false);


    //对每个控件绑定失去焦点事件
    document.querySelector("#name").addEventListener('blur', function() {
        validate('checkName');
    }, false);
    document.querySelector("#password").addEventListener('blur', function() {
        validate('checkPassword');
    }, false);
    document.querySelector("#verifypass").addEventListener('blur', function() {
        validate('verifyPassword');
    }, false);
    document.querySelector("#email").addEventListener('blur', function() {
        validate('checkEmail');
    }, false);
    document.querySelector("#phone").addEventListener('blur', function() {
        validate('checkPhone');
    }, false);

    var submit = document.querySelector("#verify");
    submit.addEventListener('click', function() {
        validate('checkAll');
    }, false);

    //传入target，type
    function ruleMessage(target, type) {
        // 对提醒消息样式进行更改
        var message = '';
        switch (type) {
            case 'name':
                message = '必填,长度为4-16个字符';
                break;
            case 'password':
                message = '必填,请输入4位或以上的密码';
                break;
            case 'verifypass':
                message = '再次输入相同密码';
                break;
            case 'email':
                message = '必填,请输入正确格式的邮箱';
                break;
            case 'phone':
                message = '必填,请输入11位手机号码';
                break;
        }
        //对输入框样式进行修改
        document.querySelector("#" + target).className = 'validate_message';

        document.querySelector("#" + target).innerText = message;
    }

    function validate(type) {
        function checkLen(type, name) {
            var validated = document.querySelector("#" + type).value;
            var len = 0;
            var status = 1; //1是成功，0是失败
            var message = name + '可用'; //默认成功消息
            var result = [];
            for (var i = validated.length - 1; i >= 0; i--) {
                //汉字长度为2
                if (/^[\u4e00-\u9fa5]+$/.test(validated[i])) {
                    len = len + 2;
                } else {
                    len++;
                }

            }
            if (len < 4 || len > 16) {
                message = "长度必须为4~16个字符";
                status = 0;
            }
            if (len == 0) {
                message = name + "不能为空";
                status = 0;
            }

            result.push(message);
            result.push(status);
            return result;
        }

        function passRe() {
            //获取两次输入的密码
            var previous = document.querySelector("#password").value;
            var validated = document.querySelector("#verifypass").value;
            var status = 1; //1是成功，0是失败
            var message = '密码输入一致';
            var result = [];

            if (validated != previous) {
                message = "密码输入不一致,请重新输入";
                status = 0;
            }

            if (validated == '') {
                message = "密码不能为空";
                status = 0;
            }

            if (checkLen('password', "密码")[1] != 1) {
                message = "前面的密码不合法";
                status = 0;
            }

            result.push(message);
            result.push(status);
            return result;

        }

        function regex(type, name, regex) {
            var validated = document.querySelector("#" + type).value;
            var message = name + '格式正确';
            var status = 1; //1是成功，0是失败
            var result = [];
            if (!regex.test(validated)) {
                message = name + "格式错误";
                status = 0;
            }

            if (validated == '') {
                message = name + "内容不能为空";
                status = 0;
            }
            result.push(message);
            result.push(status);
            return result;
        }
        var check = {
            checkName: function() {
                return showResult('name', checkLen('name', "姓名"));
            },
            checkPassword: function() {
                return showResult('password', checkLen('password', "密码"));
            },

            verifyPassword: function() {
                return showResult('verifypass', passRe());
            },
            checkEmail: function() {
                return showResult('email', regex('email', '邮箱', /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/));
            },
            checkPhone: function() {
                return showResult('phone', regex('phone', '手机', /^0?1[3|4|5|8][0-9]\d{8}$/));
            },
        }

        function checkAll() {
            if (checkLen('name', "姓名")[1] &&
                checkLen('password', "密码")[1] &&
                passRe()[1] &&
                regex('email', '邮箱', /^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/)[1] &&
                regex('phone', '手机', /^0?1[3|4|5|8][0-9]\d{8}$/)[1] == 1
            ) {
                alert("提交成功!")
            } else {
                alert("提交失败!");
            }
        }

        if (type != 'checkAll') {
            return check[type]();
        } else {
            return checkAll();
        }

    }

    function showResult(type, result) {
        // 对提醒消息样式进行更改
        document.querySelector("#" + type + "Message").innerText = result[0];
        document.querySelector("#" + type + "Message").className = result[1] == 1 ? 'success' : 'danger';

        //对输入框样式进行修改
        document.querySelector("#" + type).className = result[1] == 1 ? 'bsuccess' : 'bdanger';
        return result[1];
    }
    </script>

</html>

```

因为题目限制了只能用原生，我习惯用addEventListener去绑定事件，其实也可以直接对其在Html里面绑定事件，不过为了以后扩展方便，我还是按照个人习惯写了。其实这段代码还是有些问题的，比如不够简练。比如焦点点击和焦点失去事件我是写了两种处理方式。获得焦点的时候用js去控制，但其实这样不是很好，我看到比较好的写法是用css hack来完成。大家可能就想到了`data`属性的用处。这里我简单给出方案：

```

<div class="test" data-info="必填，长度为4-16个字符" data-validation="名字不能为空"></div>

.test:focus::after {
  position: absolute;
  bottom: 0;
  left: 120px;
  content: attr(data-info);
  font-size: 14px;
}
.test::after {
    position: absolute;
    bottom: 0;
    left: 120px;
    content: attr(data-validation);
    font-size: 14px;
}

```

不过css hack是别人的思路我就不在自己的版本里填进去了。自己的思路就是对验证方式分三类：长度，对比，正则。

然后正则是动态传入的。

但其实这个写法还没有具有很好的扩展性。比如验证全部其实可以用一种更好的方式来写，具体我将在未来进行修改。

这里我给出另一位大神用es6写的代码,他的[github地址](https://github.com/codehz/ife-j30)

```
"use strict";

//验证结果类
class ValidationResults {
  constructor(status, msg) {
    this.status = status;
    this.msg = msg;
  }
}
//计算长度（中文x2）不考虑其他特殊字符处理
function getLen(str) {
  return (str.length + encodeURI(str).split(/%..|./).length - 1) / 2;
}
//长度验证模板
function lenValid(value, name) {
  if (value.length == 0)
    return new ValidationResults(false, name + "不能为空");
  if (getLen(value) < 4 || getLen(value) > 16)
    return new ValidationResults(false, name + "长度不合法");
  return new ValidationResults(true, name + "可用");
}
//正则验证模板
function template(text, regex, success, failed) {
  if (regex.test(text))
    return new ValidationResults(true, success);
  return new ValidationResults(false, failed);
}
//验证器散列
const Validators = {
  name: () => lenValid(document.getElementById('name').value, "名字"),
  password: () => lenValid(document.getElementById('password').value, "密码"),
  ["password-re"]() {
    let ret = this.password();
    if (ret.status == false) return new ValidationResults(false, "前面的密码不合法");
    if (document.getElementById('password').value != document.getElementById('password-re').value)
      return new ValidationResults(false, "两次输入的密码不同");
    return new ValidationResults(true, "密码配对");
  },
  email: () => template(document.getElementById('email').value, /^(([^<>()\[\]\.,;:\s@\"]+(\.[^<>()\[\]\.,;:\s@\"]+)*)|(\".+\"))@(([^<>()[\]\.,;:\s@\"]+\.)+[^<>()[\]\.,;:\s@\"]{2,})$/i, "格式正确", "电子邮件格式错误"),
  phone: () => template(document.getElementById('phone').value, /^1[3|4|5|8][0-9]\d{4,8}$/, "手机格式正确", "手机格式错误")
}
//检查函数——检查指定的值，并显示
function check(el) {
  const ret = Validators[el]();
  document.getElementById(el).parentElement.className = ret.status ? "box verified" : "box error";
  document.getElementById(el).nextSibling.setAttribute("data-validation", ret.msg);
}
//检查全部，并用对话框输出结果
function submit() {
  for (let el in Validators) {
    let ret = Validators[el]();
    if (!ret.status) {
      alert('提交失败');
      return;
    }
  }
  alert('提交成功');
}

```

从这份代码里的确能学到不少es6的东东。。。而且第一看到这份代码的时候感到的是非常流畅。

## 结尾
在百度前端学院看到不少优秀前端的作品，各种方案百花齐放，让人学到不少，我将慢慢总结学习。


