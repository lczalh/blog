---
layout: post
title: "iOS开发 UITabBarController"
date: 2019-06-13 18:41:54
tags: iOS
---
## 1. 隐藏UITabBarController上的按钮
```
[viewController willMoveToParentViewController:nil];
[viewController.view removeFromSuperview];
[viewController removeFromParentViewController];
```


