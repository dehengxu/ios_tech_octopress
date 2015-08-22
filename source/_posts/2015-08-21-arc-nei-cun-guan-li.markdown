---
layout: post
title: "ARC 内存管理"
date: 2015-08-21 23:06:08 +0800
comments: true
published: true
sharing: true
footer: true
categories: 
tags: ARC
---

<a href="https://developer.apple.com/library/ios/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226-CH1-SW13" target="_blank">ARC Overview</a>

ARC 概述
 在编译时通过增加额外的代码来确保对象生命周期长度足够所需，但是概念上它通过增加合适的内存管理的代码调用来遵循和 MRC 同样的内存管理原则。

为了让编译器生成正确的代码，ARC 严格限制你能用的方法，以及如何使用自由桥接(toll-free bridging)。ARC 也为对象引用和属性生命增加了新的生命周期修饰符。

ARC 在 Xcode 4.2 OSX v10.6 he v10.7 ， 以及 iOS4， iOS5中提供支持。若引用不被 OSX v10.6 和 iOS4 支持。

对象属性默认情况下是 `strong` 属性；

ARC 会保证在最后一个对象引用使用前，不会释放掉该对象。

### ARC 的强制新规则

当使用其他编译器模式的时候，ARC 会暗示一些新的规则。规则会提供一个完全可靠的内存管理模式；在一些情况下，他们简单的强制最好的方式，在一些其他的情况下，规则会简化你的代码，或者你完全不用去处理内存管理。如果你违反了这些规则，你会立刻得到一个编译时错误，而不是一个运行时出现的轻微bug。

* 你不能显示调用 `dealloc`，或着实现，或者调用 `retain`, `release`, `retainCount` 和 `autorelease`。 

同时也不允许使用 @selector(retain), @selector(release) 等等。  

你可以实现 `dealloc` 方法，如果你需要管理一些资源而不是释放一些实例变量。
 
 ARC 中的自定义 `dealloc` 方法不需要调用 `[super dealloc]`(它会引起编译错误)。调用 super 的过程会被编译器自动并强制执行。

 你仍然能调用 CFRetain, CFRelease, 和其他相关的 Core Foundation风格对象的函数。

 * 你不能使用 `NSAllocateObject` 或者 `NSDeallocateObject`。

你使用 `alloc` 创建对象；运行时来负责释放对象；

* 你不能在 C 结构体中使用对象指针。

你能创建 `Objective-C` 对象来管理数据，而不是用 `struct`。

* id 和 void * 之间不能进行转换。

你必须使用特定的转换方式告诉编译器对象的生命期，你需要这样在 Objective-C 和 Core Foundation 类型之间转换，来做为你传递的函数参数。更多细节，请看 [Managing Toll-Free Bridging]()

* 你不能使用 `NSAutoreleasePool` 对象。

ARC 提供了 `@autoreleasepool` 块来代替它。这些比 `NSAutoreleasePool` 的效率更有优势。

* 你不能使用内存区

不再需要使用 `NSZone`，他们被现代 `Objective-C` 运行时忽略掉了。

为了和 MRC 代码进行交互，ARC 强制约束方法的命名：

* 你不能给访问器用 new 开头起名。

```
//不能正常工作，编译会提示错误。
@property NSString *newName;

//这样的命名是没问题的。
@property NSString *theNewName;
```

### ARC 带来了新的生命期修饰符

ARC 为对象带来了一些新的生命期修饰符，还有弱引用。弱引用不会延长对象的生命期，但是会在没有强引用的时候自动设置为 `nil`。

你可以使用这些修饰符的优势来管理程序中的对象。特殊的是，ARC 不会保护强循环引用。合理的运用弱关系会帮助你避免出现引用循环。

ARC 下，`strong` 是对象类型属性的默认配置。

下面看一下各种修饰符：

```
__strong
__weak
__unsafe_unretained
__autoreleasing
```

* `__strong` 是默认的配置。如果是强指针的时候，对象需要保证生命足够长。

* `__weak` 指定了引用不会保持对象引用。当没有其他的强引用指向它时，弱引用会设置成 `nil`。

* `__unsafe_unretained` 指定一个引用既不保持对象引用，也不会在不使用时设置为 `nil`。

* `__autoreleasing` 被用来描述引用类型的参数在方法 return 之后会自动被释放。

你需要正确的修饰变量，当对一个对象申明使用修饰符的时候，正确的格式如下：

```
ClassName * qulifier variableName;
```

例如:

```
NSString * __weak name;
NSString * __unsafe__unretained anotherName;
```

你需要小心地在栈上面使用 __weak 变量，参考下面的例子：

```
NSString * __weak string = [[NSString alloc] initWithFormat:@"First Name: %@", [self firstName]];
NSLog(@"string: %@", string);
```
这种情况，编译器会给出警告。因为没有其他对字符串的强引用，所以这个弱引用，会立即释放掉。

