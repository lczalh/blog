---
layout: post
title: "iOS RxDataSources"
date: 2019-04-02 10:48:47
tags: iOS
---
RxDataSource 的本质就是使用 RxSwift 对 UITableView 和 UICollectionView 的数据源做了一层包装

## 1. UITableView
数据源类型：` RxTableViewSectionedReloadDataSource`，`RxTableViewSectionedAnimatedDataSource`

## 2. UICollectionView
数据源类型：` RxTableViewSectionedReloadDataSource`，`RxCollectionViewSectionedAnimatedDataSource`

## 3. ModelType
模型类型：`SectionModel`，`AnimatableSectionModel`

## 4. 使用动画数据源，自定义模型需遵守`IdentifiableType`协议
```
class NewsListModel: Object,Mappable {
    @objc dynamic var publishTime: String? // 发布时间
    @objc dynamic var category: String? // 类型
    @objc dynamic var source: String? // 来源
    @objc dynamic var newsId: String? // 新闻ID
    @objc dynamic var title: String? // 标题
    @objc dynamic var newsImg: String? //新闻小图片url

    required convenience init?(map: Map) {
        self.init()
    }

    func mapping(map: Map) {
        publishTime   <- map["publishTime"]
        category   <- map["category"]
        source   <- map["source"]
        newsId   <- map["newsId"]
        title   <- map["title"]
        newsImg   <- map["newsImg"]
    }
}

extension NewsListModel: IdentifiableType {
    var identity: NewsListModel {
    return self
    }

    typealias Identity = NewsListModel
}
```

## 5. [Demo](https://github.com/lczalh/Diary)




