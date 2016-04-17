---
layout: post
title: "pip 使用国内的源"
date: 2016-04-02 00:23:47 +0800
comments: true
published: true
sharing: true
footer: true
categories:
---

# 修改 ~/.pip/pip.conf 文件

添加

```
[global]
index-url=http://mirrors.aliyun.com/pypi/simple/
# 或者 http://pypi.douban.com/simple/

```

# 添加 --trusted-host 信任站点

pip 会每次都提示使用 `--trusted-host` 来设置信任站点，这样未免太繁琐了

我们可以在 ~/.pip/pip.conf 中添加一下内容

```
[install]
trusted-host=mirrors.aliyun.com
# 或者 pypi.douban.com
```
