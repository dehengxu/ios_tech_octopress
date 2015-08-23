---
layout: post
title: "bash 使用技巧"
date: 2015-08-23 23:11:02 +0800
comments: true
published: true
sharing: true
footer: true
categories: bash
---

#### 递归搜索目录下所有文件内容

bash

```
ls ./**/*.ext | xargs grep algorit
```

> 搜索当前目录下所有的 ext 文件，并且匹配是否包含关键词 `algorit`

#### 查找进程

```
es -ef|grep name
```

> 查找进程名包含 `name` 的进程

#### 查看开启的端口

```
lsof -i:8080
```

> 查看使用端口 8080 的进程


