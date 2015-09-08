---
layout: post
title: "使用 cocoapods 来管理基于 svn 的子项目"
date: 2015-09-08 16:01:14 +0800
comments: true
published: true
sharing: true
footer: true
categories: 
---

首先，感谢 clarkda 为cocoapods 提供了这个 repo-svn 的 svn 支持插件。

根据 [https://github.com/clarkda/cocoapods-repo-svn](https://github.com/clarkda/cocoapods-repo-svn) 上的说明

```

gem install cocoapods-repo-svn 来安装 repo-svn 插件。

pod repo-svn add my-svn-repo http://svn-repo-url 添加你的私有 svn 项目仓库

pod repo-svn update my-svn-repo 更新项目

pod repo-svn remove my-svn-repo 删除项目

```

其实不添加项目也是可以的，需要为 svn 上的子项目添加 .podspec 文件，使项目可被 cocoapods 识别并管理，并为你自己的项目添加 Podfile 文件。

#### 第一步：为你用到的所有子项目，添加 podspec

pod spec create my-svn-repo

编辑 libname.podspec 文件，主要设置好 s.source, s.source_files, s.exclude_files, s.framework, s.library 字段。

其中 s.source 的字段需要把地址改成 :svn => "http://svnurl/"

#### 第二步：编辑 Podfiles

加上 pod 'my-svn-repo', :svn => "http://10.255.223.227:81/svn/readersdk/cppsdk/branches/2.2.1_buildReaderKit"

现在，就可以享受 cocoapods 为你的 svn 项目带来自动化配置管理了。