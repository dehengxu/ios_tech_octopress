---
layout: post
title: "gitsh 的安装和使用"
date: 2015-09-15 17:50:13 +0800
comments: true
published: true
sharing: true
footer: true
categories: 
---

项目地址:  https://github.com/thoughtbot/gitsh

### 前提：

1. 需要安装 ruby 环境

2. 安装 m4 编译工具

### 安装步骤：

1. 下载项目

```
git clone https://github.com/thoughtbot/gitsh.git
```

2. 安装 gems

国内用户可以修改  Gemfile 中的 `source 'https://rubygems.org'` 为 source 'http://ruby.taobao.org'

因为某些原因使用淘宝的 ruby gems 源可达性更有保障

执行  `bundle install`

3. 执行 `./autogen.sh`

4. 配置环境

```
export PATH="$PATH:/PROJECT/gitsh/bin"
```

最后终于可以使用了

### 使用

任何位置输入 `gitsh` 会进入到 git 命令行环境中，最大的优点是，git 命令可以只输入子命令就可以了。

比如：`git branch` 可以改用 `branch`，就这么简单。

直接回车会提示当前文件的变化情况，一切都变得容易了很多。

如果不想使用 gitsh 也可以安装  `zsh` 命令行终端 [zsh](http://www.zsh.org/)；再配合我的 [gitbash](https://github.com/xudeheng/gitbash) 简易别名系统简化 git 命令，安装命令 `curl -sSL https://raw.githubusercontent.com/xudeheng/gitbash/master/install.sh | bash`。




