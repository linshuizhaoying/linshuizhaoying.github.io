---
layout: post
title: 百度前端学院之听UI组件之日历组件2
tags: [学习,私货,前端,环境]
keywords: Javascript,代码,日历组件,工厂,前端,学习总结
description: 
---

## 前言

昨天写的是最基础的，还有些功能没加进去，比如左右按钮跳转上一年下一年,比如对指定输入框绑定日历组件等等。今天早起将其完成。

## 正文

看下新增的需求:

```

		日期选择面板默认隐藏，会显示一个日期显示框和一个按钮，点击这两个部分，会浮出日历面板。再点击则隐藏。
		点击选择具体日期后，面板隐藏，日期显示框中显示选取的日期
		增加一个接口，用于当用户选择日期后的回调处理


```

![imgn](http://img.haoqiao.me/active83.gif)

效果图如上。需要新加的一个是对输入框的`offsetLeft` `offsetTop`获取并重新设定组件的位置，达到一种点击输入框就能让组件在其正下方显示。

然后因为要用户自己添加回调处理，这样我们之间将其写成一个config传入到组件，然后判断传入的fn是否为`function`即可。

来看下我们新版的js代码：


```
window.onload = function() {
    var config = {
        minYear: 2001,
        maxYear: 2025,
        year: 2017,
        mouth: 6,
        fn: function() {
            console.log("2333");
        },
    };
    datePickerFactory('datePlugin',config);


    function datePickerFactory(target,config) {
        var datePickerFactory = {
            Config: {
                minYear: config.minYear || 2000,
                maxYear: config.maxYear || 2024,
                year:    config.year    ||  5  ,
                mouth:   config.mouth   ||  7
            },
            //绑定年份和月份选择事件
            bindSelect: function() {
                var that = this;
                var yearPicker = document.querySelector(".yearPicker");
                var selectYear = document.querySelector(".selectYear");
                yearPicker.addEventListener('click', function(e) {
                    if (yearPicker.className.indexOf('active') != -1) {
                        yearPicker.className = 'datePicker-select yearPicker';
                    } else {
                        yearPicker.className = 'datePicker-select yearPicker active';
                    }
                }, false);

                selectYear.addEventListener('click', function(e) {
                    document.querySelector(".currYear").innerText = e.target.innerText;
                    that.Config.year = e.target.innerText;
                    yearPicker.className = 'datePicker-select yearPicker active';
                    that.renderContent(that.getWeek(), that.getdate());
                }, false);

                var mouthPicker = document.querySelector(".mouthPicker");
                var selectMouth = document.querySelector(".selectMouth");

                mouthPicker.addEventListener('click', function(e) {
                    if (mouthPicker.className.indexOf('active') != -1) {
                        mouthPicker.className = 'datePicker-select mouthPicker';
                    } else {
                        mouthPicker.className = 'datePicker-select mouthPicker active';
                    }

                }, false);

                selectMouth.addEventListener('click', function(e) {
                    document.querySelector(".currMouth").innerText = e.target.innerText;
                    that.Config.mouth = e.target.innerText;
                    mouthPicker.className = 'datePicker-select mouthPicker active';
                    that.renderContent(that.getWeek(), that.getdate());
                }, false);

            },
            //渲染日历列表
            renderContent: function(weekDay, mouthDay) {
                var dateContent = document.querySelector(".dateContent");
                dateContent.innerHTML = '';
                //创建空的日期占位
                for (var i = 0; i < weekDay; i++) {
                    var li = document.createElement("li");
                    var a = document.createElement("a");
                    a.innerHTML = '';
                    li.appendChild(a);
                    dateContent.appendChild(li);
                }
                //正常日期
                for (var j = 1; j <= mouthDay; j++) {
                    var li = document.createElement("li");
                    var a = document.createElement("a");
                    a.innerHTML = j;
                    li.appendChild(a);
                    dateContent.appendChild(li);
                }

            },
            //根据最大最小年份生成列表
            initYear: function() {
                var that = this;
                var selectYear = document.querySelector(".selectYear");
                for (var i = that.Config.minYear; i <= that.Config.maxYear; i++) {
                    var li = document.createElement("li");
                    var a = document.createElement("a");
                    a.innerHTML = i;
                    li.appendChild(a);
                    selectYear.appendChild(li);
                }
            },
            //确定某个月的第一天是星期几
            getWeek: function() {
                var that = this;
                var thisDay = new Date();
                var thisYear = parseInt(that.Config.year);
                var thisMouth = parseInt(that.Config.mouth);
                //setFullYear() 方法用于设置指定年份和月份。月份从0开始计数
                thisDay.setFullYear(thisYear, thisMouth - 1, 1);
                //getDay() 方法可返回表示星期的某一天的数字。
                var weekDay = thisDay.getDay();
                return weekDay;
            },
            //某个月有多少天
            getdate: function() {
                var that = this;
                var thisYear = parseInt(that.Config.year);
                var mouth = parseInt(that.Config.mouth);
                if (mouth == 1 || mouth == 3 || mouth == 5 || mouth == 7 || mouth == 8 || mouth == 10 || mouth == 12) {
                    return 31;
                } else if (mouth == 4 || mouth == 6 || mouth == 9 || mouth == 11) {
                    return 30;
                } else if ((thisYear % 100 != 0 && thisYear % 4 == 0) || thisYear % 400 == 0) {
                    return 29;
                } else {
                    return 28;
                }
            },
            //提供标准输出接口
            getFormatDate: function() {
                var that = this;
                var dateContent = document.querySelector(".dateContent");
                dateContent.addEventListener('click', function(e) {
                    if (e.currentTarget != e.target) { //这句是为了防止多选日期的时候产生bug
                        var formatDate = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                        var input = document.querySelector("." + target);
                        input.value = formatDate;
                        e.target.className = 'active';
                        that.toggle();
                        //执行回调
                        if(typeof(config.fn)=="function"){
                            config.fn();
                        }
                    }
                }, false);
            },
            //绑定插件到指定输入框
            bindInput: function() {
                var that = this;
                var input = document.querySelector("." + target);

                //调整位置对应输入框底部
                var left = input.offsetLeft;
                var top = input.offsetTop;
                document.querySelector(".datePicker").style.left = left + 'px';
                input.addEventListener('click', function(e) {
                    that.toggle();
                }, false);
            },
            //绑定上一年/下一年事件
            preNext:function(){
                var that = this;
                var pre = document.querySelector(".datePicker-pre");
                var next = document.querySelector(".datePicker-next");
                var currYear =  document.querySelector(".currYear");

                pre.addEventListener('click', function(e) {
                    if(parseInt(currYear.innerText) -1 >= that.Config.minYear){
                        currYear.innerText = parseInt(currYear.innerText) - 1;
                        that.Config.year = currYear.innerText;
                        that.renderContent(that.getWeek(), that.getdate()); //重新渲染
                    }
                }, false);
                next.addEventListener('click', function(e) {
                    if(parseInt(currYear.innerText) +1 <= that.Config.maxYear){
                        currYear.innerText = parseInt(currYear.innerText) + 1;
                        that.Config.year = currYear.innerText;
                        that.renderContent(that.getWeek(), that.getdate());//重新渲染
                    }
                }, false);
            },
            //显示/隐藏日历
            toggle: function() {
                var datePicker = document.querySelector(".datePicker");
                if (datePicker.className.indexOf('show') != -1) {
                    datePicker.className = 'datePicker';
                } else {
                    datePicker.className = 'datePicker show';
                }
            },
            //初始化插件
            init: function() {
                var that = this;
                document.querySelector(".currYear").innerText = that.Config.year;
                that.initYear();
                that.bindInput();
                that.bindSelect();
                that.preNext();
                that.renderContent(that.getWeek(), that.getdate());

                that.getFormatDate();

            }

        }
        return datePickerFactory.init();
    }



}

```

基本上注释都写的很全了。然后可以继续下一环节。





