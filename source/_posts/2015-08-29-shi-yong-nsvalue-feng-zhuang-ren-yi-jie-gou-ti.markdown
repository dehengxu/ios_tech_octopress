---
layout: post
title: "使用 NSValue 封装任意结构体"
date: 2015-08-29 12:55:21 +0800
comments: true
published: true
sharing: true
footer: true
categories: objc
---

## NSValue

装包，拆包，分别用到 `value:withObjCType:` 和 `getValue:`两个方法。

第一个是通过 type 类型，将 value 封装成 NSValue 对象。

后者是通过 NSValue 对象，获取值型数据

假设有一个自定义结构体

```
struct Node {
    struct Node *next;
    char *name;
    int age;
};
```

```
    struct Node *header = (struct Node *)malloc(sizeof(struct Node));
    header->name = "Nicholasxu";
    header->age = 89;    
    NSValue *value = [NSValue value:header withObjCType:@encode(struct Node)];
```

通过 `@encode` 将C 类型转换成 Objective-C 能识别的 char * 形式。

解包方法如下：

```
    struct Node *chain = (struct Node *)malloc(sizeof(struct Node));
    [value getValue:chain];
    NSLog(@"%s", chain->name);
	NSLog(@"%s", chain->age);

```

先分配结构题内存 `chain`，再将数据解包出来，并填充 `chain` 的空间。

### 内存布局

上面的例子中，`value` 和 header 都是在堆上分配的空间并且 header 的内存和 value 内部的数据是同一个内存空间，要做好 header 的内存管理工作，否则会导致 value 无法访问内部数据(出现野指针)。而且 chain 的数据是从 value 中复制过来的。


