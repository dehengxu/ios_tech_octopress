---
layout: post
title: "Objective-C中的协议引用头文件的问题"
date: 2015-08-16 00:59:41 +0800
comments: true
categories: 
---

Objective-C 中循环引用头文件，会导致，头文件中声明的符号无法识别的问题。典型的问题是:

类 `SomeClass` 不被识别，必须在头文件中通过前导生命的方式让编译器认识该类 `@class SomeClass;`