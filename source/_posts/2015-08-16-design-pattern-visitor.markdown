---
layout: post
title: "设计模式-访问者模式"
date: 2015-08-16 08:05:25 +0800
comments: true
categories: 设计模式
---

### 模式说明：

> 访问者模式(Visitor Pattern)，是一种具有双分派能力的模式，它通过 Visitor 接口约定为已有类 Element 进行扩展，扩展的目标是尽可能少的修改已有类 Element ，在 Visitor 中增加新扩展的接口，对 Element 中的现有接口进行访问。而 Visitor 以外的部分，不需要知道 Element 的具体实现和操作，进行了封装。
> 所谓`双分派`指的是，该模式可以通过不同的 `Visitor` 对不同的 `Element` 进行访问。

举例：

> 场景介绍，以学校为背景，引入 Teacher, Student, Parent 三类人作为已有类型， 引入 ConcreteVisitor 作为 Visitor 的具体实现类，对以上三者进行扩展。

> `ElementProtocol` 作为 Teacher, Student, Parent 的公共接口协议。
> `VisitorProtocol` 作为访问者的公共接口协议。
> `Teacher, Student, Parent` 三者是具体的 Element 实现。
> `ConcreteVisitor` 是具体的 Visitor 实现。

当学校外面的人需要访问 `Teacher` 的时候，不需要直接与 `Teacher` 交流，而是去找 `ConcreteVisitor` 通过它与 `Teacher` 进行交流。


[Objective-c源码](git@github.com:xudeheng/ObjcDesignPatterns.git)