你也需要小心的在传參引用的时候使用对象。下面的代码工作正常：

```
NSError *error;
BOOL OK = [myObject performOperationWithError:&error];
if (!OK) {
    // Report the error.
    // ...
```

无论如何，error 申明是隐式的：

```
NSError * __strong e;
```

方法申明通常会是：

```
-(BOOL)performOperationWithError:(NSError * __autoreleasing *)error;
```
编译器会重写代码为：

```
NSError * __strong error;
NSError * __autoreleasing tmp = error;
BOOL OK = [myObject performOperationWithError:&tmp];
error = tmp;
if (!OK) {
    // Report the error.
    // ...
```

使用生命期修饰符避免强引用循环

你可以使用生命期修饰符避免强引用循环。举个例子，通常如果你在父子结构中有一个对象图谱，并且所有的父节点需要引用他们的子节点，反之也是这样。你将会制造出父子强引用关系和子父强引用关系。其他的情况可能更微妙，特别当他们用到了 `block` 对象。

在 MRC 模式下，`__block id x;` 不会持有(retain) x 的引用。在 ARC 模式，`__block id x;` 默认会持有 x 的引用。为了在 ARC 下面达到 MRC 的行为，你可以使用 `__unsafe_unretained __block id x;`。无论如何 `__unsafe_unretained` 表示未被持有的对象还是比较危险的，所以不鼓励这么使用。可以使用 __weak 或者设置 __block value 为 nil 来接触循环引用。

下面的代码通过 MRC 模式下的方式解释了这种问题。

```
MyViewController *myController = [[MyViewController alloc] init…];
// ...
myController.completionHandler =  ^(NSInteger result) {
   [myController dismissViewControllerAnimated:YES completion:nil];
};
[self presentViewController:myController animated:YES completion:^{
   [myController release];
}];
```

你可以使用 `__block` 修饰符，并在 completion handler 中设置 `myController` 变量为 nil。

```
MyViewController * __block myController = [[MyViewController alloc] init…];
// ...
myController.completionHandler =  ^(NSInteger result) {
    [myController dismissViewControllerAnimated:YES completion:nil];
    myController = nil;
};
```
你也可以使用临时 `__weak` 变量。下面的代码展示了简单的实现：

```
MyViewController *myController = [[MyViewController alloc] init…];
// ...
MyViewController * __weak weakMyViewController = myController;
myController.completionHandler =  ^(NSInteger result) {
    [weakMyViewController dismissViewControllerAnimated:YES completion:nil];
};
```

为了避免引用循环，你需要这样做：

```
MyViewController *myController = [[MyViewController alloc] init…];
// ...
MyViewController * __weak weakMyController = myController;
myController.completionHandler =  ^(NSInteger result) {
    MyViewController *strongMyController = weakMyController;
    if (strongMyController) {
        // ...
        [strongMyController dismissViewControllerAnimated:YES completion:nil];
        // ...
    }
    else {
        // Probably nothing...
    }
};
```

### ARC 使用新的表达式来管理自动释放池

```
@autoreleasepool {
	// Code, such as a loop that creates a large number of temporary objects.
}
```

在块开始的时候，释放池被推入栈；块退出(break, return, goto 等等) 释放池会弹出栈。为了兼容现有代码，如果推出会产生异常，那么这个自动释放池不会被弹出。

### 管理 Outlets 的方式成为了跨平台方式

这种申明 `outlets` 的模式在 iOS 和 OSX 中通过 ARC 得到了改变，并同时跨越了这两个平台。这种模式中，你需要这样来适配：outlets 需要设置为 weak，因为 File's Owner 对于 nib 文件中的顶级对象是强引用。

全部细节请看这里 <a href="https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/LoadingResources/Introduction/Introduction.html#//apple_ref/doc/uid/10000051i" target="_blank">Resource Programming Guide</a> 中的 <a href="https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/LoadingResources/Introduction/Introduction.html#//apple_ref/doc/uid/10000051i" target="_blank">Nib Files</a>

### 栈变量会被初始化成 nil

使用 ARC , strong, weak, 和 autoreleasing 栈变量，现在会隐式初始化成 nil。例如:

```
- (void)myMethod {
    NSString *name;
    NSLog(@"name: %@", name);
}
```

### 使用编译器标志来开启或关闭 ARC

你可以使用 `-fobjc-arc` 编译器标志来开启 ARC。如果读你来说某些文件使用 MRC 更方便的话，你也可以对每个文件单独使用 ARC 模式。项目使用 ARC 为默认方式，你可以使用 `-fno-objc-arc` 来关闭 ARC 。

ARC 被 Xcode 4.2 ， OSX v10.6 和 iOS4 以及后续版本。弱引用不被 OSX v10.6 和 iOS4 支持。Xcode 4.1 和以前的版本 不支持 ARC。


