---
layout: post
title: "iOS开发 RunTime"
date: 2019-05-22 15:44:45
tags: iOS
---

## 1.方法交换
我们可以将Method Swizzling功能封装为类方法，作为NSObject的类别，这样我们后续调用也会方便些。

NSObject+Swizzling.h
```
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface NSObject (Swizzling)

/**
 方法交换

 @param originalSelector 源方法
 @param swizzledSelector 新方法
*/
+ (void)methodSwizzlingWithOriginalSelector:(SEL)originalSelector
                        andSwizzledSelector:(SEL)swizzledSelector;
                        
@end

NS_ASSUME_NONNULL_END
```
NSObject+Swizzling.m
```
#import "NSObject+Swizzling.h"
#import<objc/runtime.h>

@implementation NSObject (Swizzling)

+ (void)methodSwizzlingWithOriginalSelector:(SEL)originalSelector
                        andSwizzledSelector:(SEL)swizzledSelector {
    Class class = [self class];
    //原有方法
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    //替换原有方法的新方法
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
    //先尝试給源SEL添加IMP，这里是为了避免源SEL没有实现IMP的情况
    BOOL didAddMethod = class_addMethod(class,originalSelector,
    method_getImplementation(swizzledMethod),
    method_getTypeEncoding(swizzledMethod));
    if (didAddMethod) {//添加成功：说明源SEL没有实现IMP，将源SEL的IMP替换到交换SEL的IMP
        class_replaceMethod(class,swizzledSelector,
        method_getImplementation(originalMethod),
        method_getTypeEncoding(originalMethod));
    } else {//添加失败：说明源SEL已经有IMP，直接将两个SEL的IMP交换即可
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

@end
```
ViewController.m
```
#import "ViewController.h"
#import "NSObject+Swizzling.h"

@implementation ViewController (Swizzling)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        // 交换对象方法
        [self methodSwizzlingWithOriginalSelector:@selector(viewWillAppear:) andSwizzledSelector:@selector(lczViewWillAppear:)];
        
        // 交换类方法
        [object_getClass([LibDevModel class]) methodSwizzlingWithOriginalSelector:@selector(onScanOver:) andSwizzledSelector:@selector(lczOnScanOver:)];
    });
}

```

## 2.消息转发

2.1 转发对象方法

LibDevModel+Swizzling.h
```
#import <DoorMasterSDK/DoorMasterSDK.h>

NS_ASSUME_NONNULL_BEGIN

@interface LibDevModel (Swizzling)
- (void)lczTimerCancel;
@end

NS_ASSUME_NONNULL_END
```

LibDevModel+Swizzling.m
```
#import "LibDevModel+Swizzling.h"
#import "GLGuardViewController.h"

@implementation LibDevModel (Swizzling)

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    // 获取 aSelector 的方法签名
    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    if (!signature) { // 如果无法获取
        // 判断 ViewController 是否能够响应 aSelector
        if ([ViewController instancesRespondToSelector:aSelector]) {
            // 获取 ViewController 中 aSelector 的方法签名
            signature = [ViewController instanceMethodSignatureForSelector:aSelector];
        }
    }
    return signature;
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([ViewController instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:[ViewController new]];
    }
}

```
GLGuardViewController.m
```
#import "GLGuardViewController.h"


@interface GLGuardViewController ()

@end

@implementation GLGuardViewController

- (void)lczTimerCancel {
    NSLog(@"调用了");
}
```

2.2 转发类方法

LibDevModel+Swizzling.h
```
#import <DoorMasterSDK/DoorMasterSDK.h>

NS_ASSUME_NONNULL_BEGIN

@interface LibDevModel (Swizzling)
+ (void)lczTimerCancel;
@end

NS_ASSUME_NONNULL_END
```
LibDevModel+Swizzling.m
```
+ (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    // 获取 aSelector 的方法签名
    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    if (!signature) { // 如果无法获取
        // 判断 ViewController 是否能够响应 aSelector
        if ([GLGuardViewController respondsToSelector:aSelector]) {
            // 获取 ViewController 中 aSelector 的方法签名
            signature = [GLGuardViewController methodSignatureForSelector:aSelector];
        }
    }
    return signature;
}

+ (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([GLGuardViewController respondsToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:[GLGuardViewController class]];
    }
}
```
GLGuardViewController.m
```
#import "GLGuardViewController.h"


@interface GLGuardViewController ()

@end

@implementation GLGuardViewController

+ (void)lczTimerCancel {
    NSLog(@"调用了");
}
```



