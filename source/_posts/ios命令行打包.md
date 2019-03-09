title: iOS命令行打包
entitle: 'ipa-command-packaging'
author: 唐先森
avatar: /images/favicon.png
authorLink: 'https://www.tangkunyin.com'
authorAbout: 'https://about.tangkunyin.com'
authorDesc: 一个写代码的「伪文人」
categories: 技术
timestamp: 1528014752
date: 2018-06-03 16:32:32
tags:
    - iOS
    - 开发日常
keywords: iOS, ipa
description: 来自2015.9.16的笔记：iOS命令行打包
photos:
    - /img/2018/15280148179275.jpg
---

### 1、概述：

打包这事儿其实就是让`xcrun`来干，而`xcodebuild`只是`xcrun`的一个软链接。分工如下：
> 1、`xcodebuild`负责讲工程源文件编译成xxx.app；
> 2、`xcrun`负责给xxx.app签名并打包成xxx.ipa

### 2、工作步骤：

1、先查看本机命令编译环境及需要编译项目的信息：

![](/img/2018/15280148179275.jpg)

2、开始编译：

> 1、清理：xcodebuild -target Test clean  
> 2、编译：xcodebuild -target Test
> 3、打包：xcrun -sdk iphoneos PackageApplication -v ./build/Release-iphoneos/Test.app -o ~/ipas/test.ipa


### 3、查看结果：

![](/img/2018/15280149958188.jpg)

### 4、特别提示：
> 以上打包方式仅对`*.xcodeproj`项目有效，如果是cocoapod项目，则需要改一遍编译命令：


```

xcodebuild -workspace Test.xcworkspace -scheme Test -configuration Release -derivedDataPath build

```


