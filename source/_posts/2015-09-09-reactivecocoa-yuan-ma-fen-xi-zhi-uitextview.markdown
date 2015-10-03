---
layout: post
title: "ReactiveCocoa 源码分析之UITextView"
date: 2015-09-09 16:53:35 +0800
comments: true
published: false
sharing: true
footer: true
categories: 3rd,RAC
---

一个简单案例分析，利用 RAC 来监听 UITextView 的文字改变。

测试代码：

```

    RACSignal *signal = [self.textView rac_textSignal];
    [signal subscribeNext:^(id x) {
        NSLog(@"subscribe next ");
    }];

```

当 textView 的文字改变，会得到一下结果：

```

Test[11820:295655] subscribe next 
Test[11820:295655] subscribe next 

```

UITextView (RACSignalSupport) 源码：

```

- (RACSignal *)rac_textSignal {
    @weakify(self);
    RACSignal *signal = [[[[[RACSignal
        defer:^{
            @strongify(self);
            return [RACSignal return:RACTuplePack(self)];
        }]
        concat:[self.rac_delegateProxy signalForSelector:@selector(textViewDidChange:)]]
        reduceEach:^(UITextView *x) {
            return x.text;
        }]
        takeUntil:self.rac_willDeallocSignal]
        setNameWithFormat:@"%@ -rac_textSignal", self.rac_description];

    RACUseDelegateProxy(self);

    return signal;
}

```

分析：

