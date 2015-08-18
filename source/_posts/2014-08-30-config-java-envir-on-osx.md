---
layout: post
title: "Config java envir on OSX"
date: 2014-08-30 14:45:49 +0800
comments: true
categories: Java
---

Error message:

> /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home/bin: No such file or directory

Reason:

> JAVA_HOME environment variable incorrect, It should be below.

Add line into file `~/.profile`.

```
export PATH="$PATH:/usr"
```

路径根据实际情况来定，有的系统可能是`/usr/local/java`

我的路径布局是 `/usr/bin`
