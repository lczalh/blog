---
layout: post
title: "iOS开发 SkeletonView"
date: 2019-06-17 14:12:42
tags: iOS
---
[SkeletonView](https://github.com/Juanpe/SkeletonView "SkeletonView")一种优雅的方式，向用户显示正在发生的事情，并为他们正在等待的内容做好准备

## 1.安装
`pod "SkeletonView"`

## 2.如何使用
### 2.1在适当的位置导入SkeletonView
`import SkeletonView`
### 2.2设置视图skeletonables
`avatarImageView.isSkeletonable = true`
### 2.3显示骨架
```
view.showSkeleton()                 // Solid
view.showGradientSkeleton()         // Gradient
view.showAnimatedSkeleton()         // Solid animated
view.showAnimatedGradientSkeleton() // Gradient animated
```
### 2.4更新骨架（如颜色，动画等）
```
view.updateSkeleton()                 // Solid
view.updateGradientSkeleton()         // Gradient
view.updateAnimatedSkeleton()         // Solid animated
view.updateAnimatedGradientSkeleton() // Gradient animated
```

### 2.5隐藏骨架
```
view.hideSkeleton()
```

## 3.UITableView
![markdown](https://github.com/lczalh/blog/blob/master/source/images/SkeletonView/tableView.jpg?raw=true "UITableView")

### 3.1遵守`SkeletonTableViewDataSource`协议，并实现协议方法
```
extension SearchMovieViewController: SkeletonTableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.models.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let model = self.models[indexPath.row]
        let cell = tableView.dequeueReusableCell(withIdentifier: "SearchMovieCell", for: indexPath) as! SearchMovieCell
        if model.vod_pic?.isEmpty == false {
            cell.movieImageView.kf.indicatorType = .activity
            cell.movieImageView.kf.setImage(with: ImageResource(downloadURL: URL(string: model.vod_pic!)!), placeholder: UIImage(named: "zanwutupian"))
        }
        cell.titleLabel.text = model.vod_name
        cell.detailsLabel.text = model.vod_blurb
        return cell
    }

    func collectionSkeletonView(_ skeletonView: UITableView, cellIdentifierForRowAt indexPath: IndexPath) -> ReusableCellIdentifier {
        return "SearchMovieCell"
    }
}
```
### 3.2 设置视图skeletonables
```
class SearchMovieCell: DiaryBaseTableViewCell {

/// 图片
var movieImageView: UIImageView!

/// 标题
var titleLabel: UILabel!

/// 详情
var detailsLabel: LCZAlignTopLabel!


override func config() {
    self.isSkeletonable = true
    movieImageView = UIImageView()
    self.contentView.addSubview(movieImageView)
    movieImageView.snp.makeConstraints { (make) in
        make.left.equalToSuperview().offset(15)
        make.centerY.equalToSuperview()
        make.height.equalTo(100)
        make.width.equalTo(80)
    }
    movieImageView.layer.cornerRadius = 5
    movieImageView.clipsToBounds = true
    movieImageView.contentMode = .scaleAspectFill
    movieImageView.setContentHuggingPriority(.required, for: .horizontal)
    movieImageView.setContentCompressionResistancePriority(.required, for: .horizontal)
    movieImageView.isSkeletonable = true

    titleLabel = UILabel()
    self.contentView.addSubview(titleLabel)
    titleLabel.snp.makeConstraints { (make) in
        make.left.equalTo(movieImageView.snp.right).offset(5)
        make.top.equalTo(movieImageView.snp.top)
        make.right.equalToSuperview().offset(-15)
    }
    titleLabel.font = LCZBoldFontSize(size: 16)
    titleLabel.setContentHuggingPriority(.required, for: .vertical)
    titleLabel.setContentCompressionResistancePriority(.required, for: .vertical)
    titleLabel.text = "xxxxxxxxx"
    titleLabel.isSkeletonable = true
    titleLabel.textAlignment = .left

    detailsLabel = LCZAlignTopLabel()
    self.contentView.addSubview(detailsLabel)
    detailsLabel.snp.makeConstraints { (make) in
        make.left.equalTo(movieImageView.snp.right).offset(5)
        make.top.equalTo(titleLabel.snp.bottom).offset(5)
        make.right.equalToSuperview().offset(-15)
        make.bottom.equalTo(movieImageView.snp.bottom)
    }
    detailsLabel.font = LCZFontSize(size: 14)
    detailsLabel.numberOfLines = 0;
    detailsLabel.textColor = LCZRgbColor(160, 160, 160, 1)
    detailsLabel.isSkeletonable = true
    detailsLabel.textAlignment = .left
    }

}

```

### 3.3 显示骨架
```
self.searchMovieView.tableView.visibleCells.forEach { $0.showAnimatedGradientSkeleton(usingGradient: SkeletonGradient(baseColor: UIColor.clouds),animation: GradientDirection.topLeftBottomRight.slidingAnimation()) }
```

## 4.UICollectionView
![markdown](https://github.com/lczalh/blog/blob/master/source/images/SkeletonView/collectionView.jpg?raw=true "UICollectionView")

### 4.1 遵守`SkeletonCollectionViewDataSource`协议，并实现协议方法
```
func collectionSkeletonView(_ skeletonView: UICollectionView, cellIdentifierForItemAt indexPath: IndexPath) -> ReusableCellIdentifier {
    return "MovieHomeCell"
}

func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
    let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "MovieHomeCell", for: indexPath) as! MovieHomeCell
    let model = self.models[indexPath.row]
    if model.vod_pic?.isEmpty == false {
        cell.imageView.kf.indicatorType = .activity
        cell.imageView.kf.setImage(with: ImageResource(downloadURL: URL(string: model.vod_pic!)!), placeholder: UIImage(named:"zanwutupian"))
    }
    cell.titleLabel.text = model.vod_name
    return cell
}
// 显示骨架cell的个数
func collectionSkeletonView(_ skeletonView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    return 12
}
```

### 4.2 设置视图skeletonables
```
class MovieHomeCell: DiaryBaseCollectionViewCell {

/// 图片
var imageView = UIImageView()

/// 标题
var titleLabel = UILabel()

override func config() {
    self.isSkeletonable = true
    self.contentView.layer.cornerRadius = 10
    self.layer.cornerRadius = 10
    self.contentView.clipsToBounds = true

    self.contentView.addSubview(self.titleLabel)
    self.titleLabel.font = LCZBoldFontSize(size: 16)
    self.titleLabel.snp.makeConstraints({ (make) in
        make.bottom.equalToSuperview().offset(-10)
        make.left.equalToSuperview().offset(10)
        make.right.equalToSuperview().offset(-10)
    })
    self.titleLabel.textAlignment = .center
    self.titleLabel.font = LCZFontSize(size: 14)
    self.titleLabel.textColor = LCZHexadecimalColor(hexadecimal: "#FECE1D")
    self.titleLabel.isSkeletonable = true
    self.titleLabel.text = "乐淘视界"

    self.contentView.addSubview(self.imageView)
    self.imageView.contentMode = .scaleAspectFill
    self.imageView.clipsToBounds = true
    self.imageView.snp.makeConstraints { (make) in
        make.left.top.right.equalToSuperview()
        make.bottom.equalTo(self.titleLabel.snp.top).offset(-10)
    }
    self.imageView.isSkeletonable = true
}

}
```

### 3.3 显示骨架
```
view.isSkeletonable = true
self.moreMoviesView.collectionView.prepareSkeleton(completion: { done in
    self.view.showAnimatedGradientSkeleton(usingGradient: SkeletonGradient(baseColor: UIColor.clouds),
    animation: GradientDirection.topLeftBottomRight.slidingAnimation())
})
```
