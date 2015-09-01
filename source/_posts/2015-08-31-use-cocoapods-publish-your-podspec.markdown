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

如果你将 source 路径设置为 git@github.com.....，会出现这种警告提示：`- WARN  | Git SSH URLs will NOT work for people behind firewalls configured to only allow HTTP, therefore HTTPS is preferred.`。将 source 地址设置为 https://github.com.... 的形式，可以解决此问题。

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

这个步骤第一次执行，会比较慢，做好相应准备，比如翻好墙，以防长时间无响应。

比如我最近提交的: CoreDataEnvir

```Bash
pod trunk push CoreDataEnvir.podspec --verbose
/usr/bin/git pull --ff-only
    From https://github.com/CocoaPods/Specs
        c812aae..c4e2a00  master     -> origin/master
    Updating c812aae..c4e2a00
    Fast-forward
    CocoaPods-version.yml                              |    2 +-
    .../1.5/1PasswordExtension.podspec.json            |   37 +
    .../1.1/ABFRealmMapView.podspec.json               |   31 +
    .../1.1/ABFRealmSearchViewController.podspec.json  |   31 +
...
    Specs/wpxmlrpc/0.8.1/wpxmlrpc.podspec.json         |   27 +
    .../0.1.4/youtube-ios-player-helper.podspec.json   |   35 +
    .../0.2.2/youtube-parser.podspec.json              |   20 +
    Specs/yuntongxun/0.0.1/yuntongxun.podspec.json     |   36 +
    2832 files changed, 118838 insertions(+), 323 deletions(-)
    create mode 100644 Specs/1PasswordExtension/1.5/1PasswordExtension
    create mode 100644 Specs/ABFRealmMapView/1.1/ABFRealmMapView.podsp
    create mode 100644 Specs/ABFRealmSearchViewController/1.1/ABFRealm
    create mode 100644 Specs/ABFRealmTableViewController/1.0/ABFRealmT
    create mode 100644 Specs/ABLoader/0.0.2/ABLoader.podspec.json
...
    create mode 100644 Specs/tw10n/1.1.1/tw10n.podspec.json
    create mode 100644 Specs/twitter-text/1.13.0/twitter-text.podspec.json
    create mode 100644 Specs/wpxmlrpc/0.8.1/wpxmlrpc.podspec.json
    create mode 100644 Specs/youtube-ios-player-helper/0.1.4/youtube-ios-player-helper.podspec.json
    create mode 100644 Specs/youtube-parser/0.2.2/youtube-parser.podspec.json
    create mode 100644 Specs/yuntongxun/0.0.1/yuntongxun.podspec.json

 CocoaPods 0.39.0.beta.3 is available.
 To update use: `sudo gem install cocoapods --pre`
 [!] This is a test version we'd love you to try.

 For more information see http://blog.cocoapods.org
 and the CHANGELOG for this version http://git.io/BaH8pQ.

 Validating podspec
     CoreDataEnvir (0.4) - Analyzing on iOS platform.
   Preparing

 Analyzing dependencies

 Fetching external sources
 -> Fetching podspec for `CoreDataEnvir` from `/Users/nicholasxu/Projects/iOS/XFramework/CoreDataEnvir/CoreDataEnvir.podspec`

 Resolving dependencies of

 Comparing resolved specification to the sandbox manifest
   A CoreDataEnvir

 Downloading dependencies

 -> Installing CoreDataEnvir (0.4)
   > Copying CoreDataEnvir from
   `/Users/nicholasxu/Library/Caches/CocoaPods/Pods/External/CoreDataEnvir/25047baf5458c8928d40aa32eccec759-043e4` to
   `../../../../../../private/var/folders/6r/flwjnj4x4wsb8wcbmjj88_z40000gn/T/CocoaPods/Lint/Pods/CoreDataEnvir`
   - Running pre install hooks

```

经过了漫长的 Pods master specs 的下载过程，才开始进行库的验证过程，每一个 tag 都会验证，这个过程也很漫长，好在过去的每一个 tag 我都执行过验证。

> PS:如果需要提交到私有库，你需要执行 `pod trunk push CoreDataEnvir.podspec`


### 如果你的验证阶段经常 failed，有两种解决办法：

1. 解决所有的 warning, 配置好 podspec 文件。
2. 在lint 后面增加一个参数 `--allow-warning`，为了看清楚出错的原因你也可以增加  `--verbose`。

---

[Ref](https://guides.cocoapods.org/making/getting-setup-with-trunk.html)
[podspec](https://guides.cocoapods.org/syntax/podspec.html)
