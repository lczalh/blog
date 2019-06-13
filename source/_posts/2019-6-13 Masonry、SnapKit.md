---
layout: post
title: "iOS开发 SnapKit、Masonry"
date: 2019-06-13 18:20:24
tags: iOS
---
## 1. SnapKit
优先完全显示内容、抵抗压缩
```
self.title.setContentHuggingPriority(.required, for: .horizontal)
self.title.setContentCompressionResistancePriority(.required, for: .horizontal)
```

## 2. Masonry
优先完全显示内容、抵抗压缩
```
[self.titleLabel setContentHuggingPriority:UILayoutPriorityRequired forAxis:UILayoutConstraintAxisHorizontal];
[self.titleLabel setContentCompressionResistancePriority:UILayoutPriorityRequired forAxis:UILayoutConstraintAxisHorizontal];
```

## 3. 获取frame
```
self.titleLabel.layoutIfNeeded()
print(self.titleLabel.frame)
```

