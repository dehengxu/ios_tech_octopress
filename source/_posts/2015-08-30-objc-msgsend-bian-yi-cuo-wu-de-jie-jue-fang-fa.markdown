---
layout: post
title: "Runtime 在新版本 Xcode 中引起的编译错误，及解决方法"
date: 2015-08-30 12:13:22 +0800
comments: true
published: true
sharing: true
footer: true
categories: Xcode
---


## objc_msgSend

最近在 Xcode6.4 上编译一个老旧的库时，出现了运行时函数 `objc_msgSend` 的编译错误，提示:

`Too many arguments to function call, excepted 0, have 7`，说函数不需要参数，却传入了7个参数。

原来是预编译指令的问题:

找到编译出错的那个库，选中 target -> Build Settings 搜索框中输入 `objc`

定位到这里:

![msgSend_fix](/images/refered/runtime_msgsend_fix.png)

将 `Enable Strict Checking of objc_msgSend Calls`  字段设为 `NO`

关闭 `objc_msgSend` 的严格调用检查。

## object->isa

最近在 Xcode6.4 上编译依赖 JSONKit 的时候，对于 JSONKit 内部通过指针访问 `isa` 变量代码，引起编译错误，提示:

`Assignment to Objective-C's isa is deprecated in favor of object_setClass()` 或者 `Assignment to Objective-C's isa is deprecated in favor of object_getClass()`

如图:

![isa_err](/images/refered/runtime_isa_error.png)

解决办法：找到 JSONKit 项目 target -> Build Settings 搜索框中输入 `isa`

定位到:

![isa_err](/images/refered/runtime_isa_fix.png)

修改为 `NO` 即可。





