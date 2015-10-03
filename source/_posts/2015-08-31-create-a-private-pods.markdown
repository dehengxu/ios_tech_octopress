---
layout: post
title: "Create a private pods"
date: 2015-08-31 22:20:30 +0800
comments: true
published: true
sharing: true
footer: true
categories:
---

### 1. 创建私有 Spec 仓库

没必要 fork CocoaPods 的Spec 主仓库。你可以建立自己的仓库，也不用公开。

而且你也可以创建自己的私有本地仓库

```
cd /opt/git
mkdir Specs.git
cd Specs.git
git init --bare
```

### 2. 添加私有仓库到 CocoaPods 安装文件中

`pod repo add REPO_NAME SOURCE_URL`

> 如果你想检测是否安装成功，可以这样测试：

```
cd ~/.cocoapods/repos/REPO_NAME
pod repo lint
```
### 3. 添加你的 podspec 到你自己的仓库

pod repo push REPO_NAME SPEC_NAME.podspec

---

最后，你需要在 podfile 文件中通过 `source` 指令来使用你自己的仓库：

```
source 'YOUR_REPO_URL'
```

[REF](https://guides.cocoapods.org/making/private-cocoapods.html)
