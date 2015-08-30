---
layout: post
title: "objc_msgSend 编译错误的解决方法"
date: 2015-08-30 12:13:22 +0800
comments: true
published: true
sharing: true
footer: true
categories: 
---

最近在 Xcode6.4 上编译一个老旧的库时，出现了运行时函数 `objc_msgSend` 的编译错误，提示:

`Too many arguments to function call, excepted 0, have 7`，说函数不需要参数，却传入了7个参数。

原来是预编译指令的问题:

找到编译出错的那个库，选中 target -> Build Settings 搜索框中输入 `objc`

定位到这里:

![msgSend_fix](/images/refered/msgSend_error_fix.png)

将

`Enable Strict Checking of objc_msgSend Calls`  字段设为 `NO`

关闭 `objc_msgSend` 的严格调用检查。


