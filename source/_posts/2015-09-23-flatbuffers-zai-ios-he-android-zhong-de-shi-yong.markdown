---
layout: post
title: "FlatBuffers 在 iOS 和 Android 中的使用"
date: 2015-09-23 11:47:01 +0800
comments: true
published: false
sharing: true
footer: true
categories:
---

FlatBuffers is an efficient cross platform serialization library for C++, with support for Java, C# and Go. It was created at Google specifically for game development and other performance-critical applications.

> PS:还支持 Python

## iOS 平台
FlatBuffers 未对 Objective-C 提供支持，由于对 C++ 的支持，也可以用于 iOS 平台。

在客户端方面横向对比 JSONKit 和 FlatBuffers 的反序列化测试结果

### 时间
