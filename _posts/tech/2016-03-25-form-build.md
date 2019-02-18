---
layout: post
title: 百度前端学院之表单自动生成工厂
tags: [学习,私货,前端,环境]
keywords: Javascript,代码,表单自动生成,工厂,前端,学习总结
description: 
---

## 前言

继续前天后续的内容。之前写的版本有点问题，都将在这个版本中的到优化。

## 正文

今天的任务描述如下：

```

实现以JavaScript对象的方式定义表单及验证规则
表单配置参考示例如下：（不需要一致，仅为参考）

    {
        label: '名称',                    // 表单标签
        type: 'input',                   // 表单类型
        validator: function () {...},    // 表单验证规
        rules: '必填，长度为4-16个字符',    // 填写规则提示
        success: '格式正确',              // 验证通过提示
        fail: '名称不能为空'               // 验证失败提示
    }
    
基于该配置项，实现一套逻辑，可以自动生成表单的展现、交互、验证

```

首先需要考虑如何生成，以及后续分类等等问题，但其实一开始写的时候并没想那么多，一看到配置项就自然而言想到用单例模式来完成。

后续的优化都是在之前的基础上进行的。

分两种情况，一种是双重验证，比如重复密码验证，一种是单层验证，比如格式验证。

然后把之前的css样式直接复制粘贴，然后就可以着手写js代码了。

看下最后实现效果

