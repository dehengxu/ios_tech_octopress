---
layout: post
title: "Block在Clang中的实现"
date: 2015-08-19 06:00:02 +0800
comments: true
published: true
sharing: true
footer: true
categories: objc
---

### 被导入的变量

#### 被导入成 const copy 变量

未被 `__block` 修饰的自变量会被导入成 `const` 拷贝。

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

第一种情况，当一个 `Block` 自身被导入的时候，copy 和 dispose 工具函数会使用到。这种情况下会需要用到一个 `copy_helper` 函数和一个 `dispose_helper` 函数。`copy_helper` 函数被传入当前栈的基指针和指向新的堆版本的指针，并被回调到运行时在 `Block` 中对被导入的域执行拷贝操作。运行时函数在这里有描述<a href="http://clang.llvm.org/docs/Block-ABI-Apple.html#runtimehelperfunctions" target="__blank">Runtime Helper Functions</a>.


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

### 导入 `__block` 修饰的变量

#### `__block`修饰的变量的布局

编译器必须将 `__block` 修饰的变量放置到一个特殊的结构体中:

```
struct _block_byref_foo {
    void *isa;
    struct Block_byref *forwarding;
    int flags;   //refcount;
    int size;
    typeof(marked_variable) marked_variable;
};
```

当对一个 Block 引用进行 `Block_copy()` 和  `Block_release()` 操作的时候， 某种类型的变量会需要用到工具函数。在“C”级别只有`Block`类型的变量，或者 `__attribute__((NSObject))` 修饰的变量需要工具函数。在 Objective-C 中的对象 和 C++ 中栈上的对象也需要工具函数。需要工具函数的变量使用这种形式:

```
struct _block_byref_foo {
    void *isa;
    struct _block_byref_foo *forwarding;
    int flags;   //refcount;
    int size;
    // helper functions called via Block_copy() and Block_release()
    void (*byref_keep)(void  *dst, void *src);
    void (*byref_dispose)(void *);
    typeof(marked_variable) marked_variable;
};
```

结构体会像这样初始化:

> a. `forwarding` 指针指向外围结构体的开头。
> b. `size` 域被初始化为外围结构体的总大小。
> c. `flags` 域当没有工具函数的时候设置为0，反之被设置成 (1<<25)。
> d. 工具函数被初始化（如果是当前）。
> e. 变量自身呗设置为初始值。
> f. `isa` 域被设置为 `NULL`。

#### 在词法范围内访问`__block`变量

为了通过 `copy_helper` 操作编译器 “移动” 变量到堆里面，必须简介通过结构体指针`forwarding`来重写访问这个变量。

```
int __block i = 10;
i = 11;
```

会被重写成:

```
struct _block_byref_i {
  void *isa;
  struct _block_byref_i *forwarding;
  int flags;   //refcount;
  int size;
  int captured_i;
} i = { NULL, &i, 0, sizeof(struct _block_byref_i), 10 };

i.forwarding->captured_i = 11;
```

在 Block 引用变量被 `__block` 修饰后，工具代码必须用到 `_Block_object_assign` 和 `_Block_object_dispose` 这两个运行时提供的配套函数来进行拷贝。例如:

```
__block void (voidBlock)(void) = blockA;
voidBlock = blockB;
```

会被转换成:

```
struct _block_byref_voidBlock {
    void *isa;
    struct _block_byref_voidBlock *forwarding;
    int flags;   //refcount;
    int size;
    void (*byref_keep)(struct _block_byref_voidBlock *dst, struct _block_byref_voidBlock *src);
    void (*byref_dispose)(struct _block_byref_voidBlock *);
    void (^captured_voidBlock)(void);
};

void _block_byref_keep_helper(struct _block_byref_voidBlock *dst, struct _block_byref_voidBlock *src) {
    //_Block_copy_assign(&dst->captured_voidBlock, src->captured_voidBlock, 0);
    _Block_object_assign(&dst->captured_voidBlock, src->captured_voidBlock, BLOCK_FIELD_IS_BLOCK | BLOCK_BYREF_CALLER);
}

void _block_byref_dispose_helper(struct _block_byref_voidBlock *param) {
    //_Block_destroy(param->captured_voidBlock, 0);
    _Block_object_dispose(param->captured_voidBlock, BLOCK_FIELD_IS_BLOCK | BLOCK_BYREF_CALLER)}
```

和

```
struct _block_byref_voidBlock voidBlock = {( .forwarding=&voidBlock, .flags=(1<<25), .size=sizeof(struct _block_byref_voidBlock *),
    .byref_keep=_block_byref_keep_helper, .byref_dispose=_block_byref_dispose_helper,
    .captured_voidBlock=blockA )};

voidBlock.forwarding->captured_voidBlock = blockB;
```
#### 向 `Blocks` 中导入 `__block` 变量


<a href="http://clang.llvm.org/docs/Block-ABI-Apple.html" target="_blank">block底层实现的官方文档</a>
