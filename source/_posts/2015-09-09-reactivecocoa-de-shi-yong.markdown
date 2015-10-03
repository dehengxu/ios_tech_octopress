---
layout: post
title: "ReactiveCocoa 的使用"
date: 2015-09-09 16:46:11 +0800
comments: true
published: false
sharing: true
footer: true
categories: 3rd,RAC
---

## RACDelegateProxy

一个用来完成某些委托任务的代理类，通过运行时为该类动态适配必须的委托协议，由 RAC 来替你完成委托的实现部分，并将该委托方法的回调通过信号的机制来传递。

