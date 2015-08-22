---
layout: post
title: "Objective-C block usage"
date: 2015-08-15 14:43:40 +0800
comments: true
categories: objc
---

## Block syntax

> Block description

### 1. Use block as callback

```
int (^maxBlock)(int, int);
maxBlock = ^(int a, int b) {
	return a > b ? a : b;
}

NSLog(@"max is %d", maxBlock(89, 22));

```

### 2. Use block as parameter
```
- (void)someMethod:(void(^)(void))action;

[self someMethod:^() {
	NSLog(@"Do some operation in block.");
}];


```
### 3. Use block in recurcive callback

```
void testFunc() {
	// It looks well.
	int(^fibonacci)(int) = ^(int n) {
		if (n <= 1) return n;

		return fibonacci(n - 1) + fibonacci(n - 2);
	}

	fibonacci(10);
}
//It raise exception in runtme.
//CompositeTests testExample]_block_invoke(.block_descriptor=0x00007fe0da577710, folder=0x00007fe0da575c50) + 766 at CompositeTests.m:49, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x10)

```

> So, you should add `static` before block declaring, because at this time block is not created and it raise BAD_ACCESS to memory.

### 4. __block vs __weak

我们为了防止 block 将对象进行自动强引用导致出现引用循环，有时候会使用 __block 进行 weakRef 的声明，但是  __weak 也可以达成这个目的，那么 __block 和 __weak 有什么区别呢?

我们通常会这么做，这种情况 __block 和 __weak 似乎看不出什么区别
```
__block weskRef = obj

void(^aBlock)() = ^() {
	NSLog@("%@", weakRef);
}

aBlock();

```

可是如果这样呢
```
__block weskRef = obj;

void(^aBlock)() = ^() {
	NSLog@("%@", weakRef);
}

aBlock();

obj = nil;
```
#### __block

> __block 会保证 weakRef 对象一直有效，直到 block 本身不再使用 weakRef 才会真正释放掉 obj

#### __weak

> __weak 则会在 `obj = nil`  的同时将 weakRef 置为 `nil`。

这就是 `__block` 和 `__weak` 的区别。


