---
layout: post
title: "libdispatch 源码分析"
date: 2015-08-18 08:57:37 +0800
comments: true
published: true
categories: objc
---

GCD 是苹果的多线程框架，提供了大量便捷的特性，允许用户利用多核 cpu 技术 ，使用中，最常接触的是 

```
dispatch_queue_t dispatch_get_global_queue()

dispatch_queue_t dispatch_queue_create()

void dispatch_async()

dispatch_semaphore_t dispatch_semaphor_create()

```

### 关键词

后面的描述中会用到一下几个关键词:

`dq`: dispatch_queue 的简写

`qos`: quality of service 的简写

GCD 虽然好用，但对于 GCD 的底层实现我们却知之甚少，好在苹果开源了 GCD 的源码项目 libdispatch，我们可以在这里下载目前最新的版本 [libdispatch](http://opensource.apple.com/tarballs/libdispatch/libdispatch-442.1.4.tar.gz)

### dispatch_queue_create() 函数

我们来看看 dispatch_queue_create 函数的定义

```
dispatch_queue_t
dispatch_queue_create(const char *label, dispatch_queue_attr_t attr)
{
	return dispatch_queue_create_with_target(label, attr,
			DISPATCH_TARGET_QUEUE_DEFAULT);
}
```

可以看到，其实 dispatch_queue_create 调用了私有函数 dispatch_queue_create_with_target ，移步到源码

```
// skip zero
// 1 - main_q
// 2 - mgr_q
// 3 - mgr_root_q
// 4,5,6,7,8,9,10,11,12,13,14,15 - global queues
// we use 'xadd' on Intel, so the initial value == next assigned
unsigned long volatile _dispatch_queue_serial_numbers = 16;

dispatch_queue_t
dispatch_queue_create_with_target(const char *label, dispatch_queue_attr_t dqa,
		dispatch_queue_t tq)
{
#if DISPATCH_USE_NOQOS_WORKQUEUE_FALLBACK
	// Be sure the root queue priorities are set
	dispatch_once_f(&_dispatch_root_queues_pred, NULL,
			_dispatch_root_queues_init);
#endif
	bool disallow_tq = (slowpath(dqa) && dqa != DISPATCH_QUEUE_CONCURRENT);
	if (!slowpath(dqa)) {
		dqa = _dispatch_get_queue_attr(0, 0, false, false);
	} else if (dqa->do_vtable != DISPATCH_VTABLE(queue_attr)) {
		DISPATCH_CLIENT_CRASH("Invalid queue attribute");
	}
	dispatch_queue_t dq = _dispatch_alloc(DISPATCH_VTABLE(queue),
			sizeof(struct dispatch_queue_s) - DISPATCH_QUEUE_CACHELINE_PAD);
	_dispatch_queue_init(dq);
	if (label) {
		dq->dq_label = strdup(label);
	}
	qos_class_t qos = dqa->dqa_qos_class;
	bool overcommit = dqa->dqa_overcommit;
#if HAVE_PTHREAD_WORKQUEUE_QOS
	dq->dq_priority = _pthread_qos_class_encode(qos, dqa->dqa_relative_priority,
			overcommit);
#endif
	if (dqa->dqa_concurrent) {
		dq->dq_width = DISPATCH_QUEUE_WIDTH_MAX;
	} else {
		// Default serial queue target queue is overcommit!
		overcommit = true;
	}
	if (!tq) {
		if (qos == _DISPATCH_QOS_CLASS_UNSPECIFIED) {
			qos = _DISPATCH_QOS_CLASS_DEFAULT;
		}
#if DISPATCH_USE_NOQOS_WORKQUEUE_FALLBACK
		if (qos == _DISPATCH_QOS_CLASS_USER_INTERACTIVE &&
				!_dispatch_root_queues[
				DISPATCH_ROOT_QUEUE_IDX_USER_INTERACTIVE_QOS].dq_priority) {
			qos = _DISPATCH_QOS_CLASS_USER_INITIATED;
		}
#endif
		bool maintenance_fallback = false;
#if DISPATCH_USE_NOQOS_WORKQUEUE_FALLBACK
		maintenance_fallback = true;
#endif // DISPATCH_USE_NOQOS_WORKQUEUE_FALLBACK
		if (maintenance_fallback) {
			if (qos == _DISPATCH_QOS_CLASS_MAINTENANCE &&
					!_dispatch_root_queues[
					DISPATCH_ROOT_QUEUE_IDX_MAINTENANCE_QOS].dq_priority) {
				qos = _DISPATCH_QOS_CLASS_BACKGROUND;
			}
		}

		tq = _dispatch_get_root_queue(qos, overcommit);
		if (slowpath(!tq)) {
			DISPATCH_CLIENT_CRASH("Invalid queue attribute");
		}
	} else {
		_dispatch_retain(tq);
		if (disallow_tq) {
			// TODO: override target queue's qos/overcommit ?
			DISPATCH_CLIENT_CRASH("Invalid combination of target queue & "
					"queue attribute");
		}
		_dispatch_queue_priority_inherit_from_target(dq, tq);
	}
	_dispatch_queue_set_override_priority(dq);
	dq->do_targetq = tq;
	_dispatch_object_debug(dq, "%s", __func__);
	return _dispatch_introspection_queue_create(dq);
}
```

看几个关键的地方 

`_dispatch_queue_init(dq);`

`dq->dq_width = DISPATCH_QUEUE_WIDTH_MAX;`

### dispatch_async()

`dispatch_async()` 会调用 `_dispatch_async_f()` 完成真正的异步派发任务的操作。

_dispatch_async_f() 中

```
	if (dq->dq_width == 1 || flags & DISPATCH_BLOCK_BARRIER) {
		return _dispatch_barrier_async_f(dq, ctxt, func, pp, flags);
	}

```

说明当队列宽度为1 的时候，会自动通过 `_dispatch_barrier_async_f` 加入到等待队列中。

```
	dc = fastpath(_dispatch_continuation_alloc_cacheonly());
	if (!dc) {
		return _dispatch_async_f_slow(dq, ctxt, func, pp, flags);
	}
```

从缓存中获取继续运行的队列如果没有，就通过 _dispatch_async_f_slow 从堆上面创建一个队列

### dispatch_sync


### dispatch_get_global_queue

`
return _dispatch_get_root_queue(qos, flags & DISPATCH_QUEUE_OVERCOMMIT);
`

通过全局 dq 的优先级别设置好 qos 属性，在从队列池 `_dispatch_root_queues[]` 中，取出一个 dq 出来，类似于这样:

```
return &_dispatch_root_queues[
				DISPATCH_ROOT_QUEUE_IDX_MAINTENANCE_QOS_OVERCOMMIT];
```

得到的队列底层代码信息:

```
[DISPATCH_ROOT_QUEUE_IDX_MAINTENANCE_QOS_OVERCOMMIT] = {
		.do_vtable = DISPATCH_VTABLE(queue_root),
		.do_ref_cnt = DISPATCH_OBJECT_GLOBAL_REFCNT,
		.do_xref_cnt = DISPATCH_OBJECT_GLOBAL_REFCNT,
		.do_suspend_cnt = DISPATCH_OBJECT_SUSPEND_LOCK,
		.do_ctxt = &_dispatch_root_queue_contexts[
				DISPATCH_ROOT_QUEUE_IDX_MAINTENANCE_QOS_OVERCOMMIT],
		.dq_label = "com.apple.root.maintenance-qos.overcommit",
		.dq_running = 2,
		.dq_width = DISPATCH_QUEUE_WIDTH_MAX,
		.dq_serialnum = 5,
	}
```
