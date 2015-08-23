---
layout: post
title: "Timer ⌚回调事件被UI交互阻塞的解决办法"
date: 2015-08-23 14:59:03 +0800
comments: true
published: true
sharing: true
footer: true
categories: objc
---

分析原因：

UI 事件运行在主线程，并且UI 事件的运行模式下 runloop 过滤掉了 timer 事件。

回顾一下 runloop 的运行机制我们知道，runloop 是一个处理事件的循环，为了提高效率和隔离事件，它将事件进行了模式划分，通过 Runloop mode 的方式进行分类，并提供了 `runMode:beforeDate:` 方法 或者 `CFRunLoopRunInMode(CFStringRef mode, CFTimeInterval seconds, Boolean returnAfterSourceHandled)` 函数。所以 runloop 在 UI 事件进行中，始终运行在 NSRunLoopCommonModes 模式下。

### Runloop 的模式有这几种：

<a href="https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html" target="_blank">Runloop modes</a>>

#### Default: NSDefaultRunLoopMode(Cocoa), kCFRunLoopDefaultMode(Core Foundation)

> The default mode is the one used for most operations. Most of the time, you should use this mode to start your run loop and configure your input sources.

#### Connection: NSConnectionReplyMode (Cocoa)

> Cocoa uses this mode in conjunction with NSConnection objects to monitor replies. You should rarely need to use this mode yourself.

#### Modal: NSModalPanelRunLoopMode (Cocoa)

> Cocoa uses this mode to identify events intended for modal panels.

#### Event tracking: NSEventTrackingRunLoopMode (Cocoa)

> Cocoa uses this mode to restrict incoming events during mouse-dragging loops and other sorts of user interface tracking loops.

#### Common modes: NSRunLoopCommonModes (Cocoa), kCFRunLoopCommonModes (Core Foundation)

> This is a configurable group of commonly used modes. Associating an input source with this mode also associates it with each of the modes in the group. For Cocoa applications, this set includes the default, modal, and event tracking modes by default. Core Foundation includes just the default mode initially. You can add custom modes to the set using the CFRunLoopAddCommonMode function.

### 解决方案有以下两种：

#### 1. 在其他线程上运行 timer，并在需要的时候将操作发送到主线程

objc:

```
- (void)startTimer
{
    NSThread *th = [[NSThread alloc] initWithTarget:self selector:@selector(threadRun) object:nil];
    [th start];
}

- (void)addInfinitTimerToCurrentRunloopWithTarget:(id)target selector:(SEL)selector inMode:(NSString *)mode
{
    NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timerRun:) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:mode];
}

- (void)threadRun
{
    [self addInfinitTimerToCurrentRunloopWithTarget:self selector:@selector(timerRun:) inMode:@"MyTimerMode"];

    while (YES) {
        NSLog(@"%s while..", __FUNCTION__);
        
        [[NSRunLoop currentRunLoop] runMode:@"MyTimerMode" beforeDate:[NSDate distantFuture]];
    }
}

- (void)timerRun:(id)sender
{
    NSLog(@"~");
    [self performSelectorOnMainThread:@selector(someSelector) withObject:nil waitUntilDone:YES];
}

- (void)someSelector
{
	//....
}

```

这里用 NSThread 和自定义的计时器模式 `@"MyTimerMode"` 来实现。


#### 2.在主线程加入计时器

objc:

```
- (void)addInfinitTimerToCurrentRunloopWithTarget:(id)target selector:(SEL)selector inMode:(NSString *)mode
{
    NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timerRun:) userInfo:nil repeats:YES];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:mode];
}

//运行在主线程上

[self addInfinitTimerToCurrentRunloopWithTarget:self selector:@selector(timerRun:) inMode:NSRunLoopCommonModes];

```

让 timer 所运行的 Runloop mode 和UI 滚动事件的 Runloop mode 一致即可。


