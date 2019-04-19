---
layout: post
title: "iOS XcodeBuild打包ipa"
date: 2019-03-04 10:07:13
tags: iOS
---

XcodeBuild是一个命令行工具，可以用来对Xcode工程或工作区进行编译、查找、分析、测试等各种操作

## 1. 进入项目的目录
`$ cd /Users/tanitsubukousha/Desktop/GLGL`   /Users/tanitsubukousha/Desktop/GLGL: 为你的项目路径

## 2. 清除编译过程生成的文件
`$ xcodebuild clean -$workspace $project_Name.$projectType -scheme $project_Name -configuration $configuration`
```
$workspace:  project / workspace（CocoaPods）
$project_Name: 项目名称
$projectType:  xcodeproj / xcworkspace（CocoaPods）
$configuration:  Debug / Release
```
出现` ** CLEAN SUCCEEDED **` 则清除成功

## 3. 生成 .xcarchive 文件
`$ xcodebuild archive -$workspace $project_Name.$projectType -scheme $project_Name -archivePath ./$project_Name.xcarchive`
执行完后目录下会生成 `$project_Name.xcarchive` 文件。

## 4. 配置plist文件
在当前目录下创建一个`app-store.plist`    appstore: 自定义名称
内容如下
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-3-04 XcodeBuild打包ipa/app-store.png?raw=true "app-store")
4.1关于method内容
```
app-store,   #AppStore正式生产环境包
ad-hoc,  #生产测试包
enterprise,  #企业包
development  #开发测试包
```

## 5. 打包ipa
`$ xcodebuild -exportArchive -exportOptionsPlist app-store.plist -archivePath ./$project_Name.xcarchive -exportPath ./autoPackage -allowProvisioningUpdates`
```
app-store.plist 创建的pislt文件
./autoPackage  ipa存放的路径
```
打包成功 会在当前目录生成一个`autoPackage`文件夹 
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-3-04 XcodeBuild打包ipa/ipa.png?raw=true "ipa")

## 6. 打包脚本
[XcodeprojShell](https://github.com/lczalh/XcodeprojShell "XcodeprojShell") 

使用方法：
### 6.1 将以下文件放入项目目录
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-3-04 XcodeBuild打包ipa/脚本.png?raw=true "脚本")
### 6.2 进入项目的目录
`$ cd 你的项目路径`
### 6.3 运行脚本
`$ ./XcodeprojShell.sh` 根据提示操作即可

## 7. 远程打包（ 必须每次先获取`keychain`的访问权限）
`$ security unlock-keychain -p $password /Users/$userName/Library/Keychains/Login.keychain`
```
$userName 主机用户名
$password 主机密码
```

## 8. 对应用重签名
### 8.1 将得到的ceshi.ipa 进行解压  `$ unzip ceshi.ipa`
### 8.2 删除旧签名 `$ rm -rf Payload/ceshi.app/_CodeSignature/`
### 8.3 将 `$ codesign -d --entitlements - Payload/XXX.app` 命令打印的内容,创建entitlements.plist文件
### 8.4 签名 `codesign -f -s "iPhone Distribution: XXX" --entitlements entitlements.plist Payload/ceshi.app`

