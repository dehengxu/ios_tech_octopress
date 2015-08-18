---
layout: post
title: "使用现代Objective-C"
date: 2015-08-16 15:32:41 +0800
comments: true
categories: 
---

现代 Objective-C 比传统 Objective-C 增加了很多优雅的语法和特性。

### instancetype

instancetype 和 id de区别在于 instancetype 返回值在编译时会将对象与对应的类进行方法匹配，而 id 则作为万用类型，不会提示任何的警告，因为 id 类型包含所有类型，所以发送任何消息，编译器都认为是正确的。这就造成了安全隐患，也是Apple 极力推荐使用 instancetype 的原因。

### Properties

### 枚举宏

将 enum 换成 NS_ENUM 和 NS_OPTIONS

### 对象初始化

> 宏 NS_DESIGNATED_INITIALIZER

### 自动引用计数 ARC
