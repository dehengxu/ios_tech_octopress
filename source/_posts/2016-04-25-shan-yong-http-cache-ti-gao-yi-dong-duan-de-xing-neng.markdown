---
layout: post
title: "善用 HTTP cache 提高移动端的性能"
date: 2016-04-25 11:45:20 +0800
comments: true
published: false
sharing: true
footer: true
categories: 
---

## HTTP 缓存策略由各种相关的头部和状态码组成

HTTP 缓存的头部有以下几种:

`Cache-Control`, `Expires`, `ETag`, `Last-Modified`, `If-No-Matched`, 其中除了 `If-No-Matched` 是客户端的头部以外，其余的都是服务器所使用的头部信息。

下面分别说明一下各个头部的功能和作用，最后通过一个简单的 C/S 应用程序来实际演示 HTTP cache 的运作机制。

### Cache-Control

### Expires

### ETag

### Last-Modified

### If


