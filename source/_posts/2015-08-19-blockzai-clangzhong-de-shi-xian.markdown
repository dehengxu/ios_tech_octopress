---
layout: post
title: "Block在Clang中的实现"
date: 2015-08-19 06:00:02 +0800
comments: true
published: true
sharing: true
footer: true
categories: 
---

### 被导入的变量

#### 被导入成 const copy 变量

未被 `__block` 标记的自变量会被导入成 `const` 拷贝。

这个例子就导入了一个 `int` 类型变量:

```
int x = 10;
void (^vv)(void) = ^{ printf("x is %d\n", x); }
x = 11;
vv();
```

上面这段代码将会被编译成下面这2段内容:

```
struct __block_literal_2 {
    void *isa;
    int flags;
    int reserved;
    void (*invoke)(struct __block_literal_2 *);
    struct __block_descriptor_2 *descriptor;
    const int x;
};

void __block_invoke_2(struct __block_literal_2 *_block) {
    printf("x is %d\n", _block->x);
}

static struct __block_descriptor_2 {
    unsigned long int reserved;
    unsigned long int Block_size;
} __block_descriptor_2 = { 0, sizeof(struct __block_literal_2) };
```

```
struct __block_literal_2 __block_literal_2 = {
      &_NSConcreteStackBlock,
      (1<<29), <uninitialized>,
      __block_invoke_2,
      &__block_descriptor_2,
      x
 };
```

概括来说，标量(应该是指 int, float 这类值类型)，结构体，联合体 和 函数指针通常会被导入成 `const` 拷贝。

#### 将 Block 引用导入成 `const` 拷贝

第一种情况，当一个 `Block` 自身被导入的时候，copy 和 dispose 工具函数会使用到。这种情况下会需要用到一个 `copy_helper` 函数和一个 `dispose_helper` 函数。`copy_helper` 函数被传入当前栈的基指针和指向新的堆版本的指针，并被回调到运行时在 `Block` 中对被导入的域执行拷贝操作。运行时函数在这里有描述[Runtime Helper Functions](http://clang.llvm.org/docs/Block-ABI-Apple.html#runtimehelperfunctions).


简单的例子:

```
void (^existingBlock)(void) = ...;
void (^vv)(void) = ^{ existingBlock(); }
vv();

struct __block_literal_3 {
   ...; // existing block
};

struct __block_literal_4 {
    void *isa;
    int flags;
    int reserved;
    void (*invoke)(struct __block_literal_4 *);
    struct __block_literal_3 *const existingBlock;
};

void __block_invoke_4(struct __block_literal_2 *_block) {
   __block->existingBlock->invoke(__block->existingBlock);
}

void __block_copy_4(struct __block_literal_4 *dst, struct __block_literal_4 *src) {
     //_Block_copy_assign(&dst->existingBlock, src->existingBlock, 0);
     _Block_object_assign(&dst->existingBlock, src->existingBlock, BLOCK_FIELD_IS_BLOCK);
}

void __block_dispose_4(struct __block_literal_4 *src) {
     // was _Block_destroy
     _Block_object_dispose(src->existingBlock, BLOCK_FIELD_IS_BLOCK);
}

static struct __block_descriptor_4 {
    unsigned long int reserved;
    unsigned long int Block_size;
    void (*copy_helper)(struct __block_literal_4 *dst, struct __block_literal_4 *src);
    void (*dispose_helper)(struct __block_literal_4 *);
} __block_descriptor_4 = {
    0,
    sizeof(struct __block_literal_4),
    __block_copy_4,
    __block_dispose_4,
};
```

这里描述了 Block 如何被使用:

```
struct __block_literal_4 _block_literal = {
      &_NSConcreteStackBlock,
      (1<<25)|(1<<29), <uninitialized>
      __block_invoke_4,
      & __block_descriptor_4
      existingBlock,
};
```

#### 导入 __attribute__((NSObject)) 变量

GCC 在结构指针里引入饿了 `__attribute__((NSObject))` 概念，表示 “这是一个对象”。它很有用，因为很多低级别的数据结构被声明为透明的结构指针，例如`CFStringRef`, `CFArrayRef`, 等等。当在 C 里面使用的时候，仍然会有真实的对象存在，并且在第二种情况会请求拷贝，生成工具函数。编译器生成的贝工具函数需要使用运行时工具函数 `_Block_object_assign` ，并且在调用工具函数的时候，运行时函数 `_Block_object_dispose` 会被调用。

举个例子，下面的`Block` foo:

```
struct Opaque *__attribute__((NSObject)) objectPointer = ...;
...
void (^foo)(void) = ^{  CFPrint(objectPointer); };
```

会生成下面的工具函数:

```
void __block_copy_foo(struct __block_literal_5 *dst, struct __block_literal_5 *src) {
     _Block_object_assign(&dst->objectPointer, src-> objectPointer, BLOCK_FIELD_IS_OBJECT);
}

void __block_dispose_foo(struct __block_literal_5 *src) {
     _Block_object_dispose(src->objectPointer, BLOCK_FIELD_IS_OBJECT);
}
```


<a href="http://clang.llvm.org/docs/Block-ABI-Apple.html" target="_blank">block底层实现的官方文档</a>
