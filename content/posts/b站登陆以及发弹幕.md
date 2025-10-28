---
title: b站登陆以及发弹幕
date: 2016-03-23
tags:
  - bilibili
  - http
---

本文只说明不使用验证码的登录，使用验证码的也是大同小异，打开B站F12然后登录，看一下HTTP报文就知道了。

## 登录

登录密码需要加密（不过经测试，明文也是能登录的，如果用明文登录请跳至第三步），用的是RSA加密算法。

1. 首先要获取公钥：

  ```
  GET:https://passport.bilibili.com/login?act=getkey&_= + 当前时间
  ```

  返回的json类似这个样子：

  ```
  {"hash":"4d656f543748e605","key":"-----BEGIN PUBLIC KEY-----\nMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCdScM09sZJqFPX7bvmB2y6i08J\nbHsa0v4THafPbJN9NoaZ9Djz1LmeLkVlmWx1DwgHVW+K7LVWT5FV3johacVRuV98\n37+RNntEK6SE82MPcl7fA++dmW2cLlAjsIIkrX+aIvvSGCuUfcWpWFy3YVDqhuHr\nNDjdNcaefJIQHMW+sQIDAQAB\n-----END PUBLIC KEY-----\n"}
  ```

2. 把hash值加到你的密码前面，然后用公钥加密。注意公钥是Base64编码过的，去掉头尾和换行之后要先解码。同样的，公钥加密之后的密文也要经过Base64编码。
3. 登录请求：

  ```
  POST:https://passport.bilibili.com/ajax/miniLogin/login
  ```

  随之还要提交一个表单，有四个字段。
  "userid=用户名&pwd=密码&captcha=验证码&keep=是否保持登录状态"
  验证码填空就行，keep填1为是，0为否。
  登录成功后，返回的json:

  ```
  {"status":true,"ts":1458750856,"data":{"code":0,"crossDomain":"https://passport.bilibili.com/crossDomain?DedeUserID=241087&DedeUserID__ckMd5=7e772bf49d639da6&Expires=1800&SESSDATA=ed6f17cd%2C1458837256%2C7cd89c4f&gourl=https%3A%2F%2Fpassport.bilibili.com%2Fajax%2FminiLogin%2Fredirect"}}
  ```

crossDomain包含了你的cookie信息。这个信息是必须的，在发弹幕时需要用到，否则会提示你需要登录。
把DedeUserID，DedeUserID__ckMd5，SESSDATA三个值保存下来就行了。

## 发弹幕

发弹幕的请求：

```
POST:http://live.bilibili.com/msg/send
```

需要提交表单，有六个字段。
"color=颜色&fontSize=字体大小&mode=模式&msg=内容&rnd=时间&roomId=房间号"
颜色填固定值16777215，字体大小25，mode填1。
注意把你的Cookie添加到你的http header里去。
形如：

```
Cookie: DedeUserID=241087; DedeUserID__ckMd5=7e772bf49d639da6; SESSDATA=ed6f17cd%2C1458751426%2C14f99d59;
```

成功返回的json:

```
{"code":0,"msg":"","data":[]}
```

---
能够模拟登陆之后，能够调用的API就很多了，发弹幕只是其中一个简单的例子，剩下的就自己尝试吧w。
