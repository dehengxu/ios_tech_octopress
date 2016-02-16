---
layout: post
title: "j2objc 的使用和常见问题"
date: 2015-09-29 15:12:29 +0800
comments: true
published: false
sharing: true
footer: true
categories: 
---

Issues:

`_u_charDirection", referenced from:  _JavaLangCharacter_getDirectionalityWithInt_ in libjre_emul.a`

> 添加 libicucore 库引用

[Ref](https://groups.google.com/forum/#!topic/j2objc-discuss/0F3ITf7wEnU)

`adler32", referenced from: -[JavaUtilZipAdler32 updateWithInt:] in libjre_emul.a`

> 添加 libz 库引用

[Ref](https://github.com/google/j2objc/issues/340)
