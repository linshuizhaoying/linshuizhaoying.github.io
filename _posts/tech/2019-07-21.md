---
layout: post
title: 前端验证码方案小结
category: 技术
tags: [总结,开发,前端,开发,验证码]
keywords: 总结,开发,前端,开发,验证码]
description: 
---

## 前言
 
 七月份发生了很多事情，然后精力基本上也没多少放在研究新技术上，除了日常工作，还请了一个星期假回老家订婚，领证。所以这篇文章会比较简单。

## 正文
 
 作为前端我们常常会遇到一些场景需要用到验证码，在过去我们会比较依赖后端的服务。但是有了node我们基本上都能自己来。但是涉及到验证码就肯定会涉及到安全的问题。
 
 因此我们不可能仅仅只是纯粹的展示验证码和校验。
 

## 方案1

通过 svg-captcha 生成验证码，然后将验证码文本通过唯一的 csrf token 存储到redis服务。

前端得到svg的验证码显示，然后将输入的验证码通过接口传递给node端判断是否正确。

node端通过csrf读取redis保存的验证码。

## 方案2

通过 svg-captcha 生成验证码，然后将验证码文本以对称加密方式一起传给前端。前端直接校验。

## 代码

获取验证码 参考代码如下：

```

// 返回用户验证码图像
export const verifyCode = async (ctx, next) => {
  const fontSize = ctx.query.size || 40;
  const width = ctx.query.w || 150;
  const height = ctx.query.h || 50;
  svgCaptcha.options.fontSize = fontSize; // captcha的宽度
  svgCaptcha.options.width = width; // 验证码的高度
  svgCaptcha.options.height = height; // captcha文本大小
  // svgCaptcha.options.charPreset = charPreset; // 随机字符预设
  const captcha = svgCaptcha.create({
    size: 4,
    // 验证码去掉模糊人眼容易认错的字符
    ignoreChars: '0o1iLlI',
    noise: 2,
    color: false
  });
  const csrf = ctx.request.body.data.csrf || 'verifyCodeSecret';
  const key = cipher(captcha.text.toLowerCase(), 'RC4', csrf);
  ctx.body = {
    status: '0',
    result: 'success',
    data: {
      svg: captcha.data,
      key: key
    }
  };
  return;
};

```


加密代码

```

import * as CryptoJS from 'crypto-js';
export const cipher = (str, method, keyStr) => {
  const key = CryptoJS.enc.Utf8.parse(keyStr);
  const encryptedData = CryptoJS[method].encrypt(str, key);
  const encryptedStr = encryptedData.ciphertext.toString();

  return encryptedStr;
};

export const decipher = (data, method, keyStr) => {
  if (!(data instanceof Object)) return data;
  const key = CryptoJS.enc.Utf8.parse(keyStr);
  const result = Object.assign({}, data);
  Object.keys(data).forEach(element => {
    const encryptedHexStr = CryptoJS.enc.Hex.parse(data[element]);
    const encryptedBase64Str = CryptoJS.enc.Base64.stringify(encryptedHexStr);
    const decryptedData = CryptoJS[method].decrypt(encryptedBase64Str, key);
    const decryptedStr = decryptedData.toString(CryptoJS.enc.Utf8);

    result[element] = decryptedStr;
  });
  return result;
};

```



### 前端react引用

>因为方法二返回的svg是纯代码的，无法直接插入页码，需要通过特殊方式显示
><span dangerouslySetInnerHTML={{ __html: appStore.verifyCodeSvg }}></span>


