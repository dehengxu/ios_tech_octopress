---
layout: post
title: "GCD 的常规用法"
date: 2015-08-23 15:56:50 +0800
comments: true
published: true
sharing: true
footer: true
categories: objc, GCD
---

GCD 通过将多线程抽象为 `dispatch_queue_t` 类型的对象，以 Block 作为计算单元实现的一套基于 C 语言的开源多线程库 `libdispatch`。

GCD 让多线程开发变的更加方便，简洁。

首先明确几个 GCD 中的特性:

1. 全局队列都是异步的。
2. 主队列是同步的。
3. 串行队列中在当前上下文执行同步计算会产生死锁。

### 创建并行，串行队列

```
dispatch_queue_create("q1", DISPATCH_QUEUE_CONCURRENT);
```

```
dispatch_queue_create("q1", DISPATCH_QUEUE_SERIAL);
```

### 异步执行任务

```
disaptch_async(queue, ^() {
	//Todo....
});
```

### 同步执行任务的方式

1.    dispatch_semaphor_t

	```
	dispatch_semaphor_t semaphor = disaptch_semaphor_create(1);

	dispatch_async(queue, ^() {
		dispatch_semaphor_wait(semaphor);
		//Todo...

		//Finished.
		dispatch_semaphor_signal(semaphor);
	});

	```

	创建一个只允许一个线程访问的信号量 `disaptch_semaphor_create(1);`

	通知其他访问对象等待 `dispatch_semaphor_wait(semaphor);`

	资源访问结束后，退回占用的资源，并通知其他访问对象可以进行访问。 `dispatch_semaphor_signal(semaphor);`


2.    dispatch_group_wait

	```
	dispatch_group_t group = dispatch_group_create();

	dispatch_group_async(group, q1, ^ {
		//Task 1.
	});

	dispatch_group_async(group, q1, ^ {
		//Task 2.
	});

	dispatch_group_async(group, q1, ^ {
		//Task 3.
	});

	dispatch_async(queue, ^() {
		dispatch_group_wait(group);
		//Todo....

	});
	```

	`dispatch_group_async`: 添加任务到分组中。

	`dispatch_group_wait(group);` 等待 group 里的所有任务完成后方可继续。


3.    dispatch_group_notify

	```
	dispatch_group_notify(group, q1, ^ {
		//Todo...

	});
	```

	`dispatch_group_notify`: group中的所有任务完成后，执行 block 中的代码。

4.    dispatch_group_enter, dispatch_group_leave

	```
	dispatch_group_enter(group);

	dispatch_async(q1, ^ {
		//Todo...

		dispatch_group_leave(group);
	});

	```

	该代码等价于:

	```
	dispatch_group_async(q1, ^ {
		//Todo...
	});
	```

	作用也是将任务放入到 q1 队列中，同时也添加到分组中。

### 使用 apply 来提高并行效率

```
    dispatch_apply(5000, q1, ^(size_t i) {
        NSLog(@"%zu",  i);
    });
```

你会发现如果 q1 是并发队列，输出的数字是无序的状态。这就是 dispatch_apply 带来的好处，它能快速将当前的迭代任务立刻发送到队列中执行，如果是并发队列，这个效率是很高的。如果你对任务的顺序有要求，或者队列是串行队列，那就不需要使用 dispatch_apply 了。

### dispatch_barrier_async

阻碍异步，意思是通过 dispatch_barrier_async 提交的任务，不会立刻执行，而是会阻碍它之后提交的任务，直到它之前的任务全部完成，它才会执行，并且它执行完毕后，其后提交的任务才能继续进行。

而在 dispatch_barrier_async 之前或者之后的任务都是并发执行的。

```
dispatch_queue_t q = dispatch_queue_create("test", DISPATCH_QUEUE_CONCURRENT);

dispatch_async(q, ^() {printf("A\n");});
dispatch_async(q, ^() {printf("B\n");});
dispatch_async(q, ^() {printf("C\n");});
dispatch_async(q, ^() {printf("D\n");});

dispatch_barrier_async(q, ^() {printf("barrier\n");});

dispatch_async(q, ^() {printf("E\n");});
dispatch_async(q, ^() {printf("F\n");});
dispatch_async(q, ^() {printf("G\n");});


```

输出结果是:

```
A --
C  |  
D  |--> //顺序随机产生
B _|
barrier //这里的输出位置不会变化
F --
E  |--> //顺序随机产生
G --

```


但是 dispatch_barrier_async 只有提交到自己创建的并发队列时，才有上面的效果。如果是全局队列，或者串行队列，那么 dispatch_barrier_async 和 dispatch_async 是等效的。

