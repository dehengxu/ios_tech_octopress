---
layout: post
title: "Use cocoapods publish your podspec"
date: 2015-08-31 22:10:34 +0800
comments: true
published: true
sharing: true
footer: true
categories: Cocoapods
---

### 创建 podspec

pod spec create NAME

### 补完 podspec 文件

参考例子：

```Ruby
Pod::Spec.new do |spec|
  spec.name             = 'Reachability'
  spec.version          = '3.1.0'
  spec.license          = { :type => 'BSD' }
  spec.homepage         = 'https://github.com/tonymillion/Reachability'
  spec.authors          = { 'Tony Million' => 'tonymillion@gmail.com' }
  spec.summary          = 'ARC and GCD Compatible Reachability Class for iOS and OS X.'
  spec.source           = { :git => 'https://github.com/tonymillion/Reachability.git', :tag => 'v3.1.0' }
  spec.source_files     = 'Reachability.h,m'
  spec.framework        = 'SystemConfiguration'
  spec.requires_arc     = true
end
```

[Offical Docs](https://guides.cocoapods.org/making/specs-and-specs-repo.html)

### 验证代码

`pod lib lint CoreDataEnvir.podspec`

### 验证 podspec

`pod spec lint CoreDataEnvir.podspec`

### 注册 Cocoapods 会话

为了保证与 Cocoapods 通信的合法性，需要在你的电脑上注册一个会话，方法如下：

```Bash
pod trunk register orta@cocoapods.org 'Orta Therox' --description='macbook air'
```

执行完后，你的邮箱会收到一封激活邮件，点击其中的激活链接后，你的会话就算是合法了。

### 提交你的 podspec

`pod trunk push CoreDataEnvir.podspec` 这条命令会把你的 podspec 提交到公共库中。

如果需要提交到私有库，你需要执行 `pod trunk push REPO CoreDataEnvir.podspec`
