---
layout: post
title: "通过 clang 分析 block 的机制"
date: 2015-08-23 08:35:18 +0800
comments: true
published: true
sharing: true
footer: true
categories: 
---

### 简单看一下编译器对 __block 做了什么

#### 1. 普通变量在栈空间上

objc:

```
- (void)useBlock
{
	int  = 2;
}
```

c++:

```
static void _I_Block4Clang_useBlock(Block4Clang * self, SEL _cmd) {
    int i = 2;

}
```

> 很简单的处理——没有任何处理，变量依然分配在了栈空间上。

#### 2. __block 修饰的变量

objc:

```
- (void)useBlock
{
	int __block i = 2;
}
```

c++:

```

struct __Block_byref_i_0 {
  void *__isa;
__Block_byref_i_0 *__forwarding;
 int __flags;
 int __size;
 int i;
};

static void _I_Block4Clang_useBlock(Block4Clang * self, SEL _cmd) {
    __attribute__((__blocks__(byref))) __Block_byref_i_0 i = {(void*)0,(__Block_byref_i_0 *)&i, 0, sizeof(__Block_byref_i_0), 2};

}

```

我们看到如果是 block 修饰的变量，虽然定义在在栈空间上，但是编译器会将它分配到堆的空间上(编译器的做法是，通过 `__Block_byref_i_0` 结构来封装变量 `i`)，这样的好处是 block 可以通过 `__Block_byref_i_0 *` 指针访问堆上的变量 i 了。

### 再看一下编译器对 block 做了什么

#### 1. 空 block。

objc:

```
- (void)useBlock
{
  ^(){};
}
```

c++:

```
//这是 block 的c++定义
struct __Block4Clang__useBlock_block_impl_0 {
  //这是 block 实现的结构体类型
  struct __block_impl impl;
  //这是 block 描述信息的结构体类型
  struct __Block4Clang__useBlock_block_desc_0* Desc;

  //这是构造函数
  __Block4Clang__useBlock_block_impl_0(void *fp, struct __Block4Clang__useBlock_block_desc_0 *desc, int flags=0) {
  	//指明了 block 类型(分配在栈上的block)
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    //对应的函数指针，这才是实际执行的代码
    impl.FuncPtr = fp;
    //block描述体，包含了数据信息
    Desc = desc;
  }
};

/**
 * block 被转成的 c++ 函数定义
 * struct __Block4Clang__useBlock_block_impl_0 *__cself 指向 block 对象本身
 */
static void __Block4Clang__useBlock_block_func_0(struct __Block4Clang__useBlock_block_impl_0 *__cself) {


    }
//block 描述信息的结构申明
static struct __Block4Clang__useBlock_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __Block4Clang__useBlock_block_desc_0_DATA = { 0, sizeof(struct __Block4Clang__useBlock_block_impl_0)};

//实际调用 block 的函数
static void _I_Block4Clang_useBlock(Block4Clang * self, SEL _cmd) {

    ((void (*)())&__Block4Clang__useBlock_block_impl_0((void *)__Block4Clang__useBlock_block_func_0, &__Block4Clang__useBlock_block_desc_0_DATA));
}
```

#### 2.引用栈变量并修改它

objc:

```
- (void)useBlock
{
    __block int i = 2;
    ^(){
        i = 8;
    };
}
```

c++:

