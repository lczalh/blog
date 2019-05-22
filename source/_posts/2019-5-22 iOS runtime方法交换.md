---
layout: post
title: "iOS开发 runtime方法交换"
date: 2019-05-22 15:44:45
tags: iOS
---
我们可以将Method Swizzling功能封装为类方法，作为NSObject的类别，这样我们后续调用也会方便些。
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
例举
```
#import "ViewController.h"
#import "NSObject+Swizzling.h"
@interface ViewController ()

@end

@implementation ViewController

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        [self methodSwizzlingWithOriginalSelector:@selector(viewWillAppear:) andSwizzledSelector:@selector(lczViewWillAppear:)];
    });
}

- (void)viewWillAppear:(BOOL)animated {
    NSLog(@"视图即将显示");
}

- (void)lczViewWillAppear:(BOOL)animated {
    NSLog(@"视图即将显示前打印");
    [self lczViewWillAppear:animated];
}

- (void)viewDidLoad {
    [super viewDidLoad];
}
```




