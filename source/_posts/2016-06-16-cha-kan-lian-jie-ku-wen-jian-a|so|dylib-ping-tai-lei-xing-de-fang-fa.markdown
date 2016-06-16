---
layout: post
title: "查看链接库文件 a|so|dylib 平台类型的方法"
date: 2016-06-16 12:34:40 +0800
comments: true
published: true
sharing: true
footer: true
categories:
---
macOS:

1. 可以使用 `lipo -info some_static_lib.a` 来查看一个静态库的目标平台类型，一般结果是这样的:

```
lipo -info some_static_lib.a
Architectures in the fat file: some_static_lib.a are: armv7 arm64
```

  说明该静态库是针对 armv7 和 arm64 架构的cpu进行的编译。

2. 使用  `file some_dynamic_lib.so` 来查看一个动态库的目标平台类型，结果如下:

```
file armeabi/libumeng_opustool.so
armeabi/libumeng_opustool.so: ELF 32-bit LSB shared object, ARM, version 1 (SYSV), dynamically linked, stripped
```

  说明该动态库是 arm 动态链接库，采用了 ELF 规范的 32位动态库。