![imgn](http://img.haoqiao.me/active81.gif)

最终代码如下:

```

    /*
    	表单自动生成工厂v1.0.0
			使用了单例模式。
			分两种情况，一种是双重验证，比如重复密码验证
								一种是单层验证，比如格式验证
			暂时只写了input生成，代码思路应该还是清楚的,后续添加应该还是方便的。。。

     */

    var data = [{
        label: '名字',
        name: 'name',
        type: 'input',
        validator: function(name) {
            return checkLen(name) >= 4 && checkLen(name) <= 16;
        },
        rules: '必填，长度为4-16个字符',
        success: '格式正确',
        fail: '格式错误'
    }, {
        label: '密码',
        name: 'password',
        type: 'password',
        validator: function(password) {
            return checkLen(password) >= 4 && checkLen(password) <= 16;
        },
        rules: '必填，长度为4-16个字符',
        success: '格式正确',
        fail: '格式错误'
    },{
        label: '重复密码',
        name: 'verifypassword',
        type: 'password',
        validator: function(password) {
            return verifypassword("password",password);
        },
        rules: '再次输入相同密码',
        success: '输入相同',
        fail: '两次密码输入不一致,请重新输入'
    },{
        label: '邮箱',
        name: 'email',
        type: 'input',
        validator: function(email) {
            return checkRegex(email,/^(([^<>()\[\]\\.,;:\s@"]+(\.[^<>()\[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/);
        },
        rules: '必填,请输入合法邮箱',
        success: '格式正确',
        fail: '格式错误'
    },{
        label: '手机',
        name: 'phone',
        type: 'input',
        validator: function(phone) {
            return checkRegex(phone,/^0?1[3|4|5|8][0-9]\d{8}$/);
        },
        rules: '必填,请输入合法手机号码',
        success: '格式正确',
        fail: '格式错误'
    }

    ];

    window.onload = function() {
        for (var i = 0 ;i < data.length ;i++) {
            formsFactory(data[i]);
        }
        var el = document.createElement("div");
        el.className = "submit btn btn-primary";
        el.innerText = "提交";
        el.addEventListener('click', function() {
            checkAll();
        }, false);
        document.body.appendChild(el);
    }

    function checkLen(string) {
        var len = 0;
        for (var i = string.length - 1; i >= 0; i--) {
            //汉字长度为2
            if (/^[\u4e00-\u9fa5]+$/.test(string[i])) {
                len = len + 2;
            } else {
                len++;
            }

        }
        return len;

    }
    function verifypassword(lastpassId,string){
    	var lastpass = document.querySelector('#'+lastpassId).value;
    	return string == lastpass;
    }

    function checkRegex(string, regex) {
        var validated = string;
        return regex.test(validated);
    }

    function checkAll() {
        var List = document.querySelectorAll("input");
        var flag = 0;
        console.log(List.length)
        for (var i = List.length - 1; i >= 0; i--) {
            if (List[i].className == 'bsuccess') {
                flag++;
            }
        }

        if (flag == List.length) {
            alert("提交成功!")
        } else {
            alert("提交失败!");
        }

    }


    function formsFactory(data) {
        var config = data;

        var valiadate = {
            arrangement: {
                label: config.label, // 表单标签
                name: config.name, // 表单名称
                type: config.type, // 表单类型
                validator: config.validator, // 表单验证规
                rules: config.rules, // 填写规则提示
                success: config.success, // 验证通过提示
                fail: config.fail // 验证失败提示
            },

            generateInput: function(type) {
                var that = this;
                var container = document.createElement("div")
                container.className = 'formscontent';
                var span = document.createElement("span");
                span.innerText = that.arrangement.label;

                var p = document.createElement("p");
                p.className = 'validate_message';

                var input = document.createElement("input");
                input.name = that.arrangement.name;
                input.type = that.arrangement.type;
                input.id = that.arrangement.name;
                input.addEventListener('focus', function() {
                  p.innerText = that.arrangement.rules;
                }, false);

                input.addEventListener('blur', function() {
                	  var validate = '';
                		if(type == 'single'){
                			validate = that.arrangement.validator(this.value);
                		}else if(type == 'verify'){
                			validate = that.arrangement.validator(this.value) && this.value.length!=0;
                		}
                    if (validate) {
                        input.className = 'bsuccess';
                        p.innerText = that.arrangement.label + that.arrangement.success;
                        p.className = 'success';
                    } else if (this.value.length == 0) {
                        input.className = 'bfail';
                        p.innerText = that.arrangement.label + '不能为空';
                        p.className = 'fail';
                    } else {
                        input.className = 'bfail';
                        p.innerText = that.arrangement.fail;
                        p.className = 'fail';
                    }
                }, false);

                container.appendChild(span);
                container.appendChild(input);
                container.appendChild(p);
                document.body.appendChild(container);
            },
            init: function() {
                var that = this;
                //判断类型
                switch (that.arrangement.name) {
                    case 'name':
                        that.generateInput('single');
                        break;
                    case 'password':
                        that.generateInput('single');
                        break;
                    case 'verifypassword':
                        that.generateInput('verify');
                        break;
                    case 'email':
                        that.generateInput('single');
                        break;
                    case 'phone':
                        that.generateInput('single');
                        break;
                }
            }

        }
        return valiadate.init();
    }
    
```

都挺简单就不仔细分析了。前天的验证全部方法重写了

```

    function checkAll() {
        var List = document.querySelectorAll("input");
        var flag = 0;
        console.log(List.length)
        for (var i = List.length - 1; i >= 0; i--) {
            if (List[i].className == 'bsuccess') {
                flag++;
            }
        }

        if (flag == List.length) {
            alert("提交成功!")
        } else {
            alert("提交失败!");
        }

    }
    
```

这个思路是基于每个Input输入都正确才会添加bsuccess样式，只要判断拥有这个样式的Input数量是不是等于页面中所有的Input数量就可以得出是否全部验证。并不需要把每个验证方法都去执行一遍。

当然，好奇的我还是去看了一下es6实现的版本，贴一下代码:

```

"use strict";
//计算长度
function getlen(text) {
  return (text.length + encodeURI(text).split(/%..|./).length - 1) / 2;
}
//检查函数，并设置标记
function check(value, box, tip, template) {
  if (template.validator(value)) {
    box.className = "box verified";
    tip.setAttribute("data-validation", template.success);
    return true;
  } else {
    box.className = "box error";
    tip.setAttribute("data-validation", template.fail);
    return false;
  }
}
//生成
function genForm() {
  return data.map(template => {
    let box = document.createElement("div");//单个项目的容器
    box.className = "box";

    let label = document.createElement("label");//标签
    label.appendChild(document.createTextNode(template.label));
    box.appendChild(label);

    let tip = document.createElement("div");//提示文本的父元素
    tip.setAttribute("data-info", template.rules);
    tip.className = "tip";

    let valueGetter = null;//保存用于无须遍历DOM树快速查找含有值的元素

    if (template.type == 'radio') {
      let container = document.createElement("div");//单选的父元素
      container.className = 'radio-option';

      let elCache = [];

      for (let option in template.radioExt) {
        let labelContainer = document.createElement("label");//每个选项的标签

        let optionEl = document.createElement("input");//选项
        optionEl.setAttribute("type", "radio");
        optionEl.setAttribute("name", template.name);
        optionEl.setAttribute("value", option);
        optionEl.addEventListener("change", () => check(optionEl.value, box, tip, template));//设置验证器

        elCache.push(optionEl);

        labelContainer.appendChild(optionEl);
        labelContainer.appendChild(document.createTextNode(template.radioExt[option]));

        container.appendChild(labelContainer);
      }

      //catch elCache to lambda function
      valueGetter = () => {
        for (let it of elCache) if (it.checked) return it.value;
        return null;
      };

      box.appendChild(container);
    } else {
      let el = document.createElement("input");//文本框
      el.setAttribute("type", template.type);
      el.setAttribute("name", template.name);

      if (template.attr) {
        for (let key in template.attr) {
          el.setAttribute(key, template.attr[key]);
        }
      }

      el.addEventListener('blur', () => check(el.value, box, tip, template));//设置验证器

      valueGetter = () => el.value;

      box.appendChild(el);
    }

    box.appendChild(tip);

    let validator = () => check(valueGetter(), box, tip, template);

    return {validator, box};
  });
}

let formels = genForm();

function checkAll() {
  for (let formel of formels)
    if (!formel.validator())
      return alert('提交失败');
  return alert('提交成功');
}

window.onload = () => {
  let form = document.getElementById('target');

  for (let formel of formels)
    form.appendChild(formel.box);//逐个加入

  let el = document.createElement("input");//加入按钮
  el.setAttribute("type", "button");
  el.value = "Submit";
  el.addEventListener('click', checkAll);
  console.log(el);
  form.appendChild(el);
}

```

这份顺便实现了radio的版本。真的是学无止境啊=-=

## 结尾

接下来有时间的话是去捣鼓其它任务。到时候会一一放出来。



