---
title: iOS 防止应用崩溃
catalog: true
date: 2019-03-23 19:38:51
subtitle:
header-img: ios.jpg
tags: iOS开发
---
面试网易有道，面试官在问 Runtime 时，提到了关于应用崩溃的问题，应该如何避免。

好吧，果然是道高一尺魔高一丈，面试前我还特意写了一篇博文 [iOS Runtime 学习](https://www.jianshu.com/p/86b8b3ee012f)，用来总结学习这方面的知识，应对面试，结果还是把我问到了，为什么每次觉得已经够深入了，却还是不够深入呢？为什么每次面试都能问到我不会的呢？感觉看了那么多面试题，还是有不会的。

没办法，记录一下吧，虽然面试凉凉，但这确实是一个很好的问题。

先解决 方法找不到 `unrecognized selector sent to instance` 而产生的崩溃吧。

## 一、实现原理
说到实现原理，这里又涉及到一道面试题，**从一个对象收到一个它无法响应的方法到崩溃之间发生了什么？**

**答：当一个方法找不到的时候，会走拦截调用和消息转发流程。**

大致流程是：

1、系统会调用该类的 `+resolveInstanceMethod:` 方法进行判断，如果返回 YES， 则表示能接受消息，NO 表示不能接受消息并进入第下步。

2、系统会调用该类的 `-forwardingTargetForSelector:`，在这个方法中，可以把这个消息转发给其他对象。敲黑板，这里是重点。

3、如果第二部返回 nil，那么首先它会发送 `-methodSignatureForSelector:` 消息，获得函数的参数和返回值类型。如果 `-methodSignatureForSelector:` 返回nil，Runtime则会发出 `-doesNotRecognizeSelector:` 消息，程序这时也就挂掉了。如果返回了一个函数签名，Runtime 就会创建一个 NSInvocation 对象并发送 `-forwardInvocation:`消息给目标对象。

其实在上面提到的那篇文章中已经说得很清楚了，只差最后一步了，就是这一点没捅破，真的很可惜。如果想了解更多，可以看看上面提到的那篇[文章]((https://www.jianshu.com/p/86b8b3ee012f))。

## 二、实现

1、给 NSObject 添加一个分类，再在这个分类中自定义一个方法，`-my_forwardingTargetForSelector:`，在 `+load` 方法中，交换 NSObject 的 `forwardingTargetForSelector:` 和 `my_forwardingTargetForSelector:`
```
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Method originalMethod = class_getInstanceMethod([NSObject class], @selector(forwardingTargetForSelector:));
        Method swizzledMethod = class_getInstanceMethod([NSObject class], @selector(my_forwardingTargetForSelector:));
        method_exchangeImplementations(originalMethod, swizzledMethod);
    });
}
```

2、在自定义的方法中，先判断当前对象是否已经实现了消息转发方法，如果没有实现，就动态创建一个类，给这个类动态添加一个方法，再把消息转发给这个动态创建的方法，这样就不会崩溃了。
```
- (id)my_forwardingTargetForSelector:(SEL)aSelector {
    
    // 获取NSObject的消息转发方法
    SEL sel = NSSelectorFromString(@"forwardingTargetForSelector:");
    Method method = class_getInstanceMethod(NSClassFromString(@"NSObject"), sel);
    // 获取当前类的消息转发方法
    Method _m = class_getInstanceMethod([self class],sel);
    
    // 类本身有没有实现消息转发流程
    BOOL transmit = method_getImplementation(_m) == method_getImplementation(method);
    
    /// 有木有实现下一步消息转发流程
    if (transmit) {
        /// 判断有没有实现第三步消息转发
        SEL sel1 = NSSelectorFromString(@"methodSignatureForSelector:");
        Method method1 = class_getInstanceMethod(NSClassFromString(@"NSObject"), sel1);
        
        Method _m1 = class_getInstanceMethod([self class], sel1);
        transmit = method_getImplementation(_m1) == method_getImplementation(method1);
        
        if (transmit) {
            // 创建一个新类
            NSString *errClassName = NSStringFromClass([self class]);
            NSString *errSel   = NSStringFromSelector(aSelector);
            NSLog(@"出问题的类，出问题的方法 == %@ %@", errClassName, errSel);
            
            NSString *className = @"CrachClass";
            Class cls = NSClassFromString(className);
            /// 如果类不存在 动态创建一个类
            if (!cls) {
                Class superCls = [NSObject class];
                cls = objc_allocateClassPair(superCls, className.UTF8String, 0);
                /// 给类添加方法
                class_addMethod(cls, aSelector, (IMP)Crash, "@@:@");
                objc_registerClassPair(cls);
            }
            /// 如果类没有对应的方法，则动态添加一个
            if (!class_getInstanceMethod(NSClassFromString(className), aSelector)) {
                class_addMethod(cls, aSelector, (IMP)Crash, "@@:@");
            }
            /// 把消息转发到当前动态生成类的实例上
            return [[NSClassFromString(className) alloc] init];
        }
    }
    return [self my_forwardingTargetForSelector:aSelector];
}
static int Crash(id slf, SEL selector) {
    return 0;
}
```

主要是注意两点：
>1：类有没有实现 `forwardingTargetForSelector:` 函数；
2：类有没有实现 `methodSignatureForSelector:` 函数，实现了就表明类实现了消息转发流程，那么不处理这个类。

## 三、其他类避免崩溃
还有一个常见的崩溃就是数组越界 `index 2 beyond bounds [0 .. 1]`，对于这一类方法，方法是把系统的方法和自定义的方法交换，在自定义的方法里判断是否越界。

根据上面的思路，给 NSArray 添加分类，交换 NSArray 的 `objectAtIndex:` 方法。

然鹅，但是，然并暖，程序还是崩溃了，信息如下：
```
Terminating app due to uncaught exception 'NSRangeException', reason: '*** -[__NSArrayI objectAtIndexedSubscript:]: index 2 beyond bounds [0 .. 1]'
```
恍然大悟，遂把需要交换的系统方法换成 `objectAtIndexedSubscript:`，而不是 `objectAtIndex:`，再次尝试。

What(⊙o⊙)?

程序还是崩溃了，崩溃信息还跟上面一样，这 TM 是几个意思？再次仔细查看崩溃日志，终于发现问题了：`-[__NSArrayI objectAtIndexedSubscript:]:`。这里是调用了 `__NSArrayI` 这个类的 `objectAtIndexedSubscript:` 方法，而不是 `NSArray`。再来一次，具体代码如下：
```
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class __NSArrayI = NSClassFromString(@"__NSArrayI");
        Method originalMethod1 = class_getInstanceMethod(__NSArrayI, @selector(objectAtIndexedSubscript:));
        Method swizzledMethod1 = class_getInstanceMethod(__NSArrayI, @selector(my_objectAtIndexedSubscript:));
        method_exchangeImplementations(originalMethod1, swizzledMethod1);
    });
}

- (id)my_objectAtIndexedSubscript:(NSUInteger)idx {
    if (idx < 0 || idx >= self.count) {
        NSLog(@"数组越界了~~~~~");
        return self[0];
    }
    return [self my_objectAtIndexedSubscript:idx];
}
```
最后，终于成功了。