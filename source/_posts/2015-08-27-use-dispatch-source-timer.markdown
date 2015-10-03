---
layout: post
title: "Use dispatch source timer"
date: 2015-08-27 16:58:03 +0800
comments: true
published: true
sharing: true
footer: true
categories: objc
---

```

dispatch_queue_t queue = dispatch_queue_create("com.xxx.timer", DISPATCH_QUEUE_CONCURRENT);

dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);

dispatch_source_set_timer(timer, DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC, NSEC_PER_SEC / 10);

// 设置定时器回调 block
dispatch_source_set_event_handler(timer, ^ {
	//do something...
});

dispatch_resume(timer);

```

关键点： timer 一定要被 retain ，防止提前释放，否则 timer 无效。

关闭 timer 可以使用 `dispatch_source_cancel` 函数。


