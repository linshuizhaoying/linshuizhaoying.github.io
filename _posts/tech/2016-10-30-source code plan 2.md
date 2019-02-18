---
layout: post
title: 源码阅读计划之timeago
tags: [学习,整理,前端,实战总结,源码学习,source code]
keywords: web,代码,前端,学习总结,源码阅读,源码学习,源码细节
description: 
---

## 前言

大概半个月之前我看到了这个小型的javascript库，这个库的功能就一个`用来将datetime时间转化成类似于*** 时间前的描述字符串，例如：“3小时前”。`
但是却非常火爆，因为作者围绕这个核心功能开发一些很实用的特性，当初保存这个库的时候就有个想法好好学习一下它的源码。而且作者也是个中国人，中文文档也是齐全的。测试也是写了的。这就很方便学习了。

## 正文

先去[timeago](https://github.com/hustcc/timeago.js)的github地址下载整个源文件，然后来看下测试文件。
根据功能来看 `timeago`分

```

1.设置相对日期
2.格式化时间戳，字符串
3.自动实时渲染
4.本地化和多语言支持

```

来直接看源码

```
// 插件普遍起手写法，兼容exports导出
/*
// 这么写会报错，因为这是一个函数定义：
function() {}()

// 常见的（多了一对括号），调用匿名函数：
(function() {})()

// 但在前面加上一个布尔运算符（只多了一个感叹号），就是表达式了，将执行后面的代码，也就合法实现调用,而 + - || 都有这样的功能
!function() {}()

*/
!function (root, factory) {
  if (typeof module === 'object' && module.exports)
    module.exports = factory(root);
  else
    root.timeago = factory(root);
}(typeof window !== 'undefined' ? window : this,
function () {
  var cnt = 0, // timer计数
    // 英文 年月日时分秒转成数组
    indexMapEn = 'second_minute_hour_day_week_month_year'.split('_'),
    // 中文年 月日时分秒转成数组
    indexMapZh = '秒_分钟_小时_天_周_月_年'.split('_'),
    // 地区化，默认是en和zh_CN两种
    locales = {
      'en': function(number, index) {
        if (index === 0) return ['just now', 'right now'];
        var unit = indexMapEn[parseInt(index / 2)];
        if (number > 1) unit += 's';
        return [number + ' ' + unit + ' ago', 'in ' + number + ' ' + unit];
      },
      'zh_CN': function(number, index) {
        if (index === 0) return ['刚刚', '片刻后'];
        var unit = indexMapZh[parseInt(index / 2)];
        return [number + unit + '前', number + unit + '后'];
      }
    },
    // second, minute, hour, day, week, month, year(365 days)
    SEC_ARRAY = [60, 60, 24, 7, 365/7/12, 12],
    SEC_ARRAY_LEN = 6,
    ATTR_DATETIME = 'datetime';

  // 格式化日期，将日期，字符串，时间戳转为日期实例
  function toDate(input) {
    // 如果传进来的是日期实例则直接返回
    if (input instanceof Date) return input;
    // 如果是时间戳
    if (!isNaN(input)) return new Date(toInt(input));
    if (/^\d+$/.test(input)) return new Date(toInt(input, 10));
    // 如果是字符串
    input = (input || '').trim().replace(/\.\d+/, '') // remove milliseconds
      .replace(/-/, '/').replace(/-/, '/')
      .replace(/T/, ' ').replace(/Z/, ' UTC')
      .replace(/([\+\-]\d\d)\:?(\d\d)/, ' $1$2'); // -04:00 -> -0400
    return new Date(input);
  }
  // 强制转为数字整型
  function toInt(f) {
    return parseInt(f);
  }
  // 根据指定的本地化语言，格式化输出diff为'多少秒之前'
  function formatDiff(diff, locale, defaultLocale) {
    // 如果没指定则默认为en语言包
    locale = locales[locale] ? locale : (locales[defaultLocale] ? defaultLocale : 'en');
    // 判断传进来的是diff是当前时间之前还是之后
    var i = 0,
      agoin = diff < 0 ? 1 : 0; // 1之前 / 0之后
    // 取绝对值
    diff = Math.abs(diff);
    // SEC_ARRAY = [60, 60, 24, 7, 365/7/12, 12] 秒_分钟_小时_天_周_月_年
    // diff 是 时间差,用秒来做单位 diff一次从秒开始除直到无法整除时便计算出最终间隔(小时/月/年)
    for (; diff >= SEC_ARRAY[i] && i < SEC_ARRAY_LEN; i++) {
      diff /= SEC_ARRAY[i];
    }
    diff = toInt(diff);
    i *= 2;
    // 用来定位语言包的合适位置
    if (diff > (i === 0 ? 9 : 1)) i += 1;
    return locales[locale](diff, i)[agoin].replace('%s', diff);
  }
  // 返回指定时间与当前时间的时间差，用秒作为单位
  function diffSec(date, nowDate) {
    nowDate = nowDate ? toDate(nowDate) : new Date();
    return (nowDate - toDate(date)) / 1000;
  }
  // 计算下一个周期
  function nextInterval(diff) {
    var rst = 1, i = 0, d = Math.abs(diff);
    for (; diff >= SEC_ARRAY[i] && i < SEC_ARRAY_LEN; i++) {
      diff /= SEC_ARRAY[i];
      rst *= SEC_ARRAY[i];
    }
    // return leftSec(d, rst);
    d = d % rst;
    d = d ? rst - d : rst;
    return Math.ceil(d);
  }
  // 从给定的节点中获取date属性的值
  function getDateAttr(node) {
    if (node.getAttribute) return node.getAttribute(ATTR_DATETIME);
    if(node.attr) return node.attr(ATTR_DATETIME);
  }
  // 渲染函数
  function Timeago(nowDate, defaultLocale) {
    var timers = {}; 
    if (! defaultLocale) defaultLocale = 'en'; 
    // 渲染，加个cnt好分辨是哪个定时器，到时候清空好清空
    function doRender(node, date, locale, cnt) {
      var diff = diffSec(date, nowDate);
      node.innerHTML = formatDiff(diff, locale, defaultLocale);
      // 每秒渲染一次
      timers['k' + cnt] = setTimeout(function() {
        doRender(node, date, locale, cnt);
      }, nextInterval(diff) * 1000);
    }
    // 调用私有函数format
    this.format = function(date, locale) {
      return formatDiff(diffSec(date, nowDate), locale, defaultLocale);
    };
    // 调用私有函数render
    this.render = function(nodes, locale) {
      if (nodes.length === undefined) nodes = [nodes];
      for (var i = 0; i < nodes.length; i++) {
        doRender(nodes[i], getDateAttr(nodes[i]), locale, ++ cnt); // render item
      }
    };
    // 清楚渲染
    this.cancel = function() {
      for (var key in timers) {
        clearTimeout(timers[key]);
      }
      timers = {};
    };
    // 设置语言
    this.setLocale = function(locale) {
      defaultLocale = locale;
    };
    // 返回
    return this;
  }
  
  // 返回新对象
  function timeagoFactory(nowDate, defaultLocale) {
    return new Timeago(nowDate, defaultLocale);
  }
  // 重新指定语言类型
  timeagoFactory.register = function(locale, localeFunc) {
    locales[locale] = localeFunc;
  };

  return timeagoFactory;
});

```

## 结尾

看完这个源码非常简单而且很适合新手学习它的组织结构。而且还可以深入思考一下，有些插件其实并不需要做的很全，只要你把核心功能写的明白了然，它也是独一无二的。





