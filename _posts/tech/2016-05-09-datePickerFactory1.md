---
layout: post
title: 百度前端学院之听UI组件之日历组件
tags: [学习,私货,前端,环境]
keywords: Javascript,代码,日历组件,工厂,前端,学习总结
description: 
---

## 前言
前段时间全身心投入到复习，因此没有时间来写博客，这几天结束了考试，将前段时间的内容一点点赶上。

## 正文
 
![imgn](http://haoqiao.qiniudn.com/active82.gif)

先去抄了一个界面，然后开始写代码。

题目要求如下:

```
		组件默认一直呈显示状态
		通过某种方式选择年、月，选择了年月后，日期列表做相应切换
		通过单击某个具体的日期进行日期选择
		组件初始化时，可配置可选日期的上下限。可选日期和不可选日期需要有样式上的区别
		提供设定日期的接口，指定具体日期，日历面板相应日期选中
		提供获取日期的接口，获取日历面板中当前选中的日期，返回一个日期对象（或其他形式，自定）
任务注意事项
		示例图仅为参考，样式及交互方式不需要完全实现一致
		可以使用JQuery等类库，不可直接使用现成的日历组件
		请注意代码风格的整齐、优雅
		代码中含有必要的注释

```

个人爱好把组件完全独立出来，因此使用原生js来写。

```
html:
    <div class="datePicker">
        <div class="datePicker" style="left: 100px; display: block;">
            <div class="datePicker-header">
                <b class="datePicker-pre"></b>
                <b class="datePicker-next"></b>
                <div class="datePicker-select yearPicker">
                    <div class="date-hd">
                        <b class="date-icon"></b>
                        <span class="currYear">2016</span>
                    </div>
                    <div class="date-bd">
                        <ul class="selectYear">
                        </ul>
                    </div>
                </div>
                <div class="datePicker-select mouthPicker">
                    <div class="date-hd">
                        <b class="date-icon"></b>
                        <span class="currMouth">5</span>
                    </div>
                    <div class="date-bd">
                        <ul class="selectMouth">
                            <li><a>1</a></li>
                            <li><a>2</a></li>
                            <li><a>3</a></li>
                            <li><a>4</a></li>
                            <li><a>5</a></li>
                            <li><a>6</a></li>
                            <li><a>7</a></li>
                            <li><a>8</a></li>
                            <li><a>9</a></li>
                            <li><a>10</a></li>
                            <li><a>11</a></li>
                            <li><a>12</a></li>
                        </ul>
                    </div>
                </div>
            </div>
            <div class="datePicker-title">
                <ul>
                    <li>日</li>
                    <li>一</li>
                    <li>二</li>
                    <li>三</li>
                    <li>四</li>
                    <li>五</li>
                    <li>六</li>
                </ul>
            </div>
            <div class="datePicker-body">
                <ul class="clearfix dateContent"></ul>
            </div>
        </div>
    </div>
```

```
css:
/*
    reset
*/

body,div,b,ul,li,a{
    padding: 0;
    margin: 0;
}

li,ul,ol{
    list-style: none;
}
a{
    text-decoration: none;
}
body{
    font-family: 'Microsoft yahei';
    font-size: 14px;
}
.clearfix:after,
.clearfix:before{
    content: '';
    display: table;
}
.clearfix:after{
    clear: both;
}
.clearfix{
    *zoom:1;
    overflow: hidden;
}


.datePicker{
    position: absolute;
    z-index: 100;
    width: 250px; 
}

.datePicker-header{
    position: relative;
    height: 35px;
    line-height: 35px;
    background-color: #d9edf7;
    text-align: center;
}
.datePicker-select{
    position: relative;
    display: inline-block;
}
.datePicker-select .date-hd{
    position: relative;
    width: 80px;
    height: 25px;
    line-height: 25px;
    background: #fff;
    font-size: 12px;
}
.datePicker-select .date-hd span{
    padding-right: 10px;
}
.datePicker-select .date-icon{
    position: absolute;
    border: 6px solid transparent;
    right: 10px;
    top: 9px;
    border-top-color: #ccc;
    cursor: pointer;
}
.datePicker-select .date-icon:after{
    content: '';
    position: absolute;
    top: -8px;
    right: -7px;
    border: 7px solid transparent;
    border-top-color: #fff;
    
}

.datePicker-select .date-bd{
    position: absolute;
    top: 25px;
    width: 80px;
    display: none;
    background: #fff;
    font-size: 12px;
    border: 1px solid #d9edf7;
    border-top: none;
    margin-left: -1px;
}
.datePicker-select .date-bd ul{
    max-height: 200px;
    overflow: auto;
}

.datePicker-select .date-bd a{
    padding-right: 6px;
    color: #333;
    cursor: pointer;
}
.datePicker-select .date-bd a:hover{
    background: #d9edf7;
    color: #fff;
    display: block;
}
.datePicker-select .date-bd li{
    height: 25px;
    line-height: 25px;
}
.datePicker-select.active .date-bd{
    display: block;
}
.datePicker-pre,
.datePicker-next{
    position: absolute;
    top: 9px;
    border: 7px solid transparent;
    transition: .3s all ease-out;
    cursor: pointer;
}

.datePicker-pre{
    border-right-color: rgba(255,255,255,.6);
    left: 5px;
}
.datePicker-pre:hover{
    border-right-color: rgba(255,255,255,1);
}
.datePicker-next{
    border-left-color: rgba(255,255,255,.6);
    right: 5px;
}
.datePicker-next:hover{
    border-left-color: rgba(255,255,255,1);
}

.datePicker-title{
    border-left: 1px solid #ccc;
    border-right: 1px solid #ccc;
}
.datePicker-title ul{
    height: 30px;
    line-height: 30px;
    overflow: hidden;
}
.datePicker-body li,
.datePicker-title li{
    width: 14%;
    float: left;
    text-align: center;
    
}
.datePicker-body ul{
    border:1px solid #ccc;
    padding: 10px 0;
}
.datePicker-body li{
    height: 35px;
    line-height: 35px;
    cursor: pointer;
}
.datePicker-body li a{
    display: block;
    color: #333;
}
.datePicker-body li.active a,
.datePicker-body li:hover a{
    background-color: #d9edf7;
    color: #fff;
}

```

接下来直接看js代码:

```
window.onload = function() {

    datePickerFactory('datePicker');


    function datePickerFactory(id,year,mouth) {
        var datePickerFactory = {
            Config: {
                minYear:2000,
                maxYear:2024,
                year: 2016, 
                mouth: 5
            },
            //绑定年份和月份选择事件
            bindSelect:function(){
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
                    document.querySelector(".currYear").innerText =  e.target.innerText;
                    that.Config.year = e.target.innerText;
                    yearPicker.className = 'datePicker-select yearPicker active';
                    that.renderContent(that.getWeek(),that.getdate());
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
                    document.querySelector(".currMouth").innerText =  e.target.innerText;
                    that.Config.mouth = e.target.innerText;
                    mouthPicker.className = 'datePicker-select mouthPicker active';
                    that.renderContent(that.getWeek(),that.getdate());
                }, false);

            },
            renderContent: function(weekDay,mouthDay) {
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
            initYear:function(){
                var that = this;
                var selectYear = document.querySelector(".selectYear");
                for (var i = that.Config.minYear ; i <= that.Config.maxYear; i++) {
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
                if(mouth==1||mouth==3||mouth==5||mouth==7||mouth==8||mouth==10||mouth==12){
                    return 31;
                } else if (mouth == 4 || mouth == 6 || mouth == 9 || mouth == 11) {
                    return 30;
                } else if ((thisYear % 100 != 0 && thisYear % 4 == 0) || thisYear % 400 == 0) {
                    return 29;
                } else {
                    return 28;
                }
            },
            getFormatDate:function(){
                 var that = this;
                 var dateContent = document.querySelector(".dateContent");
                 dateContent.addEventListener('click', function(e) {
                   var formatDate = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                   console.log(formatDate);
                }, false);
            },
            init: function() {
                var that = this;
                that.initYear();
                that.renderContent(that.getWeek(),that.getdate());
                that.bindSelect();
                that.getFormatDate();

            }

        }
        return datePickerFactory.init();
    }



}

```

这次写的很简洁，基本就是一个需求一个方法，然后用工厂模式组合一下就出来了。关于如何把样式和js代码完全组合在一起构成一个即插即用的组件也有一点点想法，大概思路就是Shadow DOM.这也是这几天阅读的收获。

## 结尾
先写一篇做个开头。接下来会慢慢恢复更新。


