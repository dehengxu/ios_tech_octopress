---
layout: post
title: "自定义类如何提供 KVC 兼容特性"
date: 2015-08-29 15:10:33 +0800
comments: true
published: true
sharing: true
footer: true
categories: objc
tags: KVC, KVO
---

### KVC 兼容性

#### 对一的关系

对于属性来说，附属值会存在一对一的关系，这就需要你的类拥有一下特征：

* 实现 `-key`, `-is<Key>` 方法类，或者存在有一个叫 `key` 或者 `_key` 的实例对象。(显然，这是 getter 访问器的规则)。虽然 key 名字常常以小写字母开头，KVC 也支持 key名称以大写字母开头，例如：`URL`。

* 如果该属性是可以修改的，那么就应该实现 `-set<Key>:` 方法。PS:框架会自动寻找 `key` 或者 `_key`形式的变量来完成对应操作。

* 你的 `-set<Key>:` 实现方法不应该执行验证操作。

* 如果你需要验证 `key` 的话，你需要实现 `-validate<key>:error:` 的方法。

#### 索引化的对多关系

对于索引化的对多关系而言，KVC 兼容性需要你的类拥有这些特征：

* 实现名为 `-<key>` 并返回数组的方法。

* 或者拥有一个名为 `key` 或 `_key` 的数组实例对象变量。

* 或者实现 `-countOf<key>` 方法，并且实现 `-objectIn<key>AtIndex:` 和 `-<key>AtIndexes:` 两个方法中的一个或所有方法。

* 你也能可选的实现 `-get<key>:range:` 方法来提高性能表现。

对于可修改的索引化有序对多关系，KVC 兼容性也需要你的类提供这些特征：

* 实现 `-insertObject:in<key>AtIndex:` 和 `-insert<Key>:atIndexex:` 中的一个或所有方法。

* 当然如果愿意，你可也可以实现 `-replaceObjectIn<key>AtIndex:withObject:` 或 `-replace<key>AtIndexes:with<key>:` 来提高性能表现。这仅仅是个可选项。 :)

#### 无序的对多关系兼容性

对于无序的对多关系来说，KVC 兼容性需要你的类提供这些特征：

* 实现能返回集合的名为 `-<key>` 的方法。

* 或者包含一个名为 `key` 或 `_key` 的实例变量。

* 或者实现 `-countOf<key>`，`-enumeratorOf<key>` 和 `-memberOf<key>:` 方法。

对于可变的无序对多关系而言，KVC 兼容性需要你的类也提供这些：

* 实现方法 `-add<key>Object:` 或 `-add<key>:` 中的一个或所有。

* 实现方法 `-remove<key>Object:` 或 `-remove<key>:`中的一个或所有。

* 你也可以实现 `-intersect<key>:` 和 `-set<key>:` 来提高性能表现，不过这是个可选项，根据你面临的情况自己选择吧。