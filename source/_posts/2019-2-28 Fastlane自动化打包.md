---
layout: post
title: "iOS Fastlane自动化打包ipa"
date: 2019-02-28 18:00:41
tags: iOS
---

Fastlane是一套使用Ruby写的自动化工具集，旨在简化Android和iOS的部署过程，自动化你的工作流。它可以简化一些乏味、单调、重复的工作，像截图、代码签名以及发布App。[Fastlane文档](https://docs.fastlane.tools/ "Fastlane文档")

## 1. 安装Xcode命令行工具
安装命令: `$ xcode-select --install`
提示: `xcode-select: error: command line tools are already installed, use "Software Update" to install updates`  表示已经安装

## 2. 安装Fastlane
安装命令: `$ sudo gem install fastlane -NV` 或 `brew cask install fastlane`
检查是否安装成功 `fastlane --version`

## 3. 初始化Fastlane
cd到项目路径：`$ cd /Users/tanitsubukousha/Desktop/GLGL`
初始化Fastlane：`$ fastlane init`
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-2-28 Fastlane自动化打包/fastlane初始化.png?raw=true "Fastlane初始化")
```
1. 📸  Automate screenshots (自动化截图)
2. 👩‍✈️  Automate beta distribution to TestFlight (自动testfilght型配置)
3. 🚀  Automate App Store distribution (自动发布型配置)
4. 🛠  Manual setup - manually setup your project to automate your (需要手动配置内容)
```
我这里选择 4  等待一下会让你按回车 (共三次)  即安装成功！
在项目中会生成 fastlane文件夹、Gemfile、Gemfile.lock
fastlane文件夹：Appfile (存储有关开发者账号相关信息)、Fastfile (核心文件，主要用于 命令行调用和处理具体的流程)
## 4. 文件配置
### 4.1 打开Appfile文件 修改以下配置
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-2-28 Fastlane自动化打包/Appfile.png?raw=true "Appfile")
### 4.2 打开Fastfile文件 修改以下配置
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-2-28 Fastlane自动化打包/Fastfile.png?raw=true "Fastfile")
### 4.3 关于export_method方法
```
app-store,   #AppStore正式生产环境包
ad-hoc,  #生产测试包
enterprise,  #企业包
development  #开发测试包
```

## 5. 证书管理插件
match是fastlane的一个功能组件, 能自动从苹果官方上下载证书和pp文件同步到我们的git仓库中
cd到项目路径：`$ cd /Users/tanitsubukousha/Desktop/GLGL`
安装命令: `$ fastlane match init`
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-2-28 Fastlane自动化打包/match.png?raw=true "match")

### 5.1 先删除旧证书和pp文件
分别执行以下命令
```
$ fastlane match nuke development
$ fastlane match nuke distribution

```

### 5.2 生成证书和pp文件
cd到项目路径：`$ cd /Users/tanitsubukousha/Desktop/GLGL`
分别执行以下命令（首次执行时,会要求输入Git仓库密码,用来对证书进行加密,后续其他机器获取证书时使用该密码进行解密）
```
$ fastlane match development
$ fastlane match adhoc
$ fastlane match appstore

```

## 6. 打包ipa
打包命: `$ fastlane test`  test是Fastfile文件中的lane名称
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-2-28 Fastlane自动化打包/ipa.png?raw=true "ipa")

## 7. 问题
### 7.1 `Automatic signing is disabled and unable to generate a profile. To enable automatic signing, pass -allowProvisioningUpdates to xcodebuild.`
Xcode 9 之后禁止其直接访问钥匙串, 导致持续集成时执行 xcode build 会出现找不到证书或证书配置文件。修改 fastlane 配置文件来解决
![markdown](https://github.com/lczalh/blog/blob/master/source/images/2019-2-28 Fastlane自动化打包/7.1问题.png?raw=true "7.1问题")
`export_xcargs: "-allowProvisioningUpdates"`