```
struct __Block_byref_i_0 {
  void *__isa;
__Block_byref_i_0 *__forwarding;
 int __flags;
 int __size;
 int i;
};

struct __Block4Clang__useBlock_block_impl_0 {
  struct __block_impl impl;
  struct __Block4Clang__useBlock_block_desc_0* Desc;

  //i 变量被封装到了 __Block_byref_i_0 结构内部
  __Block_byref_i_0 *i; // by ref
  __Block4Clang__useBlock_block_impl_0(void *fp, struct __Block4Clang__useBlock_block_desc_0 *desc, __Block_byref_i_0 *_i, int flags=0) : i(_i->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __Block4Clang__useBlock_block_func_0(struct __Block4Clang__useBlock_block_impl_0 *__cself) {
  __Block_byref_i_0 *i = __cself->i; // bound by ref
  		//访问堆上的变量i，达到 block 可以修改被引用的变量的目的。
        (i->__forwarding->i) = 8;
    }

/*这里增加了一个 copy 函数，和 dispose 函数，分别用来复制 block 到堆中，释放堆上的 block 对象。*/

//通过  _Block_object_assign 将src->i 内容复制到 dst->i（我们可以推测，src 就是栈上的空间，dst就是堆上的空间，只有这样才能解释当 useBlock 的生命周期结束后，为什么 block 依然还能够执行）
static void __Block4Clang__useBlock_block_copy_0(struct __Block4Clang__useBlock_block_impl_0*dst, struct __Block4Clang__useBlock_block_impl_0*src) {_Block_object_assign((void*)&dst->i, (void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);}
//dispose 工具函数通过 _Block_object_dispose 来释放 src->i
static void __Block4Clang__useBlock_block_dispose_0(struct __Block4Clang__useBlock_block_impl_0*src) {_Block_object_dispose((void*)src->i, 8/*BLOCK_FIELD_IS_BYREF*/);}

static struct __Block4Clang__useBlock_block_desc_0 {
  size_t reserved;
  size_t Block_size;
  void (*copy)(struct __Block4Clang__useBlock_block_impl_0*, struct __Block4Clang__useBlock_block_impl_0*);
  void (*dispose)(struct __Block4Clang__useBlock_block_impl_0*);
} __Block4Clang__useBlock_block_desc_0_DATA = { 0, sizeof(struct __Block4Clang__useBlock_block_impl_0), __Block4Clang__useBlock_block_copy_0, __Block4Clang__useBlock_block_dispose_0};

//编译器将 useBlock 方法，转换成了 c++ 函数，和runtime的风格一样
static void _I_Block4Clang_useBlock(Block4Clang * self, SEL _cmd) {
	//在这里将 i 封装到了 __Block_byref_i_0 结构中
    __attribute__((__blocks__(byref))) __Block_byref_i_0 i = {(void*)0,(__Block_byref_i_0 *)&i, 0, sizeof(__Block_byref_i_0), 2};
    //这里才最终执行了 block 调用过程，并将函数指针(__Block4Clang__useBlock_block_func_0)，数据描述指针(&__Block4Clang__useBlock_block_desc_0_DATA)，数据封装(&i，这里的 i 是 __Block_byref_i_0 对象地址)
    ((void (*)())&__Block4Clang__useBlock_block_impl_0((void *)__Block4Clang__useBlock_block_func_0, &__Block4Clang__useBlock_block_desc_0_DATA, (__Block_byref_i_0 *)&i, 570425344));
}
```

#### 3.引用栈变量，但不修改它

objc:

```
- (void)useBlock
{
    int i = 2;
    ^(){
        int j = i;
        j++;
        printf("i = %d, j = %d\n", i, j);
    }();
}
```

c++:

```
struct __Block4Clang__useBlock_block_impl_0 {
  struct __block_impl impl;
  struct __Block4Clang__useBlock_block_desc_0* Desc;
  int i;
  __Block4Clang__useBlock_block_impl_0(void *fp, struct __Block4Clang__useBlock_block_desc_0 *desc, int _i, int flags=0) : i(_i) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
static void __Block4Clang__useBlock_block_func_0(struct __Block4Clang__useBlock_block_impl_0 *__cself) {
  int i = __cself->i; // bound by copy

        int j = i;
        j++;
        printf("i = %d, j = %d\n", i, j);
    }

static struct __Block4Clang__useBlock_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __Block4Clang__useBlock_block_desc_0_DATA = { 0, sizeof(struct __Block4Clang__useBlock_block_impl_0)};

static void _I_Block4Clang_useBlock(Block4Clang * self, SEL _cmd) {
    int i = 2;
    ((void (*)())&__Block4Clang__useBlock_block_impl_0((void *)__Block4Clang__useBlock_block_func_0, &__Block4Clang__useBlock_block_desc_0_DATA, i))();
}
```

