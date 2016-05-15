---
layout: post
title: 百度前端学院之听UI组件之日历组件3
tags: [学习,私货,前端,环境]
keywords: Javascript,代码,日历组件,工厂,前端,学习总结
description: 
---

## 前言

这次需求增加一个多选，感觉难度几何上升，自己勉强写了符合需求的，但是还存在不少问题，限于水平我就没进行下去了。
## 正文

需求如下:

```
		增加一个参数及相应接口方法，来决定这个日历组件是选择具体某天日期，还是选择一个时间段
		当设置为选择时间段时，需要在日历面板上点击两个日期来完成一次选择，两个日期中，较早的为起始时间，较晚的为结束时间，选择的时间段用特殊样式标示
		增加参数及响应接口方法，允许设置时间段选择的最小或最大跨度，并提供当不满足跨度设置时的默认处理及回调函数接口
		在弹出的日期段选择面板中增加确认和取消按钮

```


![imgn](http://haoqiao.qiniudn.com/active84.gif)


看上去效果好像都写好了，但是这两天写的感觉不太对，尤其是后续优化方面的桎梏。以及无法跨月份选择。这都是之前没考虑清楚，现在写完感觉也蛮混乱的。不过还是把当月的选择给写完了。

```
js:
window.onload = function() {
    var config = {
        minYear: 2001,
        maxYear: 2025,
        year: 2017,
        mouth: 6,
        type: 'multi',
        minRange: 2,
        maxRange: 10,
        fn: function() {
            console.log("2333");
        },
        rangefn: function() {
            console.log("选中范围成功!");
        }
    };
    datePickerFactory('datePlugin', config);


    function datePickerFactory(target, config) {
        var datePickerFactory = {
            Config: {
                minYear: config.minYear || 2000,
                maxYear: config.maxYear || 2024,
                year: config.year || 5,
                mouth: config.mouth || 7,
                minRange:config.minRange,
                maxRange:config.maxRange,
                type: config.type || 'radio'
            },
            //参数保存
            storage: {
                startDate: '', //选择的第一个日期
                finishDate: '', //选择的第二个日期

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
                var that = this;
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
                //标记选择日期
            
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
            //两个日期之间间隔多少天
            getDatePeriod: function(start, finish) {
                console.log("tesdt" + (start * 1 - finish * 1) / 60 / 60 / 1000 / 24)
                return Math.abs(start * 1 - finish * 1) / 60 / 60 / 1000 / 24;
            },
            clearInterval: function() {
                var array = document.querySelectorAll(".dateContent li");
                for (var i = 0; i < array.length; i++) {

                    array[i].className = ' ';
                }
            },
            //渲染两个日期之间的颜色
            renderInterval: function() {
                var that = this;
                that.renderContent(that.getWeek(), that.getdate());
                var start = that.storage.startDate.getDate();
                var finish = that.storage.finishDate.getDate();
                var array = document.querySelectorAll(".dateContent li");
                var currMouth = parseInt(document.querySelector(".currMouth").innerText);
                // console.log("that.storage.startDate"+that.storage.startDate.getMonth());
                // console.log('that.storage.finishDate '+that.storage.finishDate );
                // console.log('currMouth'+currMouth)
                // if(parseInt(currMouth))
                for (var i = 0; i < array.length; i++) {
                    if (parseInt(array[i].firstElementChild.innerText) < parseInt(finish) && parseInt(array[i].firstElementChild.innerText) > parseInt(start)) {
                        array[i].className = 'interval';
                    }
                    if(parseInt(array[i].firstElementChild.innerText) == parseInt(finish) || parseInt(array[i].firstElementChild.innerText) == parseInt(start)){
                        array[i].firstElementChild.className = 'active';
                    }
                
                }

                

            },
            //提供单选标准输出接口
            getRadioFormatDate: function() {
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
                        if (typeof(config.fn) == "function") {
                            config.fn();
                        }
                    }
                }, false);
            },
            //提供多选标准输出接口
            getMultiFormatDate: function() {
                var that = this;
                var dateContent = document.querySelector(".dateContent");
                var multi = false;
                var start = '',
                    startNum = '';
                var finish = '',
                    finishNum = '';
                var lastCheck = ''; //标记取消的那个日期
                var hadselect = 0; //已经选中的个数
                var exchange = false;
                //对日历进行事件绑定
                dateContent.addEventListener('click', function(e) {
                    if (e.currentTarget != e.target && e.target.innerText) { //这句是为了防止多选日期的时候产生bug
                        /**
                         * [只允许选中两个]
                         * 每次点击前判断是否选中,选中那么就取消选中
                         * 并且判断已经点击的个数是否超了,没超再继续
                         */

                        if (multi == false && hadselect < 2 && !exchange) {
                            if (e.target.className == 'active') {
                                e.target.className = '';
                                lastCheck = e.target.innerText;
                                multi = false;
                                hadselect--;
                            } else {
                                var input = document.querySelector("." + target);
                                e.target.className = 'active';
                                multi = true;
                                start = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                                startNum = e.target.innerText;
                                // console.log("start赋值")
                                hadselect++;
                            }

                        } else {
                            if (e.target.className != 'active' && hadselect < 2) {
                                var input = document.querySelector("." + target);
                                e.target.className = 'active';
                                // finish = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                                // finishNum = e.target.innerText; 
                                // exchange = true;

                                if (!exchange) {
                                    finish = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                                    finishNum = e.target.innerText;
                                    exchange = true;
                                } else {
                                    //如果第二次重选在第一次选择区间中间
                                    console.log('lastCheck' + lastCheck);
                                    if (parseInt(e.target.innerText) < parseInt(finishNum) && parseInt(e.target.innerText) > parseInt(startNum)) {
                                        // console.log('中间交换')
                                        finish = start;
                                        finishNum = startNum;
                                        start = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                                        startNum = e.target.innerText;
                                    } else if (parseInt(e.target.innerText) < parseInt(startNum)) {
                                        // console.log('前面交换')
                                        if (lastCheck == finishNum) {
                                            finish = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                                            finishNum = e.target.innerText;
                                        } else {
                                            start = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                                            startNum = e.target.innerText;
                                        }


                                    } else if (parseInt(e.target.innerText) > parseInt(finishNum)) {
                                        // console.log('后面交换')
                                        // start = finish;
                                        // startNum = finishNum;
                                        if (lastCheck == finishNum) {
                                            finish = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                                            finishNum = e.target.innerText;
                                        } else {
                                            start = that.Config.year + '-' + that.Config.mouth + '-' + e.target.innerText;
                                            startNum = e.target.innerText;
                                        }
                                    }

                                }

                                hadselect++;
                                console.log(that.getDatePeriod(new Date(Date.parse(start)), new Date(Date.parse(finish))));
                                //执行回调
                                console.log('开始日期' + start);
                                console.log('结束日期' + finish);
                                if (typeof(config.rangefn) == "function") {
                                    config.rangefn();
                                }
                            } else if (e.target.className == 'active') {
                                e.target.className = '';
                                lastCheck = e.target.innerText;
                                multi = false;
                                hadselect--;
                                that.clearInterval();
                                //如果全部选择并取消,那么恢复初始
                                if (hadselect == 0) {
                                    start = '';
                                    startNum = '';
                                    finish = '';
                                    finishNum = '';
                                    exchange = false;
                                }
                            }
                        }

                        if (hadselect == 2) {
                            //that.renderInterval(new Date(Date.parse(start)), new Date(Date.parse(finish)))
                            var interval = (new Date(Date.parse(start)) * 1 - new Date(Date.parse(finish)) * 1) / 60 / 60 / 1000 / 24;
                            

                            if(Math.abs(interval) > parseInt(that.Config.maxRange) || Math.abs(interval) < parseInt(that.Config.minRange)){
                                alert('不符合时间范围，请重新选择');
                                multi = false;
                                start = '',
                                    startNum = '';
                                finish = '',
                                    finishNum = '';
                                lastCheck = ''; //标记取消的那个日期
                                hadselect = 0; //已经选中的个数
                                exchange = false;
                                that.renderContent(that.getWeek(), that.getdate());

                                return;

                            }
                            if (interval < 0) {
                                that.storage.startDate = new Date(Date.parse(start));
                                that.storage.finishDate = new Date(Date.parse(finish));
                                that.renderInterval();
                            } else {
                                that.storage.startDate = new Date(Date.parse(finish));
                                that.storage.finishDate = new Date(Date.parse(start));
                                that.renderInterval();
                            }


                        }

                    }

                }, false);
                document.querySelector(".multiOk").addEventListener('click', function(e) {
                    //如果已经选择范围了
                    if (hadselect == 2) {
                        var input = document.querySelector("." + target);
                        //不确定用户是先点击前面的日期还是后面的日期,因此需要做个判断
                        if (Date.parse(start) - Date.parse(finish) < 0) {
                            input.value = start + ' to ' + finish;
                        } else {
                            input.value = finish + ' to ' + start;
                        }


                        that.toggle();
                    } else {
                        alert("您选中的范围不正确,请重新选择!");
                    }
                }, false);
                document.querySelector(".multiCancel").addEventListener('click', function(e) {
                    multi = false;
                    start = '',
                        startNum = '';
                    finish = '',
                        finishNum = '';
                    lastCheck = ''; //标记取消的那个日期
                    hadselect = 0; //已经选中的个数
                    exchange = false;
                    var input = document.querySelector("." + target);
                    input.value = '';
                    that.toggle();
                    that.renderContent(that.getWeek(), that.getdate());
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
            preNext: function() {
                var that = this;
                var pre = document.querySelector(".datePicker-pre");
                var next = document.querySelector(".datePicker-next");
                var currYear = document.querySelector(".currYear");

                pre.addEventListener('click', function(e) {
                    if (parseInt(currYear.innerText) - 1 >= that.Config.minYear) {
                        currYear.innerText = parseInt(currYear.innerText) - 1;
                        that.Config.year = currYear.innerText;
                        that.renderContent(that.getWeek(), that.getdate()); //重新渲染
                    }
                }, false);
                next.addEventListener('click', function(e) {
                    if (parseInt(currYear.innerText) + 1 <= that.Config.maxYear) {
                        currYear.innerText = parseInt(currYear.innerText) + 1;
                        that.Config.year = currYear.innerText;
                        that.renderContent(that.getWeek(), that.getdate()); //重新渲染
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
            //绑定多个日期选择按钮事件
            initMultiple: function() {
                document.querySelector(".multiContent").className = 'multiContent show';
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
                if (that.Config.type == 'radio') {
                    that.getRadioFormatDate();
                } else {
                    that.initMultiple();
                    that.getMultiFormatDate();
                }


            }

        }
        return datePickerFactory.init();
    }



}

```

可以看到为了跨月份选择我写了很多判断。但是根据我看别人的思路发现我这种写法是有问题的。
一个是可扩展性，如果后续再对多选进行扩展，会发现很难进行。
一个是性能。感觉很多地方作了不必要的判断选择。
但是这块很多实现都不是很好,唯一看到比较好的是一开始就在渲染也就是renderContent这块进行判断渲染，先拿到一个日期，然后对其进行分析渲染。




## 结尾

其实这个还不算真正的组件，原因就是它还是html,css,js分离的，因此我们接下来我们需要将其整合成一个js插件。

